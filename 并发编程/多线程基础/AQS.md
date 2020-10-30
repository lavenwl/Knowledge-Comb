### 什么是AQS

AQS是一个同步工具, 也是Lock用来实现线程同步的核心组件. 

AQS内部维护了一个FIFO的双向链表, 两个指针分别指向直接的后续节点和直接前去节点. 所以双向链表可以从任意一个节点开始访问前驱和后继. 每个Node其实是由线程封装, 当争抢锁失败后会封装成Node加入到AQS队列中去, 当获取锁的线程释放锁后, 会在队列中唤醒一个阻塞的节点.

### AQS源码分析(加锁过程)

`NofairSync.lock`

1. CAS获取锁, 如果成功, 则获取锁成功
2. CAS失败, 则调用acquire(1)走锁竞争的逻辑

> CAS原理: 这个操作是原子性的,, 不存在线程安全问题, 设计到Unsafe类的操作, 以及state, 这个属性的意义
>
> state: 是AQS中的一个属性, 他在不同的实现中所表达的含义不一样, 对于重入锁的实现来说, 他有两个含义
>
> 1. state = 0 表示无锁状态
> 2. state > 0 表示已经有线程获取到锁, 因为是重入锁, 所以 > 0
>
> Unsafe可以认为是Java中留下的一个后门, 可以提供一些低层次的操作, 如直接内存访问, 线程的挂起和恢复, CAS, 线程同步, 内存屏障等

`AQS.acquire() `

这个方法是AQS中的, 具体逻辑如下

1. 通过tryAcquire尝试获取独占锁, 如果成功返回true, 失败返回false;
2. 如果失败, 则通过addWaiter方法, 将当前线程封装成Node添加到AQS队列尾部
3. acquireQueued, 将Node作为参数, 通过自旋去尝试获取锁.

`NonfairSync.tryAquire()`

这里重写了AQS的tryAcquire方法, 具体调用了nonfairTryAcquire()

`Sync.nofairTryAcquire()`

1. 获取当前线程, 判断state状态
2. 如果state = 0, 表示当前是无锁状态, 通过CAS更新state状态
3. 如果当前属于重入, 则增加重入次数

`AQS.addWaiter()`

将当前线程封装为Node, 把node添加到同步队列

> 如果队列为空, 直接执行enq(), 添加队列, 如果队列不为空, 则放到队列末尾, 维护好双向链表指针

`enq()`

是通过自旋操作, 吧当前节点加入到队列中

`AQS.acuireQueued()`

通过addWaiter方法把线程添加到链表后, 会把Node作为参数传递到acuireQueued方法, 去竞争锁

1. 获取当前节点的前一个节点 prev
2. 如果prev节点为head节点, 那么它就有资格去争抢锁, 调用tryAquire.
3. 抢占成功后, 把或诶渡口的锁的节点设置为head, 移除原来的head节点
4. 如果获取锁失败, 则根据waitStatus决定是否需要挂起线程
5. 最后通过cancelAcquire取消获得锁的操作

`shouldParkAfterFailedAcquire`

如果ThreadA没有释放锁, 那么ThreadB与ThreadC在争抢锁中会失败, 一但失败, 会调用这个方法

> Node有5种状态
>
> 1. CANCELLED (1) : 同步队列中的线程等待超时或被中断, 需要从同步队列中取消Node节点, 器节点的waitStatus为CANCELLED, 及结束状态, 进入该状态后的节点将不会有变化.
> 2. 默认 (0) : 初始状态
> 3. SIGNAL (-1) : 只要前置节点释放锁, 就会通知标识为此状态的后续节点线程
> 4. CONDITION (-2) : 与Condition有关系
> 5. PROPAGATE (-3) : 共享模式下, 此状态线程处于可运行状态

1. 如果当前线程的前置节点的状态为SIGNAL, 那就标识可以放心挂起当前线程
2. 通过循环扫描, 把状态为CANCELLED状态的节点删除
3. 修改前置节点状态为SIGNAL, 返回false

`parkAndCheckInterrupt`

使用LockSupport.park挂起当前线程编程WAITING状态

### AQS源码分析(释放锁过程)

ReentrantLock.unlock

里面会调用Sync.release