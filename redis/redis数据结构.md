# Redis常见数据结构与场景

## string

1. **介绍** ：string 数据结构是简单的 key-value 类型。其中value最大是512M。虽然 Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **简单动态字符串**（simple dynamic string，**SDS**）。相比于 C 的原生字符串，Redis 的 SDS 不光可以保存文本数据还可以保存二进制数据，并且获取字符串长度复杂度为 O(1)（C 字符串为 O(N)）,除此之外,Redis 的 SDS API 是安全的，不会造成缓冲区溢出。

   底层数据结构为动态字符串

2. **常用命令:** `set,get,strlen,exists,decr,incr,setex（此命令往往同时需要设置过期时间，需要考虑原子性问题）` 等等。

3. **应用场景** ：最常用的结构，理论来讲可存任何类型的数据，一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

   ~~~shell
   127.0.0.1:6379> set key value #设置 key-value 类型的值
   OK
   127.0.0.1:6379> get key # 根据 key 获得对应的 value
   "value"
   127.0.0.1:6379> exists key  # 判断某个 key 是否存在
   (integer) 1
   127.0.0.1:6379> strlen key # 返回 key 所储存的字符串值的长度。
   (integer) 5
   127.0.0.1:6379> del key # 删除某个 key 对应的值
   (integer) 1
   127.0.0.1:6379> get key
   (nil)
   ~~~

   

## hash

1. **介绍** ：hash 类似于 JDK1.8 前的 HashMap，内部实现也差不多(数组 + 链表)。不过，Redis 的 hash 做了更多优化。另外，hash 是一个 string 类型的 field 和 value 的映射表，**特别适合用于存储对象**，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。

   底层对应数据结构为两种：ziplist，hashtable。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable

2. **常用命令：** `hset,hmset,hexists,hget,hgetall,hkeys,hvals` 等。

3. **应用场景:** 系统中对象数据的存储。

   

   ~~~shell
   127.0.0.1:6379> hmset userInfoKey name "sh" description "dev" age "24"
   OK
   127.0.0.1:6379> hexists userInfoKey name # 查看 key 对应的 value中指定的字段是否存在。
   (integer) 1
   127.0.0.1:6379> hget userInfoKey name # 获取存储在哈希表中指定字段的值。
   "guide"
   127.0.0.1:6379> hget userInfoKey age
   "24"
   127.0.0.1:6379> hgetall userInfoKey # 获取在哈希表中指定 key 的所有字段和值
   1) "name"
   2) "sh"
   3) "description"
   4) "dev"
   5) "age"
   6) "24"
   127.0.0.1:6379> hkeys userInfoKey # 获取 key 列表
   1) "name"
   2) "description"
   3) "age"
   127.0.0.1:6379> hvals userInfoKey # 获取 value 列表
   1) "sh"
   2) "dev"
   3) "24"
   127.0.0.1:6379> hset userInfoKey name "sh123" # 修改某个字段对应的值
   127.0.0.1:6379> hget userInfoKey name
   "sh123"
   ~~~

## list

1. **介绍** ：**list** 即是 **链表**。链表是一种非常常见的数据结构，特点是易于数据元素的插入和删除并且且可以灵活调整链表长度，但是链表的随机访问困难。许多高级编程语言都内置了链表的实现比如 Java 中的 **LinkedList**，但是 C 语言并没有实现链表，所以 Redis 实现了自己的链表数据结构。Redis 的 list 的实现为一个 **双向链表**，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

   数据结构为快速链表quicklist，而列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，即压缩列表。其将所有元素挨着存储，分配的是连续的内存。当数据量比较多的时候，改为quicklist，因为普通链表需要附加指针，会比较浪费空间

   Redis将链表和ziplist结合起来组成了quicklist，即将多个ziplist使用双向指针串起来使用，这样即满足了快速的插入删除性能，又不会出现太大的空间冗余.

2. **常用命令:** `rpush,lpop,lpush,rpop,lrange、llen` 等。

