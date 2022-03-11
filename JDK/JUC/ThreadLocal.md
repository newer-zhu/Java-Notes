**概念**

Thread对象里有一个map，key是ThreadLocal的hashcode，value是Object

每个线程都独享一块ThreadLocal Map。

**内存泄漏**

Map的Entry继承自WeakReference,所以它的key是弱引用。如果是强引用，即使threadlocal=null，key的引用仍然指向对象，造成内存泄漏。

但还是会有内存泄漏的情况

1. ThreadLocal对象被gc，key变成null，value就无法被访问。

2. 当使用static ThreadLocal的时候，延长ThreadLocal的生命周期，那也可能导致内存泄漏。因为，static变量在类未加载的时候，它就已经加载，当线程结束的时候，static变量不一定会回收。那么，比起普通成员变量使用的时候才加载，static的生命周期加长将更容易导致内存泄漏危机

3. 线程不结束，但是ThreadLocal没了，这时候可能会内存泄漏。

**优化**

JVM利用调用remove、get、set方法的时候，回收弱引用

Java为了最小化减少内存泄露的可能性和影响，在ThreadLocal的get,set的时候都会清除线程Map里所有key为null的value。所以最怕的情况就是，threadLocal对象设null了，开始发生“内存泄露”，然后使用线程池，这个线程结束，线程放回线程池中不销毁，这个线程一直不被使用，或者分配使用了又不再调用get,set方法，那么这个期间就会发生真正的内存泄露。

![image-20211029171106926](E:\学习笔记\typora\img\image-20211029171106926.png)

在ThreadLocal类中set(v)方法会调用ThreadLocalMap的set(this, v)方法

get()方法内调用 ThreadLocalMap`的 `getEntry(this)方法取值

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocal.ThreadLocalMap map = this.getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            this.createMap(t, value);
        }
    }


public T get() {
        Thread t = Thread.currentThread();
        ThreadLocal.ThreadLocalMap map = this.getMap(t);
        if (map != null) {
            ThreadLocal.ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                T result = e.value;
                return result;
            }
        }
        return this.setInitialValue();
    }

private T setInitialValue() {
        T value = this.initialValue();
        Thread t = Thread.currentThread();
        ThreadLocal.ThreadLocalMap map = this.getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            this.createMap(t, value);
        }

        if (this instanceof TerminatingThreadLocal) {
 TerminatingThreadLocal.register((TerminatingThreadLocal)this);
        }

        return value;
    }

protected T initialValue() {
        return null;
    }

void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocal.ThreadLocalMap(this, firstValue);
    }
```

**结论**

若第一次执行的操作是set则会创建ThreadLocalMap，并放入value，key是当前ThreadLocal。

若第一次执行的操作是get也会创建ThreadLocalMap。key为当前的ThreadLocal，value为null，并返回null。

**ThreadLocalMap**

Hash冲突解决办法：

根据 `key`计算 `hash`值，如果出现冲突，则向后探测，当到哈希表末尾的时候再从0开始，直到找到一个合适的位置，这种算法也决定了 `ThreadLocalMap`不适合存储大量数据。

扩容

`ThreadLocalMap`初始大小为 `16`，加载因子为 `2/3`，当 `size`大于 `threshold`时，就会进行扩容。

扩容时，新建一个大小为原来数组长度的**两倍**的数组，然后遍历旧数组中的 `entry`并将其插入到新的hash数组中，在扩容的时候，会把 `key`为 `null`的 `Entry`的 `value`值设置为 `null`，以便内存回收，减少内存泄漏问题。

**总结**

Threadlocal是什么？在堆内存中，每个线程对应一块工作内存，threadlocal就是工作内存的一小块内存。

Threadlocal有什么用？threadlocal用于存取线程独享数据，提高访问效率。