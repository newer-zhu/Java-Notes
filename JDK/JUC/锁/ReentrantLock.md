## 特点

![image-20220222190826355](E:\学习笔记\typora\img\image-20220222190826355.png)

### 概念

**不可打断模式**：在未获得锁的时候被打断仅会加上打断标记，在未获得锁的过程中对打断置之不理，继续驻留在队列。获得锁后会继续运行，只是打断标记变为true

**可打断模式**：尝试获得锁的过程中被打断就不会进入for循环，会终止获得锁的行为，直接抛出异常

**非公平锁**：不检查AQS队列，直接加锁

**公平锁**：先检查AQS队列，如果队列中没有第二或者第二节点不是当前线程则不会去竞争锁

ReentrantLock获取锁后执行的代码在try块中执行, Lock比Synchronize更广泛，且需要手动加锁解锁。发生异常时要在finally中释放锁，否则会造成死锁.

### 特定唤醒

通过标志位flag和condition来做到指定唤醒某个线程

```java
class Share{
    private int count = 0;
    private Lock lock = new ReentrantLock();
    private Condition c1 = lock.newCondition();
    private Condition c2 = lock.newCondition();
    private int flag = 1;

    public void a1() throws InterruptedException {
        lock.lock();
        try {
            while (flag != 1){
                c1.await();
            }
            System.out.println("do a1........");
            flag = 2;
            c2.signal();
        }finally {
            lock.unlock();
        }
    }

    public void a2() throws InterruptedException {
        lock.lock();
        try {
            while (flag != 2){
                c2.await();
            }
            System.out.println("do a2........");
            flag = 1;
            c1.signal();
        }finally {
            lock.unlock();
        }
    }
}
```



**可中断**

```java
try {
    //如果获取失败进入阻塞队列，这时可以被其他线程interrupt打断
    reentrantLock.lockInterruptibly();
} catch (InterruptedException e) {
    //打断后操作
    e.printStackTrace();
}
```

**锁超时**

尝试获取锁，参数可指定等待的时间

```java
reentrantLock.tryLock();
```

**公平锁**

默认不公平，公平锁会降低并发度，非必要不使用

```java
public ReentrantLock() {
    this.sync = new ReentrantLock.NonfairSync();
}
```

**条件变量**

相当于把waitSet按条件分组了，可以按条件特定signal或signalAll

## 源码分析

### 非公平锁

**Lock**

```java
public void lock() {
    	//lock()调用的AQS中的acquire方法
        this.sync.acquire(1);
    }
//AQS的acquire方法
public final void acquire(int arg) {
        if (!this.tryAcquire(arg) && this.acquireQueued(this.addWaiter(AbstractQueuedSynchronizer.Node.EXCLUSIVE), arg)) {
            selfInterrupt();
        }
    }
```

第一次出现竞争时

![image-20220227204610470](E:\学习笔记\typora\img\image-20220227204610470.png)

```java
final boolean acquireQueued(AbstractQueuedSynchronizer.Node node, int arg) {
        boolean interrupted = false;

        try {
            while(true) {
                //队列中此节点的前驱节点
                AbstractQueuedSynchronizer.Node p = node.predecessor();
                //如果被其前驱节点唤醒并成功获取了锁，就把自己设为头节点，断开其后继节点
                if (p == this.head && this.tryAcquire(arg)) {
                    this.setHead(node);
                    p.next = null;
                    //此时为true
                    return interrupted;
                }
				//将前驱节点状态改成-1，此时条件为false，下一次进入时才未true，-1表示此节点负责唤醒后继节点
                if (shouldParkAfterFailedAcquire(p, node)) {
                    interrupted |= this.parkAndCheckInterrupt();
                }
            }
        } catch (Throwable var5) {
            this.cancelAcquire(node);
            if (interrupted) {
                selfInterrupt();
            }

            throw var5;
        }
    }
//此方法在ReentrantLock中
protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            int c = this.getState();
            if (c == 0) {
                if (!this.hasQueuedPredecessors() && this.compareAndSetState(0, acquires)) {
                    this.setExclusiveOwnerThread(current);
                    return true;
                }//这里实现可重入
            } else if (current == this.getExclusiveOwnerThread()) {
                //state++
                int nextc = c + acquires;
                if (nextc < 0) {
                    throw new Error("Maximum lock count exceeded");
                }

                this.setState(nextc);
                return true;
            }

            return false;
        }

private static boolean shouldParkAfterFailedAcquire(AbstractQueuedSynchronizer.Node pred, AbstractQueuedSynchronizer.Node node) {
        int ws = pred.waitStatus;
        if (ws == -1) {
            return true;
        } else {
            if (ws > 0) {
                do {
                    node.prev = pred = pred.prev;
                } while(pred.waitStatus > 0);

                pred.next = node;
            } else {
                pred.compareAndSetWaitStatus(ws, -1);
            }

            return false;
        }
    }
//线程中止
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

![image-20220227212233673](E:\学习笔记\typora\img\image-20220227212233673.png)

**unlock()**

```java
//unlock调用AQS的release
public void unlock() {
        this.sync.release(1);
    }

