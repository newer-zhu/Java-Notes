**注意事项：**

```
读-读不互斥，读-写互斥，写-写互斥
先获取读锁再获取写锁会导致永久等待写锁
先获得写锁再获取读锁可以
读锁不能使用条件变量
```

## 原理

### t1 w.lock() , t2 r.lock()

![image-20220301190540913](E:\学习笔记\typora\img\image-20220301190540913.png)

```java
//WritelLock的Lock方法，调用的AQS方法
public void lock() {
            this.sync.acquire(1);
        }

public final void acquire(int arg) {
        if (!this.tryAcquire(arg) && this.acquireQueued(this.addWaiter(AbstractQueuedSynchronizer.Node.EXCLUSIVE), arg)) {
            selfInterrupt();
        }
    }

//ReentrantReadWriteLock重写了tryAcquire方法，此方法里没有对读锁状态的判断，证明了获取读锁后可以获取写锁
protected final boolean tryAcquire(int acquires) {
            Thread current = Thread.currentThread();
    		//锁标记
            int c = this.getState();
    		//写锁标记
            int w = exclusiveCount(c);
    		//加了写锁或读锁
            if (c != 0) {
                //自己加了写锁，此时锁重入
                if (w != 0 && current == this.getExclusiveOwnerThread()) {
                    if (w + exclusiveCount(acquires) > 65535) {
                        throw new Error("Maximum lock count exceeded");
                    } else {
                        //加锁
                        this.setState(c + acquires);
                        return true;
                    }
                } else {
                    return false;
                }
                //writerShouldBlock返回false，因为是非公平锁，不用检查队列。c==0说明此时没有任何锁
            } else if (!this.writerShouldBlock() && this.compareAndSetState(c, c + acquires)) {
                this.setExclusiveOwnerThread(current);
                return true;
            } else {
                return false;
            }
        }

final boolean acquireQueued(AbstractQueuedSynchronizer.Node node, int arg) {
        boolean interrupted = false;

        try {
            while(true) {
                AbstractQueuedSynchronizer.Node p = node.predecessor();
                if (p == this.head && this.tryAcquire(arg)) {
                    this.setHead(node);
                    p.next = null;
                    return interrupted;
                }

                if (shouldParkAfterFailedAcquire(p, node)) {
                    //加锁失败，在这里park
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
```

```java
//ReadLock的加锁方法，调用AQS的acquireShared
public void lock() {
            this.sync.acquireShared(1);
        }

public final void acquireShared(int arg) {
        if (this.tryAcquireShared(arg) < 0) {
            this.doAcquireShared(arg);
        }

    }
//ReentrantReadWriteLock重写了tryAcquireShared方法
//返回-1失败，正数成功，数值表示还有几个后继节点需要唤醒
protected final int tryAcquireShared(int unused) {
            Thread current = Thread.currentThread();
            int c = this.getState();
    		//写锁被别人加上了，失败（证明加了写锁后就不能获取读锁）
            if (exclusiveCount(c) != 0 && this.getExclusiveOwnerThread() != current) {
                return -1;
            } else {
                int r = sharedCount(c);//高16位的读锁标记
                if (!this.readerShouldBlock() && r < 65535 && this.compareAndSetState(c, c + 65536)) {
                    if (r == 0) {
                        this.firstReader = current;
                        this.firstReaderHoldCount = 1;
                    } else if (this.firstReader == current) {
                        ++this.firstReaderHoldCount;
                    } else {
                        ReentrantReadWriteLock.Sync.HoldCounter rh = this.cachedHoldCounter;
                        if (rh != null && rh.tid == LockSupport.getThreadId(current)) {
                            if (rh.count == 0) {
                                this.readHolds.set(rh);
                            }
                        } else {
                            this.cachedHoldCounter = rh = (ReentrantReadWriteLock.Sync.HoldCounter)this.readHolds.get();
                        }

                        ++rh.count;
                    }
					//加锁成功
                    return 1;
                } else {
                    return this.fullTryAcquireShared(current);
                }
            }
        }

private void doAcquireShared(int arg) {
    //把节点加到队列，是SHARED共享类型的节点
        AbstractQueuedSynchronizer.Node node = this.addWaiter(AbstractQueuedSynchronizer.Node.SHARED);
        boolean interrupted = false;

        try {
            while(true) {
                AbstractQueuedSynchronizer.Node p = node.predecessor();
                if (p == this.head) {//node此时是在队列中第二个节点
                    int r = this.tryAcquireShared(arg);
                    if (r >= 0) {
                        //替换头节点
                        this.setHeadAndPropagate(node, r);
                        p.next = null;
                        return;
                    }
                }
				//shouldParkAfterFailedAcquire是否应该park，第一次判断返回false，一个循环后还没acquire成功返回true
                if (shouldParkAfterFailedAcquire(p, node)) {
                    interrupted |= this.parkAndCheckInterrupt();
                }
            }
        } catch (Throwable var9) {
            this.cancelAcquire(node);
            throw var9;
        } finally {
            if (interrupted) {
                selfInterrupt();
            }

        }
    }

private void setHeadAndPropagate(AbstractQueuedSynchronizer.Node node, int propagate) {
        AbstractQueuedSynchronizer.Node h = this.head;
        this.setHead(node);
        if (propagate > 0 || h == null || h.waitStatus < 0 || (h = this.head) == null || h.waitStatus < 0) {
            AbstractQueuedSynchronizer.Node s = node.next;
            if (s == null || s.isShared()) {
                this.doReleaseShared();
            }
        }

    }

private void doReleaseShared() {
        while(true) {
            AbstractQueuedSynchronizer.Node h = this.head;
            if (h != null && h != this.tail) {
                int ws = h.waitStatus;
                if (ws == -1) {//把状态由-1 cas 修改为 0
                    if (!h.compareAndSetWaitStatus(-1, 0)) {
                        continue;
                    }
					//唤醒后继节点
                    this.unparkSuccessor(h);
                } else if (ws == 0 && !h.compareAndSetWaitStatus(0, -3)) {
                    continue;
                }
            }

            if (h == this.head) {
                return;
            }
        }
    }
```

