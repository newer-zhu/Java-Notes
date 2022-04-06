==继承AbstractMap，实现Map、Coloneable、Serializable==

## 重要变量

在 Java 中，散列表用链表数组实现。每个列表 被称为桶（ bucket)。
JDK使用的桶数是 2 的幂， 默认值为 16
如果散列表太满， 就需要再散列 （rehashed)。如果要对散列表再散列， 就需要创建一个桶数更多的表， 并将所有元素插入到这个新表中. 然后丢弃原来的表。装填因子（ load factor) 决定何时对散 列表进行再散列。例如， 如果装填因子为 0.75 (默认值，) 而表中超过 75%的位置已经填入元素， 这个表就会用双倍的桶数自动地进行再散列。

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

static final int MAXIMUM_CAPACITY = 1 << 30;

static final float DEFAULT_LOAD_FACTOR = 0.75f;

static final int TREEIFY_THRESHOLD = 8;

static final int UNTREEIFY_THRESHOLD = 6;

static final int MIN_TREEIFY_CAPACITY = 64;
//存放链表的数组
transient Node<K,V>[] table;
transient Set<Map.Entry<K,V>> entrySet;

transient int size;
//扩容阈值
int threshold;
final float loadFactor;
transient int modCount;
```

内部类

```java
class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

## 重要函数

```java
/**根据key算出下标
这段代码是为了对key的hashCode进行扰动计算，防止不同hashCode的高位不同但低位相同导致的hash冲突。简单点说，就是为了把高位的特征和低位的特征组合起来，降低哈希冲突的概率，也就是说，尽量做到任何一位的变化都能对最终得到的结果产生影响
*/
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

//返回比cap大的数，且此数为2的倍数
static final int tableSizeFor(int cap) {
    int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}

//获取元素
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
//获取比较简单
final HashMap.Node<K, V> getNode(int hash, Object key) {
        HashMap.Node[] tab;
        HashMap.Node first;
        int n;
    	//table校验，并根据n - 1 & hash找到桶，给first赋值桶的头节点
        if ((tab = this.table) != null && (n = tab.length) > 0 && (first = tab[n - 1 & hash]) != null) {
            Object k;
            //头节点hashcode相同且key相同
            if (first.hash == hash && ((k = first.key) == key || key != null && key.equals(k))) {
                return first;
            }

            HashMap.Node e;
            if ((e = first.next) != null) {
                //红黑树寻找逻辑
                if (first instanceof HashMap.TreeNode) {
                    return ((HashMap.TreeNode)first).getTreeNode(hash, key);
                }

                do {//遍历链表
                    if (e.hash == hash && ((k = e.key) == key || key != null && key.equals(k))) {
                        return e;
                    }
                } while((e = e.next) != null);
            }
        }

        return null;
    }

//放入key和value，第四个参数是onlyIfAbsent,第五个是evict
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

①.判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；
②.根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；
③.判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；
④.判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；
⑤.遍历table[i]，判断链表长度是否大于8（且），大于8的话（且数组的长度大于64）把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；
⑥.插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            //如果table为空，扩容，扩容后元素下标保持不动或以原容量的大小偏移
            n = (tab = resize()).length;
    /**将hash生成的整型转换成链表数组中的下标。n是table的长度，所以肯定是2的倍数，那么n-1肯定是奇数。因为奇数的高位肯定是0，这样与hash做与运算就相当于取模运算。又因为hash的高位无论如何都会变成0，所以hash高位不影响下标，所以容易造成hash冲突，需要进行hash分散算法。
    */
        if ((p = tab[i = (n - 1) & hash]) == null)
            //数组里没有链表直接放入
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            //存在数据，说明发生了hash冲突, 继续判断key是否相等，相等，用新的value替换原数据
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;//e指向table[i]的第一个节点
            else if (p instanceof TreeNode)
                //如果是树节点，创建树型节点插入红黑树中
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //如果不是树型节点，创建普通Node加入链表中；
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {//如果遍历到了尾部
                        p.next = newNode(hash, key, value, null);//插入尾部
                        //判断链表长度是否大于 8， 大于的话链表转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    //如果找到了相同的key，跳出
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                    p = e;
                }
            }
            if (e != null) { // 这个key存在映射
                V oldValue = e.value;
                //参数条件判断，是否只在不存在key的情况下put
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;//覆盖
                //回调函数
                afterNodeAccess(e);
                //返回被覆盖的值
                return oldValue;
            }
        }
        ++modCount;
    //判断是否要扩容
        if (++size > threshold)
            resize();
    //插入后回调
        afterNodeInsertion(evict);
        return null;
    }
```