3. **应用场景:** 发布与订阅或者说消息队列等，也可用做秒杀队列使用。

   ~~~shell
   127.0.0.1:6379> rpush myList value1 # 向 list 的头部（右边）添加元素
   (integer) 1
   127.0.0.1:6379> rpush myList value2 value3 # 向list的头部（最右边）添加多个元素
   (integer) 3
   127.0.0.1:6379> lpop myList # 将 list的尾部(最左边)元素取出
   "value1"
   127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
   1) "value2"
   2) "value3"
   127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
   1) "value2"
   2) "value3"
   
   127.0.0.1:6379> rpush myList2 value1 value2 value3
   (integer) 3
   127.0.0.1:6379> rpop myList2 # 将 list的头部(最右边)元素取出
   "value3"
   
   127.0.0.1:6379> rpush myList value1 value2 value3
   (integer) 3
   127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
   1) "value1"
   2) "value2"
   127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
   1) "value1"
   2) "value2"
   3) "value3"
   
   127.0.0.1:6379> llen myList
   (integer) 3
   ~~~

   

## set

1. **介绍 ：** set 类似于 Java 中的 `HashSet` 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。

   底层数据结构是value为null的hash表，因此添加删除，查找的复杂度都为O(1)

2. **常用命令：** `sadd,spop,smembers,sismember,scard,sinterstore,sunion` 等。

3. **应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景

   ~~~shell
   127.0.0.1:6379> sadd mySet value1 value2 # 添加元素进去
   (integer) 2
   127.0.0.1:6379> sadd mySet value1 # 不允许有重复元素
   (integer) 0
   127.0.0.1:6379> smembers mySet # 查看 set 中所有的元素
   1) "value1"
   2) "value2"
   127.0.0.1:6379> scard mySet # 查看 set 的长度
   (integer) 2
   127.0.0.1:6379> sismember mySet value1 # 检查某个元素是否存在set 中，只能接收单个元素
   (integer) 1
   127.0.0.1:6379> sadd mySet2 value2 value3
   (integer) 2
   127.0.0.1:6379> sinterstore mySet3 mySet mySet2 # 获取 mySet 和 mySet2 的交集并存放在 mySet3 中
   (integer) 1
   127.0.0.1:6379> smembers mySet3
   1) "value2"
   ~~~

   

## sorted set

1. **介绍：** 和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。有点像是 Java 中 HashMap 和 TreeSet 的结合体。

   底层使用了两种数据结构

   1. hash，hash作用是关联元素的value和权重score，保障元素value的唯一性，可通过value找到score。

   2. 跳跃表，目的在于给value进行排序，根据score的范围获取元素列表。

2. **常用命令：** `zadd,zcard,zscore,zrange,zrevrange,zrem` 等。

3. **应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

   ~~~shell
   127.0.0.1:6379> zadd myZset 3.0 value1 # 添加元素到 sorted set 中 3.0 为权重
   (integer) 1
   127.0.0.1:6379> zadd myZset 2.0 value2 1.0 value3 # 一次添加多个元素
   (integer) 2
   127.0.0.1:6379> zcard myZset # 查看 sorted set 中的元素数量
   (integer) 3
   127.0.0.1:6379> zscore myZset value1 # 查看某个 value 的权重
   "3"
   127.0.0.1:6379> zrange  myZset 0 -1 # 顺序输出某个范围区间的元素，0 -1 表示输出所有元素
   1) "value3"
   2) "value2"
   3) "value1"
   127.0.0.1:6379> zrange  myZset 0 1 # 顺序输出某个范围区间的元素，0 为 start  1 为 stop
   1) "value3"
   2) "value2"
   127.0.0.1:6379> zrevrange  myZset 0 1 # 逆序输出某个范围区间的元素，0 为 start  1 为 stop
   1) "value1"
   2) "value2"
   ~~~

   

## 其他数据结构

### hyperloglog

1. **介绍 ：** HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。在 Redis 里面，每个 HyperLogLog 键只需要花费 **12 KB** 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。需要注意的时，其计算出的结果是一个近似数（假设有1023468个用户进行访问，但得出的结果可能是1011325）。但其相比set结构，节省了大量的空间

2. **常用命令：** `pfadd` 、`pfcount` 、`pfmerge` 

3. **应用场景:**  网站数据分析（如pv、uv之类），周活，月活等数据、其他的一些基数统计的需求。以及一些数据的过滤去重。

   ~~~shell
   127.0.0.1:6379> pfadd uv 123 # 向hyporloglog中添加数据 pfadd key item
   (integer) 1	# 如果未计算过此数据，返回1。计算过返回0
   127.0.0.1:6379> pfadd uv 123 # 
   (integer) 0
   127.0.0.1:6379> pfcount uv # 获取计算结果
   (integer) 1
   127.0.0.1:6379> PFMERGE destkey sourcekey [sourcekey ...] # 将多个 HyperLogLog 合并为一个 HyperLogLog
   
   ~~~

   

### bitmap