### t3 r.lock(), t4 w.lock()

![image-20220301210417805](E:\学习笔记\typora\img\image-20220301210417805.png)

### t1.unlock()

```java
//unlock会调用AQS的release
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
//ReentrantReadWriteLock重写了tryRelease
protected final boolean tryRelease(int releases) {
            if (!this.isHeldExclusively()) {
                throw new IllegalMonitorStateException();
            } else {
                int nextc = this.getState() - releases;
                //获取写锁状态位
                boolean free = exclusiveCount(nextc) == 0;
                if (free) {//解锁成功，设置线程为null
                    this.setExclusiveOwnerThread((Thread)null);
                }
				//重入次数-1，设置标记位
                this.setState(nextc);
                return free;
            }
        }
```

由于t2和t3都是读锁，在队列中是Shared类型，会一直向后唤醒到最后一个非Shared节点

![image-20220301212522694](E:\学习笔记\typora\img\image-20220301212522694.png)

![image-20220301212551817](E:\学习笔记\typora\img\image-20220301212551817.png)

### t2.unlock(), t3.unlock()

![image-20220301213342531](E:\学习笔记\typora\img\image-20220301213342531.png)

```java
//ReadLock的unlock调用AQS的releaseShared
public final boolean releaseShared(int arg) {
        if (this.tryReleaseShared(arg)) {
            this.doReleaseShared();
            return true;
        } else {
            return false;
        }
    }
//子类实现的tryReleaseShared
protected final boolean tryReleaseShared(int unused) {
            Thread current = Thread.currentThread();
            int nextc;
            if (this.firstReader == current) {
                if (this.firstReaderHoldCount == 1) {
                    this.firstReader = null;
                } else {
                    --this.firstReaderHoldCount;
                }
            } else {
                ReentrantReadWriteLock.Sync.HoldCounter rh = this.cachedHoldCounter;
                if (rh == null || rh.tid != LockSupport.getThreadId(current)) {
                    rh = (ReentrantReadWriteLock.Sync.HoldCounter)this.readHolds.get();
                }

                nextc = rh.count;
                if (nextc <= 1) {
                    this.readHolds.remove();
                    if (nextc <= 0) {
                        throw unmatchedUnlockException();
                    }
                }

                --rh.count;
            }

            int c;
            do {//把读锁标记-高位的1
                c = this.getState();
                nextc = c - 65536;
            } while(!this.compareAndSetState(c, nextc));
			//返回是否已无读锁，此时nextc是1
            return nextc == 0;
        }
//AQS中的方法
private void doReleaseShared() {
        while(true) {
            AbstractQueuedSynchronizer.Node h = this.head;
            if (h != null && h != this.tail) {
                int ws = h.waitStatus;
                if (ws == -1) {
                    if (!h.compareAndSetWaitStatus(-1, 0)) {
                        continue;
                    }

                    this.unparkSuccessor(h);
                } else if (ws == 0 && !h.compareAndSetWaitStatus(0, -3)) {
                    continue;
                }
            }

            if (h == this.head) {
                return;
            }
        }
    }
```

![image-20220301213951110](E:\学习笔记\typora\img\image-20220301213951110.png)
