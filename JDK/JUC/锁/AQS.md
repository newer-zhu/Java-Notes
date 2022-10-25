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

