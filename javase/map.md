# Java8-HashMap

在java8中，HashMap由**数组+链表+红黑树**组成。在java7HashMap(无红黑树结构)中，通过key的hash值的能够快速定位到数组的具体下标，之后需要顺着链表一个个的寻找才能寻找到所需要的value，时间复杂度为O(n)。而在java8中，当链表的元素达到8个时，会将链表转换为红黑树，在红黑树中寻找元素的时间复杂度为O(logN)。



在java8中，使用Node来代表数据节点（链表中使用Node，红黑树中使用TreeNode）。

~~~java
	// 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
	// 节点数组
    transient Node<K,V>[] table;
	// 
    transient Set<Map.Entry<K,V>> entrySet;

    // 存放元素个数
    transient int size;

	// 计数器
    transient int modCount;

	// 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
	// 负载因子
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

- **loadFactor 负载因子**

	loadFactor 加载因子是控制数组存放数据的疏密程度，loadFactor 越趋近于 1，那么 数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，loadFactor 越小，也就是趋近于 0，数组中存放的数据(entry)也就越少，也就越稀疏。

	**loadFactor 太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor 的默认值为 0.75f 是官方给出的一个比较好的临界值**。

	给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 16 * 0.75 = 12 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。

- **threshold**

	**threshold = capacity \* loadFactor**，**当 Size>=threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 **衡量数组是否需要扩增的一个标准**。

## put方法

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
                //尝试插入链表的尾部(在java1.7中，采用的是头插法)
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

相较于java7中的ConcurrentHashMap，java8的实现有了较大的改动。不同于java7中采用**segment+HashEntry数组+链表**结构，其采用了**Node+链表+红黑树**结构。并使用了**Synchronized+CAS**来控制并发

## put方法

~~~java
public V put(K key, V value) {
        return putVal(key, value, false);
}


final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    // 记录链表长度
    int binCount = 0;
    // 进行自旋
    for (Node<K,V>[] tab = table; ; ) {
        // 注意这些变量所代表的含义
        Node<K,V> f; int n, i, fh; K fk; V fv;
        // 数组为空的情况下，进行初始化
        if (tab == null || (n = tab.length) == 0)
            // 初始化数组
            tab = initTable();
        // 找到hash值对应下标的的第一个节点 
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 使用CAS，尝试将node放入。如果数组该位置是空的，那么第一次就会成功。放入后break即可
            // 如果CAS失败，则说明有并发。继续循环进行下次操作
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;                   // no lock when adding to empty bin
        }
        // 这个节点正处于扩容状态
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 如果onlyIfAbsent为true，判断当前存在的节点和put的值是否相同并返回
        else if (onlyIfAbsent // check first node without acquiring lock
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv;
        else {
            //此时f为头节点并且不为空
            V oldVal = null;
            // 获取监视器锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // fh>=0，说明是链表
                    if (fh >= 0) {
                        // binCount用来记录链表长度
                        binCount = 1;
                        // 遍历链表
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 如果找到了对应的key，通过onlyIfAbsent判断是否进行valuie覆盖
                            // 之后就可break了
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            // 没找到对应的key，就将新的node置于链表尾部
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);
                                break;
                            }
                        }
                    }
                    // 红黑树
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        // 使用红黑树的putTreeVal方法
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                    // ReservationNode是内部方法使用的节点，可忽略
                    else if (f instanceof ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }
            
            if (binCount != 0) {
                // 判断是否将链表转换为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    // 此方法不一定会进行红黑树的转换
                    // 如果数组长度大于64，会选择进行数组扩容。
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}

~~~



## 初始化数组initTable()

~~~JAVA
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            // 到此说明有其他线程也在尝试初始化
            Thread.yield(); // lost initialization race; just spin
        // CAS将sizeCtl设为-1，表示抢到了锁
        else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    // 默认初始容量为16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    // 初始化数组
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    // 如果n为16，则sc = 0.75 * n
                    sc = n - (n >>> 2);
                }
            } finally {
                // 设置sizeCtl
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}

~~~

## 链表转红黑树treeifyBin

