## JDK8

取消了segment数组，直接用table保存数据，锁的粒度更小，减少并发冲突的概率。采用table数组元素作为锁，从而实现了对每一行数据进行加锁，进一步减少并发冲突的概率，并发控制使用Synchronized和CAS来操作。

#### 变量

```java
private static final int MAXIMUM_CAPACITY = 1 << 30; // 数组的最大值 

private static final int DEFAULT_CAPACITY = 16; // 默认数组长度 

static final int TREEIFY_THRESHOLD = 8; // 链表转红黑树的一个条件 

static final int UNTREEIFY_THRESHOLD = 6; // 红黑树转链表的一个条件 

static final int MIN_TREEIFY_CAPACITY = 64; // 链表转红黑树的另一个条件

static final int MOVED     = -1;  // 表示正在扩容转移 

static final int TREEBIN   = -2; // 表示已经转换成树 

static final int RESERVED  = -3; // hash for transient reservations 

static final int HASH_BITS = 0x7fffffff; // 获得hash值的辅助参数

transient volatile Node<K,V>[] table;// 默认没初始化的数组，用来保存元素 

private transient volatile Node<K,V>[] nextTable; // 转移的时候用的数组 

static final int NCPU = Runtime.getRuntime().availableProcessors();// 获取可用的CPU个数 

private transient volatile Node<K,V>[] nextTable; // 连接表，用于哈希表扩容，扩容完成后会被重置为 null 
```



```java
//延迟加载,构造中仅计算table大小,第一次使用才会创建
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
        if (loadFactor > 0.0F && initialCapacity >= 0 && concurrencyLevel > 0) {
            if (initialCapacity < concurrencyLevel) {
                initialCapacity = concurrencyLevel;
            }
//tableSizeFor依然保证计算出的大小是2**n,结果不一定是指定的initialCapacity大小
            long size = (long)(1.0D + (double)((float)((long)initialCapacity) / loadFactor));
            int cap = size >= 1073741824L ? 1073741824 : tableSizeFor((int)size);
            this.sizeCtl = cap;
        } else {
            throw new IllegalArgumentException();
        }
    }
```

#### get()