//AQS的release方法
public final boolean release(int arg) {
        if (this.tryRelease(arg)) {
            AbstractQueuedSynchronizer.Node h = this.head;
            if (h != null && h.waitStatus != 0) {
                //唤醒后继节点
                this.unparkSuccessor(h);
            }

            return true;
        } else {
            return false;
        }
    }

//reentrantLock的tryRelease
protected final boolean tryRelease(int releases) {
    		//判断锁重入的参数，state-
            int c = this.getState() - releases;
            if (Thread.currentThread() != this.getExclusiveOwnerThread()) {
                throw new IllegalMonitorStateException();
            } else {
                //发生了锁重入，默认不释放锁
                boolean free = false;
                //c==0则释放锁
                if (c == 0) {
                    free = true;
                    //设置线程为null 
                   this.setExclusiveOwnerThread((Thread)null);
                }
			//设置状态
                this.setState(c);
                return free;
            }
        }

```

**可打断模式**

即使被打断，仍会驻留在AQS队列，等待获取锁后继续运行

不可打断模式

```java
//在AQS中
private void doAcquireInterruptibly(int arg) throws InterruptedException {
        AbstractQueuedSynchronizer.Node node = this.addWaiter(AbstractQueuedSynchronizer.Node.EXCLUSIVE);

        try {
            AbstractQueuedSynchronizer.Node p;
            do {
                p = node.predecessor();
                if (p == this.head && this.tryAcquire(arg)) {
                    this.setHead(node);
                    p.next = null;
                    return;
                }
            } while(!shouldParkAfterFailedAcquire(p, node) || !this.parkAndCheckInterrupt());
			//被interrupt时抛出异常，不会再进入循环
            throw new InterruptedException();
        } catch (Throwable var4) {
            this.cancelAcquire(node);
            throw var4;
        }
    }
```

**非公平锁**

```java
//在ReentrantLock中
final boolean nonfairTryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            int c = this.getState();
            if (c == 0) {
                if (this.compareAndSetState(0, acquires)) {
                    this.setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (current == this.getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) {
                    throw new Error("Maximum lock count exceeded");
                }

                this.setState(nextc);
                return true;
            }

            return false;
        }
```

**公平锁**

```java
//在ReentrantLock中
protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
            int c = this.getState();
            if (c == 0) {
                //先检查队列是否有前驱节点
                if (!this.hasQueuedPredecessors() && this.compareAndSetState(0, acquires)) {
                    this.setExclusiveOwnerThread(current);
                    return true;
                }
            } else if (current == this.getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) {
                    throw new Error("Maximum lock count exceeded");
                }

                this.setState(nextc);
                return true;
            }

            return false;
        }
```

### 条件变量

![image-20220228220437926](E:\学习笔记\typora\img\image-20220228220437926.png)

```java
//在AQS中，条件变量的awiat
public final void await() throws InterruptedException {
            if (Thread.interrupted()) {
                throw new InterruptedException();
            } else {
                //添加节点到条件变量的单链表等待队列中，此队列节点状态都是-2
                AbstractQueuedSynchronizer.Node node = this.addConditionWaiter();
                //释放掉当前持锁线程的所有重入锁
                int savedState = AbstractQueuedSynchronizer.this.fullyRelease(node);
                int interruptMode = 0;

                while(!AbstractQueuedSynchronizer.this.isOnSyncQueue(node)) {
                    LockSupport.park(this);
                    if ((interruptMode = this.checkInterruptWhileWaiting(node)) != 0) {
                        break;
                    }
                }

                if (AbstractQueuedSynchronizer.this.acquireQueued(node, savedState) && interruptMode != -1) {
                    interruptMode = 1;
                }

                if (node.nextWaiter != null) {
                    this.unlinkCancelledWaiters();
                }

                if (interruptMode != 0) {
                    this.reportInterruptAfterWait(interruptMode);
                }

            }
        }

		//signal
        public final void signal() {
                    if (!AbstractQueuedSynchronizer.this.isHeldExclusively()) {
                        throw new IllegalMonitorStateException();
                    } else {
                        //取出同步队列的头节点
                        AbstractQueuedSynchronizer.Node first = this.firstWaiter;
                        if (first != null) {
                            this.doSignal(first);
                        }

                    }
                }

        private void doSignal(AbstractQueuedSynchronizer.Node first) {
                    do {
                        //取出条件变量里同步队列的节点
                        if ((this.firstWaiter = first.nextWaiter) == null) {
                            this.lastWaiter = null;
                        }

                        first.nextWaiter = null;
                        //转移到AQS的等待队列中尾部，转移失败就取同步队列的下一个
                    } while(!AbstractQueuedSynchronizer.this.transferForSignal(first) && (first = this.firstWaiter) != null);

                }

        final boolean transferForSignal(AbstractQueuedSynchronizer.Node node) {
            if (!node.compareAndSetWaitStatus(-2, 0)) {
                return false;
            } else {//返回转移后的前驱节点
                AbstractQueuedSynchronizer.Node p = this.enq(node);
                int ws = p.waitStatus;
                //前驱节点状态，>0表示此节点线程持有锁，statu改成-1为false说明解锁失败
                if (ws > 0 || !p.compareAndSetWaitStatus(ws, -1)) {
                    LockSupport.unpark(node.thread);
                }

                return true;
            }
        }
```

