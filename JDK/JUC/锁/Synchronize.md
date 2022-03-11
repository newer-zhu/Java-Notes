# 概念

**不可继承**

Synchronize定义的方法被子类覆盖后并不是同步的，也就是不可继承性

**虚假唤醒**

由于wait方法调用后，如果被唤醒就会直接执行wait后面的代码，如果此时wait在一个if判断body中，将不会再次执行判断，所以要用while替代if操作

**锁对象**

如果锁的是成员方法，锁对象是this，也就是调用方法的对象，如果是静态方法，则锁对象是类

**可重入锁**

Synchronize和Lock都是可重入锁，也就是能自由进出锁和内层锁，递归进入，一个线程可以重复获得同一个锁

**锁重入**

 在已经获得锁的同步方法或同步代码块内部可以调用锁定对象的其他同步方法，synchronize重入会在新增一条Lock Record，但不交换锁对象的MarkWord数据。

**死锁：**

一个线程持有一把锁同时想获取另一个锁，对另一个线程同理，此时就会死锁。可以用jps、jstack工具检测到死锁

如果线程获取所得顺序做出特定的改变就能解决死锁问题。但可能会导致饥饿问题。

**活锁：**

两个线程互相改变对方的结束条件，两个线程都一直无法结束。此处和synchronize没有关系，此处的锁代表的是广义的锁。两个线程的执行交错开就能解决活锁。

```java
class liveLock{
    static volatile int count = 10;
    static final Object lock = new Object();
            
    public static void main(String[] args) {
        new Thread(() -> {
            while (count > 0){
                Thread.sleep(3000);
                count--;
            }
        }, "t1").start();
        
        new Thread(() -> {
            while (count < 30){
                Thread.sleep();
                count++;
            }
        }, "t2").start();
    }
}
```



**饥饿：**

某个线程一直得不到执行机会

**锁消除**

由JIT即时编译器优化

sb这个引用只会在add中引用，不会被其他线程引用。所以JVM自动消除sb对象内部的锁

```java
public void add(){
    StringBuffer sb = new StringBuffer();
	sb.append(str1).append(str2);
}

```

**锁粗化**

如果一连串操作都是对同一对象加锁，JVM会把加锁的范围粗化到一连串操作的外部。比如while循环中对StringBuffer进行append操作100次，此时while部分代码会被加一次锁

**锁细分**

一个类中有不同的业务实现，且之间数据不互相影响，这时为了提高并发度可以设置这个类的多把锁。业务不同的代码选择属于自己业务的那一把锁。比如Room类可以有睡觉和学习两个功能，那就实例化studyRoom和sleepRoom两把锁。睡觉的同步代码块选择sleepRoom为锁对象，学习的同步代码块选择studyRoom作为锁对象。

坏处就是一个线程可能就要获取多把锁，会导致死锁。比如A线程获得studyRoom锁后，在同步代码块里又想获得sleepRoom，而线程B恰好与之相反，此时就进入了死锁状态。

死锁可用 jps + jstack定位

**线程经验公式：**
线程数 = 核数 * 期望cpu利用率 * 总时间（cpu计算时间 + 等待时间） / cpu计算时间

## 队列

![image-20220207003251563](E:\学习笔记\typora\img\image-20220207003251563.png)

当加上synchronized时，Mark Word就会指向Monitor，成为Owner，此时其他线程会进入EntryList等待。进入Block状态，EntryList唤醒是非公平的。若获得锁但不满足条件的线程会进入WaitSet。

==Entrylist中的线程处于阻塞状态，也就是线程还没有拿到锁。Wait Set中的线程处于等待状态，也就是拿到锁后暂时释放锁（等待条件成熟会再次获取锁）。==

synchronize是非公平锁，并不一定会把锁交给EntryList第一个节点

**monitor**

monitor是操作系统提供的，具有**WaitSet， EntryList， Owner**

Owner指向锁持有者，在EntryList中的就是Blocked状态的线程，WaitSet中的就是Waiting状态的线程。

Blocked：线程未获取到锁，在EntryList中等待获取锁

Waiting： 线程持有锁时由于不满足继续运行的条件，只能暂时放弃锁，进入WaitSet等待日后其他线程notify，唤醒以后不会直接获取锁，而是会进入EntryList与其他线程一起竞争锁。

每个对象头中的MarkWord都可以指向一个Monitor

**Lock Record**

每个线程的栈帧包含的锁记录，内部储存锁定对象的MarkWord

内部有reference指向锁定对象，还有一个对象储存交换后的Markword

解锁事Lock Record一条一条弹出，最后一条弹出时对MarkWord进行了还原

### 对象头

`Java`对象在内存中储存的布局可以分为`3`块区域：**对象头**、**实例数据**、**对齐填充**。

一个普通对象头有两个部分，Mark Word和Klass Word