~~~java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n;
    if (tab != null) {
        // 如果数组长度小于64。会进行数组扩容
        // 因为数组的长度都是2的幂数，所以可能会是32、16、8...等
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        // 下面代码是链表转红黑树
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
~~~



## 扩容

~~~java

private final void tryPresize(int size) {
    // 这个size已经是原先的两倍了
    // 判断size是否超过最大容量。若不超过则 c 为 size 的 1.5 倍，再加 1，再往上取最近的 2 的 n 次方。
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
    tableSizeFor(size + (size >>> 1) + 1);
    int sc;
    while ((sc = sizeCtl) >= 0) {
       
        Node<K,V>[] tab = table; int n;
        if (tab == null || (n = tab.length) == 0) {
             // 下面代码和初始化代码类似
            n = (sc > c) ? sc : c;
            if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        // 
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
            break;
        else if (tab == table) {
            int rs = resizeStamp(n);
            // CAS操作将sizeCtl设置为 (rs << RESIZE_STAMP_SHIFT) + 2)
            if (U.compareAndSetInt(this, SIZECTL, sc,
                                   (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
        }
    }
}
~~~

transfer方法可能会多线程调用，

~~~java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    // stride可理解为“步长”，其在单核下直接等于n，即原数组长度
    // 在多核下，为 n>>>3/NCPU,最小值为16
	// 希望让每个CPU都尽可能处理的桶数一样多，如果桶很少，就默认一个线程处理16个桶
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    
    // 如果nextTab为null，会先进行一次初始化。
    // 可保证第一次个发起数据迁移的线程调用此方法时，nextTab为null（具体可看可看此方法的调用处）
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        // nextTable和transferIndex都是concurrentmap中的属性
        nextTable = nextTab;
        transferIndex = n;
    }
    
    int nextn = nextTab.length;
    // ForwardingNode——正在迁移中的node，这个node的hash值是 MOVED
    // 当其他线程发现此node时，会跳过。起到占位作用
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    // advance为true，则说明需要进行下个位置的迁移
    boolean advance = true;
    // 完成状态
    boolean finishing = false; // to ensure sweep before committing nextTab
    // i是位置索引下标
    // bound是当前线程可以处理的区间的最小边界
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 控制i的递减(从后向前)
        while (advance) {
            int nextIndex, nextBound;
            // 如果i-->=bound，或者完成状态为true，说明迁移任务已经完成。会将advance修改为false
            // 第一次进入循环，一般会进入下面分支代码(i--无法通过)。
            if (--i >= bound || finishing)
                advance = false;
            // 当线程进入会选取最新的转移下标
            // 如果 transferIndex <=0，意味着没有区间了，扩容可结束。
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            // CAS修改 transferIndex(length-区间值)。
            else if (U.compareAndSetInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                // 获取当前线程可出去区间的最小下标
                bound = nextBound;
                // i是当前线程可以处理的当前区间的最大下标
                i = nextIndex - 1;
                advance = false;
            }
        }
        // 对i的状态进行判断？
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) {
                // 完成扩容的情况下，对相关属性进行更新
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            // CAS操作将sizeCtl-1
            // sizeCtl 在迁移前会设置为 (rs << RESIZE_STAMP_SHIFT) + 2。然后，每有一个线程参与迁移就会将 sizeCtl 加 1
            // 这里去 -1 ，代表线程完成了属于自己的任务
            if (U.compareAndSetInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                // 任务完成，可return
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                // 说明 (sc - 2) == resizeStamp(n) << RESIZE_STAMP_SHIFT，即所有的迁移任务已完成
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        // 原数组下标i处节点若为空，则放入ForwardingNode(占位)
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        // 说明被其他线程占位了。继续下一个位置
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            // 到此处说明，当前下标有实际值，并且未被占位处理。
            // 对其加锁，准备进行迁移
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 将链表一分为二，并找到原链表中的lastRun，然后lastRun和其之后的节点进行迁移
                    // lastRun之前的节点进行克隆后，分到两个链表中
                    Node<K,V> ln, hn;
                    // 头结点hash>0，是链表，否则是红黑树
                    if (fh >= 0) {
                        // 对原数组长度进行与运算
                        int runBit = fh & n;
                        // 
                        Node<K,V> lastRun = f;
                        // 遍历
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            // 与 计算
                            int b = p.hash & n;
                            // 当前节点的计算结果和头节点不同
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        // 设置低位节点
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        // 设置高位节点
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        // 循环生成两个链表
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 低位链表放在新数组下标i处
                        setTabAt(nextTab, i, ln);
                        // 高位链表放在新数组下标i+n处
                        setTabAt(nextTab, i + n, hn);
                        // 原数组下标i处进行占位，表示处理完毕
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        // 树的处理...略过
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
~~~

