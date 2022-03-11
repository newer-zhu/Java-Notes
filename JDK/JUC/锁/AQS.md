## 概述

是阻塞式锁和相关同步器工具的框架，如ReentrantLock内部就持有这个同步器，具体实现都是调用同步器的API

```java
==========================ReentrantLock成员变量================================
	private final ReentrantLock.Sync sync;

    public ReentrantLock() {//默认非公平
        this.sync = new ReentrantLock.NonfairSync();
    }
==========================Sync继承自AQS========================================
	abstract static class Sync extends AbstractQueuedSynchronizer
```



![image-20220227201347010](E:\学习笔记\typora\img\image-20220227201347010.png)

**三个组件：State，ConditionObject，CHL队列**

**AQS底层调用Park和unPark来实现线程暂停**

**条件队列是单向链表（ConditionObject里的队列），CHL队列（AQS里的队列）是双向链表。**

**采用模板模式：tryAcquire是需要子类实现的，而Acquire可以使用AQS提供的实现**

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

