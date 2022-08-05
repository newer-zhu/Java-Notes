## 概述

是阻塞式锁和相关同步器工具的框架，如ReentrantLock内部就持有这个同步器，具体实现都是调用同步器的API

![image-20220227201347010](E:\学习笔记\typora\img\image-20220227201347010.png)

**三个组件：State，ConditionObject，CHL队列**

**AQS底层调用Park和unPark来实现线程暂停**

**条件队列是单向链表（ConditionObject里的队列），CHL队列（AQS里的队列）是双向链表。**

**采用模板模式：tryAcquire是需要子类实现的，而Acquire可以使用AQS提供的实现**

**节点的waitStatus**

```java
// CANCELLED：由于超时或中断，此节点被取消。节点一旦被取消了就不会再改变状态。特别是，取消节点的线程不会再阻塞。
static final int CANCELLED =  1;
// SIGNAL:此节点后面的节点已（或即将）被阻止（通过park），因此当前节点在释放或取消时必须断开后面的节点
// 为了避免竞争，acquire方法时前面的节点必须是SIGNAL状态，然后重试原子acquire，然后在失败时阻塞。
static final int SIGNAL = -1;
// 此节点当前在条件队列中。标记为CONDITION的节点会被移动到一个特殊的条件等待队列（此时状态将设置为0），直到条件时才会被重新移动到同步等待队列 。（此处使用此值与字段的其他用途无关，但简化了机制。）
static final int CONDITION = -2;
//传播：应将releaseShared传播到其他节点。这是在doReleaseShared中设置的（仅适用于头部节点），以确保传播继续，即使此后有其他操作介入。
static final int PROPAGATE = -3;
```

## 自定义锁（不可重入锁）

```java
class myLock implements Lock {

    //独占锁,同步器类
    class MySync extends AbstractQueuedSynchronizer {
        @Override//加锁
        protected boolean tryAcquire(int arg) {
            //state变量加了volatile
            if (compareAndSetState(0,1)){
                setExclusiveOwnerThread(Thread.currentThread());
            }
            return true;
        }

        @Override//解锁
        protected boolean tryRelease(int arg) {
            setExclusiveOwnerThread(null);
            //由于是独占锁，setState无需同步处理。但是必须在上一行之后，因为volatile保证前面的操作能被其他线程读取
            setState(0);
            return true;
        }

        @Override//是否持有独占锁
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        public Condition newCondition(){
            return new ConditionObject();
        }
    }

    MySync sync = new MySync();

    @Override//加锁
    public void lock() {
        sync.acquire(1);
    }

    @Override//加锁，可打断
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override//尝试加锁（一次）
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override//超时加锁
    public boolean tryLock(long l, TimeUnit timeUnit) throws InterruptedException {
        return sync.tryAcquireNanos(1,timeUnit.toNanos(l));
    }

    @Override//解锁
    public void unlock() {
        sync.release(1);
    }

    @Override//创建条件变量
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

