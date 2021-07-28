# LRU（Least Recently Used）

LRU是Least Recently Used的缩写，即最近最少使用，是一种常用的[页面置换算法](https://baike.baidu.com/item/页面置换算法/7626091)，选择最近最久未使用的页面予以淘汰。该算法赋予每个[页面](https://baike.baidu.com/item/页面/5544813)一个访问字段，用来记录一个页面自上次被访问以来所经历的时间 t，当须淘汰一个页面时，选择现有页面中其 t 值最大的，即最近最少使用的页面予以淘汰。（以上出自百度百科）

leetcode第146题以及460题涉及到此算法

~~~java
class LRUCache {

    private Map<Integer, Integer> map;
    private int cap;

    public LRUCache(int capacity) {
        map = new LinkedHashMap<>();
        cap = capacity;
    }
    
    public int get(int key) {
        // key不存在，访问不到
        if (!map.containsKey(key)) {
            return-1;
        }

        // key存在，需要将keyPair放到最后
        int val = map.remove(key);
        map.put(key, val);
        return val;
    }
    
    public void put(int key, int value) {
        // key存在，将存在的keyPair放到最后，并更新上新value
        if (map.containsKey(key)) {
            map.remove(key);
            map.put(key, value);
            return;
        }

        // key不存在，直接放到最后
        map.put(key, value);

        // 检查缓存容量，超出上限，则移除第一个
        if (map.size() > cap) {
            map.remove(map.entrySet().iterator().next().getKey());
        }
    }

}
~~~

# Redis的淘汰机制

## 过期数据删除策略

常用的过期数据的删除策略就两个（重要！自己造缓存轮子的时候需要格外考虑的东西）：

1. **惰性删除** ：只会在取出 key 的时候才对数据进行过期检查。这样对 CPU 最友好，但是可能会造成太多过期 key 没有被删除。
2. **定期删除** ： 每隔一段时间抽取一批 key 执行删除过期 key 操作。并且，Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响。

定期删除对内存更加友好，惰性删除对 CPU 更加友好。两者各有千秋，所以 Redis 采用的是 **定期删除+惰性/懒汉式删除** 。

但是，仅仅通过给 key 设置过期时间还是有问题的。因为还是可能存在定期删除和惰性删除漏掉了很多过期 key 的情况。这样就导致大量过期 key 堆积在内存里，然后就 Out of memory 了。因此需要Redis的内存淘汰机制来解决这个问题

## 内存淘汰机制

内存淘汰机制有以下几种

- **noeviction**: 当内存不足以容纳新写入数据时，新写入操作会报错。

- **allkeys-lru**：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。

- **allkeys-random**：当内存不足以容纳新写入数据时，在**键空间**中，随机移除某个 key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。

- **volatile-lru**：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。

- **volatile-random**：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key。

- **volatile-ttl**：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。

  

其中**allkeys-lfu**和**allkeys-lfu**是redis4.0后加入的