```java
//全程不加锁
public V get(Object key) {
    //保证hash返回结果是正整数
        int h = spread(key.hashCode());
        ConcurrentHashMap.Node[] tab;
        ConcurrentHashMap.Node e;
        int n;
    //n - 1 & h计算出桶下标
        if ((tab = this.table) != null && (n = tab.length) > 0 && (e = tabAt(tab, n - 1 & h)) != null) {
            int eh;//节点的hash
            Object ek;//节点的key
            if ((eh = e.hash) == h) {
                //hash相同比较key,先==比较,b再equals比较
                if ((ek = e.key) == key || ek != null && key.equals(ek)) {
                    return e.val;
                }
                //hash为负数代表该bin再扩容中或者是treebin,调用find来查找
            } else if (eh < 0) {
                ConcurrentHashMap.Node p;
                return (p = e.find(h, key)) != null ? p.val : null;
            }
		//顺序遍历链表查找
            while((e = e.next) != null) {
                if (e.hash == h && ((ek = e.key) == key || ek != null && key.equals(ek))) {
                    return e.val;
                }
            }
        }

        return null;
    }


final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key != null && value != null) {
            int hash = spread(key.hashCode());
            int binCount = 0;
            ConcurrentHashMap.Node[] tab = this.table;

            while(true) {
                int n;
                while(tab == null || (n = tab.length) == 0) {
                    //cas初始化table
                    tab = this.initTable();
                }

                ConcurrentHashMap.Node f;
                int i;
                //此位置为空，直接放入当作头节点
                if ((f = tabAt(tab, i = n - 1 & hash)) == null) {
                    if (casTabAt(tab, i, (Node)null, new Node(hash, key, value))){
                        break;
                    }
                } else {
                    int fh;
                    //发现正在扩容，就去帮助扩容
                    if ((fh = f.hash) == -1) {
                        tab = this.helpTransfer(tab, f);
                    } else {//运行到此处必定hash冲突
                        Object fk;
                        Object fv;
                        //根据onlyIfAbsent发现此处key相同选择不覆盖旧值直接返回
                        if (onlyIfAbsent && fh == hash && ((fk = f.key) == key || fk != null && key.equals(fk)) && (fv = f.val) != null) {
                            return fv;
                        }

                        V oldVal = null;
                        //此处才开始加锁
                        synchronized(f) {
                            //再次确认头节点未被移动
                            if (tabAt(tab, i) == f) {
                                if (fh < 0) {//此处是forwarding node或红黑树结构
                                    if (f instanceof ConcurrentHashMap.TreeBin) {
                                        binCount = 2;
                                        ConcurrentHashMap.TreeNode p;
                                        if ((p = ((ConcurrentHashMap.TreeBin)f).putTreeVal(hash, key, value)) != null) {
                                            oldVal = p.val;
                                            if (!onlyIfAbsent) {
                                                p.val = value;
                                            }
                                        }
                                    } else if (f instanceof ConcurrentHashMap.ReservationNode) {
                                        throw new IllegalStateException("Recursive update");
                                    }
                                } else {//此处是普通链表
                                    label124: {
                                        binCount = 1;

                                        ConcurrentHashMap.Node e;
                                        Object ek;
                                        for(e = f; e.hash != hash || (ek = e.key) != key && (ek == null || !key.equals(ek)); ++binCount) {
                                            ConcurrentHashMap.Node<K, V> pred = e;
                                            //添加至末尾
                                            if ((e = e.next) == null) {
                                                pred.next = new ConcurrentHashMap.Node(hash, key, value);
                                                break label124;
                                            }
                                        }
									//更新操作
                                        oldVal = e.val;
                                        if (!onlyIfAbsent) {
                                            e.val = value;
                                        }
                                    }
                                }
                            }
                        }

                        if (binCount != 0) {
                            //链表长度 >= 8 就树化
                            if (binCount >= 8) {
                                this.treeifyBin(tab, i);
                            }

                            if (oldVal != null) {
                                return oldVal;
                            }
                            break;
                        }
                    }
                }
            }
			//增加size计数
            this.addCount(1L, binCount);
            return null;
        } else {
            throw new NullPointerException();
        }
    }

//初始化table，利用cas，头节点的sizeCtl == -1表示正在创建，整数则代表扩容的阈值
private final ConcurrentHashMap.Node<K, V>[] initTable() {
        ConcurrentHashMap.Node[] tab;
        while((tab = this.table) == null || tab.length == 0) {
            int sc;
            if ((sc = this.sizeCtl) < 0) {
                Thread.yield();
            } else if (U.compareAndSetInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = this.table) == null || tab.length == 0) {
                        int n = sc > 0 ? sc : 16;
                        ConcurrentHashMap.Node<K, V>[] nt = new ConcurrentHashMap.Node[n];
                        tab = nt;
                        this.table = nt;
                        //扩容阈值计算
                        sc = n - (n >>> 2);
                    }
                    break;
                } finally {
                    this.sizeCtl = sc;
                }
            }
        }
    
    //维护计数，参数x是增加值，check是链表长度。此处和LongAdder的分组累加思想一致,
    //counterCells默认有2个cells，sumCount得到的size未必是精确值
    private final void addCount(long x, int check) {
        ConcurrentHashMap.CounterCell[] cs;
        long b;
        long s;
        int sc;
        //累加有冲突了
        if ((cs = this.counterCells) != null || !U.compareAndSetLong(this, BASECOUNT, b = this.baseCount, s = b + x)) {
            boolean uncontended = true;
            ConcurrentHashMap.CounterCell c;
            long v;
            //1.累加数组不存在 2.累加单元不存在 3.累加单元累加失败
            if (cs == null || (sc = cs.length - 1) < 0 || (c = cs[ThreadLocalRandom.getProbe() & sc]) == null || !(uncontended = U.compareAndSetLong(c, CELLVALUE, v = c.value, v + x))) {
                //创建累加单元和cell，累加重试
                this.fullAddCount(x, uncontended);
                return;
            }
			//链表长度不满足扩容条件
            if (check <= 1) {
                return;
            }
			//元素个数
            s = this.sumCount();
        }

        int n;
        ConcurrentHashMap.Node[] tab;
        if (check >= 0) {
            //需要扩容
            for(; s >= (long)(sc = this.sizeCtl) && (tab = this.table) != null && (n = tab.length) < 1073741824; s = this.sumCount()) {
                int rs = resizeStamp(n);
                if (sc < 0) {
                    ConcurrentHashMap.Node[] nt;
                    if (sc >>> 16 != rs || sc == rs + 1 || sc == rs + '\uffff' || (nt = this.nextTable) == null || this.transferIndex <= 0) {
                        break;
                    }
				//newTable已经创建，帮忙扩容
                    if (U.compareAndSetInt(this, SIZECTL, sc, sc + 1)) {
                        this.transfer(tab, nt);
                    }
                    //需要扩容，newTable未创建，sc变成负数
                } else if (U.compareAndSetInt(this, SIZECTL, sc, (rs << 16) + 2)) {
                    this.transfer(tab, (ConcurrentHashMap.Node[])null);
                }
            }
        }

    }
```