1. **介绍 ：** bitmap 存储的是连续的二进制数字（0 和 1），通过 bitmap, 只需要一个 bit 位来表示某个元素对应的值或者状态，key 就是对应元素本身 。我们知道 8 个 bit 可以组成一个 byte，所以 bitmap 本身会极大的节省储存空间。

   如果要记录，在系统中执行一些特殊的操作，每天执行过某个操作的用户有多少个人，操作日志，审计日志，记录下来每个用户每天做了哪些操作，每个用户每天搞一个set，里面放他做的一些操作日志。也可以通过把网站里每一种操作每天执行过的用户都放在一个位图里，一个用户仅仅对应了一位而已。一个位图统计100万用户的行为，也不过一百多kb的内存

2. **常用命令：** `setbit` 、`getbit` 、`bitcount`、`bitop`

3. **应用场景:** 适合需要保存状态信息（比如是否签到、是否登录...）并需要进一步对这些信息进行分析的场景。比如用户签到情况、活跃用户情况、用户行为统计（比如是否点赞过某个视频）。

   ~~~shell
   # SETBIT 会返回之前位的值（默认是 0）这里会生成 7 个位
   127.0.0.1:6379> setbit mykey 7 1
   (integer) 0
   127.0.0.1:6379> setbit mykey 7 0
   (integer) 1
   127.0.0.1:6379> getbit mykey 7
   (integer) 0
   127.0.0.1:6379> setbit mykey 6 1
   (integer) 0
   127.0.0.1:6379> setbit mykey 8 1
   (integer) 0
   # 通过 bitcount 统计被被设置为 1 的位的数量。
   127.0.0.1:6379> bitcount mykey
   (integer) 2
   
   # 使用场景一：用户行为分析 很多网站为了分析你的喜好，需要研究你点赞过的内容。
   # 记录你喜欢过 001 号小姐姐
   127.0.0.1:6379> setbit beauty_girl_001 uid 1
   
   # 使用场景二：统计活跃用户 使用时间作为 key，然后用户 ID 为 offset，如果当日活跃过就设置为 1
   
   
   
   那么我该如果计算某几天/月/年的活跃用户呢(暂且约定，统计时间内只有有一天在线就称为活跃)，有请下一个 redis 的命令
   
   # 使用场景三：用户在线状态 对于获取或者统计用户在线状态，使用 bitmap 是一个节约空间效率又高的一种方法。
   # 只需要一个 key，然后用户 ID 为 offset，如果在线就设置为 1，不在线就设置为 0。
   ~~~

   

### geohash

1. **介绍 ：** 主要用于存储地理位置信息，并对存储的信息进行操作。

2. **常用命令：** `geoadd` 、`geopos` 、`geodist`、`georadius`、`georadiusbymember` 、`geohash`

3. **应用场景：** 商铺位置，附件的人等和坐标相关的业务功能

   ~~~shell
   127.0.0.1:6379> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania" # 添加坐标信息
   (integer) 2
   127.0.0.1:6379> GEODIST Sicily Palermo Catania # 计算两位置的距离
   "166274.1516"
   127.0.0.1:6379> GEORADIUS Sicily 15 37 100 km # 根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
   1) "Catania"
   127.0.0.1:6379> GEORADIUS Sicily 15 37 200 km
   1) "Palermo"
   2) "Catania"
   # georadiusbymember 和 GEORADIUS 命令一样，都可以找出位于指定范围内的元素，但是 georadiusbymember 的中心点是由给定的位置元素决定的，而不是使用经度和纬度来决定中心点。
   # GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
   
   127.0.0.1:6379> GEOPOS Sicily Palermo Catania NonExisting 
   #  用于从给定的 key 里返回所有指定名称(member)的位置（经度和纬度），不存在的返回 nil。
   1) 1) "13.36138933897018433"
      2) "38.11555639549629859"
   2) 1) "15.08726745843887329"
      2) "37.50266842333162032"
   3) (nil)
   # geohash 用于获取一个或多个位置元素的 geohash 值。
   127.0.0.1:6379> GEOHASH Sicily Palermo Catania
   1) "sqc8b49rny0"
   2) "sqdtr74hyu0"
   
   
   ~~~

   ### stream

   1. **介绍 ：** 主要用于消息队列（MQ，Message Queue），Redis 本身是有一个 Redis 发布订阅 (pub/sub) 来实现消息队列的功能，但它有个缺点就是消息无法持久化，如果出现网络断开、Redis 宕机等，消息就会被丢弃。

      