### 1.8的优化

1.8中链表的插入方式从头插法改成了尾插法，数据结构变成了数组+链表+红黑树，先插入再判断是否扩容

扩容是扩大为原数组大小的2倍，用于计算数组位置的掩码仅仅只是高位多了一个1，& 运算，1和任何数 & 都是它本身，那就分二种情况，如下图：原数据hashcode高位第4位为0和高位为1的情况；

第四位高位为0，重新hash数值不变，第四位为1，重新hash数值比原来大16(旧数组的容量)

*头插法会使链表发生反转，多线程环境下会产生环型链表，线程1在执行完`Entry<K,V> next = e.next`;假设e指向头节点，next指向e的下一个节点，此时挂起，线程2开始扩容，扩容结束后next变成了头节点，e变成了next的next，虽然1.8后尾插法可以保证顺序与扩容前一致，但是多线程下会丢数据*

## 常见问题

### 数组长度为2的幂的好处

#### 取模运算

**hashmap根据[ (table_length - 1) & hash ]获取桶下标**

Java之所有使用位运算(&)来代替取模运算(%)，最主要的考虑就是效率。位运算(&)效率要比代替取模运算(%)高很多，主要原因是位运算直接对内存数据进行操作，不需要转成十进制，因此处理速度非常快，还有一个好处就是可以很好的解决负数的问题

**之所以可以做等价代替，前提是要求HashMap的容量一定要是2^n**

X % 2^n  =  X & (2^n - 1)

假设n为3，则2^3 = 8，表示成2进制就是1000。2^3 -1 = 7 ，即0111。

此时X & (2^3 - 1) 就相当于取X的2进制的最后三位数。

从2进制角度来看，X / 8相当于 X >> 3，即把X右移3位，此时得到了X / 8的商，而被移掉的部分(后三位)，则是X % 8，也就是余数

那除了效率之外呢？

如果size=11，size=13.....，上面的式子就变成了10 & hash, 12 & hash......

那么hash碰撞的概率会非常大，用&计算下标为了能更分散我们希望hash能和1做&，这样子hash均匀的话我们的桶下标分布也会均匀。

如1010 
    1111 -> 1010，其中hash是1010，1111是15，也就是table长度-1，而2次幂-1的数后面都是1，满足条件

### hashcode位运算

```java
//jdk1.7
h ^= k.hashCode(); h ^= (h >>> 20) ^ (h >>> 12); return h ^ (h >>> 7) ^ (h >>> 4);
```

关于Java 8中的hash函数，原理和Java 7中基本类似。Java 8中这一步做了优化，只做一次16位右位移异或混合，而不是四次，但原理是不变的

#### 扩容判断

有一个元素X，hash后的值为29，二进制为： 0001 1101，数组长度为16.根据上面的式子得出29 & 15

0001 1101
&
0000 1111
0000 1101 = 13，扩容后数组长度32，原索引值为13，32比16在二进制上多出的那一位，也就是16，则

0001 1101
&
0001 1111
0001 1101 = 13 + 16，得到新下标。所以判断扩容后位置就变得非常简单，如果hashcode二进制上与扩容后长度-1最高位对应的那位是0还是1，如果是1，1&1=1，则下标+16，也就是多出来的那一位1，否则不变





## HashTable

HashMap和HashTable对于计算数组下标这件事，采用了两种方法。HashMap采用的是位运算，而HashTable采用的是直接取模

HashTable默认的初始大小为11，之后每次扩充为原来的2n+1

由于HashTable会尽量使用素数、奇数作为容量的大小。当哈希表的大小为素数时，简单的取模哈希的结果会更加均匀

当我们明确知道HashMap中元素的个数的时候，把默认容量设置成expectedSize / 0.75F + 1.0F 是一个在性能上相对好的选择，但是，同时也会牺牲些内存.

*WeakHashMap 使用弱引用 （ weak references) 保存键。 WeakReference 对象将引用保存到另外一个对象中，在这里，就是散列键。对于这种类型的 对象，垃圾回收器用一种特有的方式进行处理。通常，如果垃圾回收器发现某个特定的对象 已经没有他人引用了，就将其回收。然而， 如果某个对象只能由 WeakReference 引用， 垃圾 回收器仍然回收它，但要将引用这个对象的弱引用放人队列中。WeakHashMap 将周期性地检 查队列， 以便找出新添加的弱引用。一个弱引用进人队列意味着这个键不再被他人使用， 并 且已经被收集起来。于是， WeakHashMap 将删除对应的条目。*

 