size计算发生在put，remove改变集合元素的操作中

## JDK7

由`Segment`数组结构和`HashEntry`数组组成。`Segment`是一种可重入锁`ReentrantLock`的子类，在 `ConcurrentHashMap` 里扮演锁的角色。是一种数组和链表的结构，一个`Segment`中包含一个`HashEntry`数组，每个`HashEntry`又是一个链表结构。当对 `HashEntry`数组的数据进行修改时，必须首先获得它对应的 `Segment` 锁。

### Segment

/**并发度可以理解为程序运行时能够「同时更新」ConccurentHashMap且不产生锁竞争的最大线程数，实际上就是ConcurrentHashMap中的分段锁个数，即Segment[]的数组长度。如果并发度设置的过小，会带来严重的锁竞争问题；如果并发度设置的过大，原本位于同一个Segment内的访问会扩散到不同的Segment中，CPU cache命中率会下降，从而引起程序性能下降。*/

```java
//默认segment大小是16，一开始就创建数组，segment[0]不是延迟加载，其他是延时加载
private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```



```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
     transient volatile HashEntry<K,V>[] table; // 可以理解为包含一个HashMap
}
```

### HashEntry

volatile的变量使其在高并发下的情况下如何保证取得的元素是最新的

```java
static class H<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;
 }
```

## 弱一致性

ConcurrentHashMap的弱一致性体现在迭代器,clear和get方法，原因在于没有加锁。

1. 比如迭代器在遍历数据的时候是一个Segment一个Segment去遍历的，如果在遍历完一个Segment时正好有一个线程在刚遍历完的Segment上插入数据，就会体现出不一致性。clear也是一样。

2. get方法和containsKey方法都是遍历对应索引位上所有节点，都是不加锁来判断的，如果是修改性质的因为可见性的存在可以直接获得最新值，不过如果是新添加值则无法保持一致性。

#### put

segment的put方法会去tryLock，失败的话会最多尝试获取锁64次，尝试的过程中顺便查看该节点在链表中有没有，没有就顺便创建出来。在后续直接将创建出来的node节点的next指向头节点，然后把node设置为头节点

#### rehash

segment获取下标的方法是把hashcode右移28位后与掩码位与运算，获取桶下标的方法是（桶长度-1）与hashcode与运算，rehash里直接把hashcode与掩码与运算

扩容发生在put过程中，无需加锁。只有一个节点的桶直接放入newTable的原下标位置，如果有多个节点，找到第一个rehash后下标不变的节点，从这节点开始直接搬迁到newTable中

扩容后新节点的put也是在rehash方法中完成的

#### get

用UNSAFE保证可见性

#### size

先不加锁循环遍历计数，如果前后两次一致得出size，三次后还不一致则加锁计数。



