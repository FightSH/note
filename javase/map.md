# Java7-HashMap

java7中的HashMap是由数组+链表组成，数组中的每个元素都是一个单向链表。如下图

![1](img/1.png)

上图中，每个绿色的实体是嵌套类 Entry 的实例，Entry 包含四个属性：key, value, hash 值和用于单向链表的 next。

capacity：当前数组容量，始终保持 2^n，可以扩容，扩容后数组大小为当前的 2 倍。

loadFactor：负载因子，默认为 0.75。

threshold：扩容的阈值，等于 capacity * loadFactor

# Java7-ConcurrentHashMap



# Java8-HashMap

在java8中，HashMap由**数组+链表+红黑树**组成。在java7HashMap(无红黑树结构)中，通过key的hash值的能够快速定位到数组的具体下标，之后需要顺着链表一个个的寻找才能寻找到所需要的value，时间复杂度为O(n)。而在java8中，当链表的元素达到8个时，会将链表转换为红黑树，在红黑树中寻找元素的时间复杂度为O(logN)。

在java8中，使用Node来代表数据节点（链表中使用Node，红黑树中使用TreeNode）。

~~~java

    transient Node<K,V>[] table;


    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    final float loadFactor;
 
// 以上为HashMap的fields

static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
		...
    }

~~~

## put

~~~java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 第一次put值的时候，会进行resize()操作。第一次进行resize()时，数组长度会从null初始化到默认的16(或者自定义的初始容量)
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 找到具体的数组下标，如果此下标为null，则直接初始化Node，并放置
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
    // 数组对应下标有数据
        Node<K,V> e; K k;
        // 判断该位置的第一个数据的key是否相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果p是红黑树，则调用红黑树对应的put方法
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // p是链表，
            for (int binCount = 0; ; ++binCount) {
                //尝试插入链表的尾部
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 如果新插入的值是链表中的第8个，那么将链表转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 如果在链表中找到相同的key
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 进行value值的覆盖，因为对应的key之前已经有值了
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // size超过阈值需要进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
~~~

## resize

resize()方法用于**初始化数组或者扩容数组**，每次扩容容量为原来的两倍

~~~java
final Node<K,V>[] resize() {
    //
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 对应数组扩容
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // 对应使用 new HashMap(int initialCapacity) 初始化后，第一次 put 时
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 对应使用 new HashMap() 初始化后，第一次 put 时
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    // 初始化新数组
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 以下代码为数据迁移代码。
    if (oldTab != null) {
        // 遍历数组，进行迁移
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 当前数组下标下的链表中只有一个元素，直接迁移即可
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 红黑树
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    //处理链表。处理链表时，会将原链表拆分为两个链表，放入新的数组中(保留之前的顺序)
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 若(e.hash & oldCap) == 0，则该节点在新数组的下标位置和其在旧数组的下标位置相同
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 否则该节点在新数组的下标位置为旧数组下标位置+旧数组长度
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
~~~

## get

get方法相对简单。

~~~java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
// 通过hash值，确定对应数组下标
// 判断该位置的第一个元素是否时需要的
// 如果不是判断其是否为红黑树，如果是，按照红黑树方式进行查找
// 如果是链表，遍历链表寻找key对应的value
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
~~~



# Java8-ConcurrentHashMap

相较于java7中的ConcurrentHashMap，java8的实现有了较大的改动