**Mark Word**：记录了对象和锁的有关信息，储存对象自身的运行时数据，如哈希码(HashCode)、GC分代年龄、锁标志位、线程持有的锁、偏向线程ID、偏向时间戳、对象分代年龄等。注意这个Mark Word结构并不是固定的，它会随着锁状态标志的变化而变化，而且里面的数据也会随着锁状态标志的变化而变化，这样做的目的是为了节省空间。
**类型指针Klass Word**：指向对象的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
**数组长度**：这个属性只有数组对象才有，储存着数组对象的长度。

Mark Word结构图：

![](E:\学习笔记\typora\img\markword.png)

## 锁升级

**无锁->偏向锁->轻量级自旋锁->向OS申请重量级锁**

### 偏向锁

偏向锁具有延迟性，一个对象刚刚创建时锁状态时001

markword用54bit空间存放当前线程指针，所有调用可偏向对象的hashcode方法时，会取消偏向状态到无锁状态，因为biased状态无法储存hashcode

1. 检查`Mark Word`是否为**可偏向锁的状态**

2. 第一次使用CAS将线程ID设置到Mark word头，之后发现线程ID是自己的则表示没有锁竞争，不需要CAS
3. 之后发现不是自己线程ID，通过CAS去修改成自己的ID，若修改不成功执行4
4. 当拥有该锁的线程到达安全点（时间点上没有字节码正在执行）之后，挂起这个线程，升级为轻量级锁。

默认开启偏向锁后三位是101，否则是001，且默认有延迟，应用立马启动后不会生效，而是4s后生效。可以加上JVM参数禁止延迟。

在多线程的情况下可以禁用偏向锁。

可偏向的对象调用hash方法后会清空Mark word，导致变成不可偏向的对象。原因：Mark word里61bit放了线程Id，没有多余空间。而其他锁的hashcode存在锁记录和monitor里，自动还原。

**wait，notify属于重量级锁的操作，使用会撤销偏向锁**

**批量重偏向**：
 当一个锁对象偏向一个线程超过阈值（20）时，JVM会批量把剩余的对象批量偏向到另一个线程。

如线程A对找30个对象作为synchronized的锁对象，线程B也是如此。且两个线程总是交替运行。此时前19个对象的MarkWord在线程B的synchronize关键字的前面，同步代码块中，关键字的后面经历的步骤是101(偏向A的偏向锁) -> 00(轻量级锁) -> 001(无锁)。此时开始批量重偏向，剩余对象头的markWord不再升级轻量级锁，而是直接偏向第二个线程。即101(偏向A的偏向锁) -> 101(偏向B的偏向锁) -> 101(偏向B的偏向锁)

**批量撤销**：

当撤销偏向锁超过40次后，又会把整个类的所有锁对象全部撤销为不可偏向对象

### CAS自旋锁（轻量级锁）

1. 若为无锁（01）状态，则在当前线程中建立一个锁记录Lock Record，用于储存锁对象目前的`Mark Word`的拷贝（Displaced Mark Word）
2. 将对象头的`Mark Word`拷贝到线程的锁记录(Lock Recored)中。
3. 对象的`markword`更新为指向指向线程栈中Lock Record的指针（CAS），交换成功说明上锁成功，否则如果状态已经是00则表示已经加了轻量锁，上锁失败
4. 如果自旋结束后还是没获得锁，锁膨胀为重量级锁（01），**Mark Word中储存就是指向`monitor`对象的指针，当前线程以及后面等待锁的线程也要进入阻塞状态**

![image-20220221121101529](E:\学习笔记\typora\img\image-20220221121101529.png)

解锁后会撤销可偏向状态。

**自旋优化：**

线程不立马进入Block状态，先自旋重试，等待其他线程解锁

java6以后增加了自适应的自旋次数，自旋适合多核cpu

### 重量级锁

**锁膨胀：**

为Object对象申请一个Monitor，让Object的MarkWord指向重量级锁地址，自己进入Monitor的EnterList等待

用62bit空间存放指向内核给的重量级锁Mutex Lock的指针

**释放锁**

使用CAS操作将对象当前的Mark Word和线程中复制的Displaced Mark Word替换回来(依据Mark Word中锁记录指针是否还指向本线程的锁记录)，如果替换成功，则恢复无锁状态（01），否则说明有其他线程尝试获取该锁(此时锁已膨胀)，那就要在释放锁的同时，唤醒被挂起的线程。

## 同步代码块与同步方法

对于同步方法，JVM采用ACC_SYNCHRONIZED标记符来实现同步。 

对于同步代码块。JVM采用monitorenter、monitorexit两个指令来实现同步。

monitorenter：将lock对象的MarkWord置为Monitor指针

monitorenter：将lock对象的MarkWord重置，唤醒EntryList

synchronized是无法禁止指令重排和处理器优化的，但是它能保证有序性，因为它是排他锁，只有一个线程在工作。

*对对象加锁别忘了final修饰*
