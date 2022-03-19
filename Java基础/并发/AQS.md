# 												AQS

## 一、作用

ReentrantLock、CountDownLatch、ReentrantReadWriteLock等锁结构的底层实现

## 二、重点方法、变量含义

state：共享锁表示剩余的共享个数；独占锁表示独占线程的重入次数

| 共享锁方法      | 独占锁方法 | 作用                                                        |
| --------------- | ---------- | ----------------------------------------------------------- |
| acquireShared   | acquire    | 申请占有锁                                                  |
| releaseShared   | release    | 申请释放锁                                                  |
| tryAcquireShard | tryAcquire | 在acquire/acquireShared内部会调用这两个方法，需要在继承重写 |
| tryReleaseShard | tryRelease | 在release/releaseShard内部会调用这两个方法，需要在继承重写  |
|                 |            |                                                             |
|                 |            |                                                             |
|                 |            |                                                             |
|                 |            |                                                             |



## 三、实现原理

### 1、基本原理

是一个类似双向链表的结构，新加入的线程会作为一个节点，加在链表尾部。

当前面的链表节点执行完成，会把后面的节点设置为head。

每个等待中的线程都会在一个for循环中，不停地轮询自己是否为head，当自己为head时，执行后续操作。

所有设置head、修改state等方法都是通过cas实现的，以保证并发。

### 2、公平锁与非公平锁

公平锁直接会将当前线程置于链表尾部；

非公平锁会先试图抢占锁，失败后，才会加入链表尾部

### 3、CountDownLatch

根据构造函数传入的count，设置state

每次countdown都会将state-1

在await时，会在链表最后加入一个Node节点。自旋等待触发。
