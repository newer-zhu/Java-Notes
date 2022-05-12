**==实现了集合List，序列化serializable,随机访问RandomAccess和克隆cloneable的接口，继承了AbstractList==**

![image-20210814213427716](E:\学习笔记\typora\img\image-20210814213427716.png)

## 接口

**拷贝**

**随机访问**

允许通用算法更改其行为，提供良好的性能。实现此接口的用fori遍历更快

## 重要变量

elementData：是transient类型的，储存变量的容器

DEFAULT_CAPACITY = 10：默认容量

DEFAULTCAPACITY_EMPTY_ELEMENTDATA = new Object[0]：空参构造会赋值给elementData

EMPTY_ELEMENTDATA = new Object[0]：空数组

modCount：它表示该集合实际被修改的次数

expectedModCount ：是 ArrayList中的一个内部类——Itr（继承自Iterator）中的成员变量

## 重要函数

```java
//根据当前容量判断是否要扩容，返回值是数组的新容量
private int newCapacity(int minCapacity) {
    int oldCapacity = this.elementData.length;
    //扩容1.5倍后的容量
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (this.elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(10, minCapacity);//返回默认容量10
        } else if (minCapacity < 0) {
            throw new OutOfMemoryError();
        } else {
            return minCapacity;//不扩容
        }
    } else {
        //一般情况下返回扩容1.5倍的值，新容量过大时进入hugeCapacity逻辑
        return newCapacity - 2147483639 <= 0 ? newCapacity : hugeCapacity(minCapacity);
    }
}

//扩容方法
private Object[] grow() {
        return this.grow(this.size + 1);
    }

private Object[] grow(int minCapacity) {
        return this.elementData = Arrays.copyOf(this.elementData, 	this.newCapacity(minCapacity));
    }

//添加元素时下标校验
private void rangeCheckForAdd(int index) {
        if (index > this.size || index < 0) {
            throw new IndexOutOfBoundsException(this.outOfBoundsMsg(index));
        }
    }

//并发修改次数校验。此方法在Itr内部类里
final void checkForComodification() {
            if (ArrayList.this.modCount != this.expectedModCount) {
                throw new ConcurrentModificationException();
            }
        }

//真正删除元素的方法
//当要删除的元素在倒数第二的位置时，不会产生并发修改异常
private void fastRemove(Object[] es, int i) {
        ++this.modCount;
        int newSize;
        if ((newSize = this.size - 1) > i) {
            //此时位于i的元素已被覆盖，但最后两个下标元素会重复
            System.arraycopy(es, i + 1, es, i, newSize - i);
        }
		//置空重复的元素，更新size
        es[this.size = newSize] = null;
    }

/**数组拷贝
   *Object src : 原数组
   *int srcPos : 从元数据的起始位置开始
　　*Object dest : 目标数组
　　*int destPos : 目标数组的开始起始位置
　　*int length  : 要copy的数组的长度
*/
public static void arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
```

iterator的remove方法和ArratList的remove方法不一样。前者不会有并发修改异常，因为前者将modCount重新赋值给了expectedModCount

## 常见问题

### 并发修改

在增强for循环中，集合遍历是通过iterator进行的，但是元素的add/remove却是直接使用的集合类自己的方法。这就导致iterator在遍历的时候，会发现有一个元素在自己不知不觉的情况下就被删除/添加了，就会抛出一个异常，用来提示用户，可能发生了并发修改！

普通的for循环还是可以的，因为普通for循环并没有用到Iterator的遍历，所以压根就没有进行fail-fast的检验，或者直接使用Iterator进行操作

如果要一个arrayList中有多个相同的元素，要删除所有此元素的话要普通for循环从后往前遍历，不然只会删除第一个符和条件的元素

### 线程安全

1. 替换为vector，Vector每次扩容其大小的双倍空间

2. 用Collections.synchronizedList转换为线程安全

3. CopyOnWriteArrayList类可以解决这个问题，

   迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。

   CopyOnWriteArrayList中add/remove等写方法是需要加锁的，目的是为了避免Copy出N个副本出来，导致并发写。

   写时复制的思想是通过延时更新的策略来实现数据的最终一致性的，并非强一致性

    

   所以CopyOnWrite容器是一种读写分离的思想，读和写不同的容器。而Vector在读写的时候使用同一个容器，读写互斥，同时只能做一件事儿。

SynchronizedList中实现的类并没有都使用synchronized同步代码块。其中有listIterator和listIterator(int index)并没有做同步处理。 在使用SynchronizedList进行遍历的时候要手动加锁

### 复制ArrayList

1.clone 	2. 构造方法 	3.addAll方法

### asList

asList 得到的只是一个 Arrays 的内部类，一个原来数组的视图 List，因此如果对它进行增删操作会报错

*默认情况下ArrayList的初始容量非常小,所以如果可以预估数据量的话,分配一个较大的初始值属于最佳实践,这样可以减少调整大小的开销*

