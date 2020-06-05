参考文档: [笔记-20190511-多线程的基本原理及挑战.pdf](source/笔记-20190511-多线程的基本原理及挑战.pdf) 

### Synchronized

> synchronized是重量级锁, JavaSE 1.6 对其进行了各种优化.引入了偏向锁, 与轻量级锁.

#### 基本语法

synchronized有三种方式来加锁:

* 修饰实例方法: 作用域当前实例, 获取当前实例的锁 `实例锁`
* 静态方法: 作用域当前类对象 `类锁`
* 修饰代码块: 需要制定锁对象

### 锁的存储

* 在Hotspot虚拟机中, 对象在内存中的布局可以分为三个区域:
    * 对象头(Header)
    * 实例数据(Instance Data)
    * 对齐填充(Padding)
* 使用new 创建一个对象实例时候, JVM(Hotspot)层面实际上会创建一个instanceOopDesc对象
* instanceOopDesc对象继承自oopDesc
* oopDesc的定义包含两个成员, _mark, _metadata
* _mark 表示对象标记, 属于markOop类型, 也就是MarkWorld, 它记录了对象和锁的相关信息

| 锁状态   | 23bit  | 2bit     | 4bit       | 是否偏向锁(1bit) | 锁标志位 |
| -------- | ------ | -------- | ---------- | ---------------- | -------- |
| 无锁     | 对象的 | HashCode | 分代年龄   | 0                | 01       |
| 偏向锁   | 线程ID | Epoch    | 分代年龄   | 1                | 01       |
| 轻量级锁 | 指向   | 栈中锁   | 记录的指针 |                  | 00       |
| 重量级锁 | 指向   | 重量级   | 锁指针     |                  | 10       |
| GC标记   | 空     |          |            |                  | 11       |

![image-20200605084700935](多线程挑战(synchronized).assets/image-20200605084700935.png)

### 为什么所有的对象都可以实现锁

* 首先, java中没腾出对象都派生一个自Object类, 二每个JavaObject在JVM内部都有一个native的c++对象oop/oopDesc进行对应
* 线程在获得锁的时候就是获取一个监视器对象(monitor), monitor可以认为是一个同步对象,所有的java对象是天生携带monitor. 在hostpot源码的markOop.hpp文件中.

### Synchronized锁升级

在JDK 1.6之前, synchronized还是重量级锁, 虽然保证了线程的安全, 但是也带来了性能的开销, 真正的程序出现多线程抢占的情况又不是一直发生, 所以在JDK 1.6 之后对synchronized锁进行了优化, 提供了synchronized锁升级的机制.在synchronized中的锁存在四种状态: `无锁`, `偏向锁`, `轻量级锁`, `重量级锁`.

### 偏向锁的基本原理

鉴于大部分情况下, 共享资源多是由同一个线程重复获得,  所以.当一个线程访问加了同步锁的代码块时, 会在对象头中存储当前线程的ID, 后续这个线程进入和退出这段加了同步锁的代码块时, 不需要再次加锁与释放锁, 而是直接比较对象头里面是否存储了指向当前线程的偏向锁. **如果相等, 表示偏向锁偏向于当前线程, 不需要重复获取锁**.

> 在我们的应用开发中, 绝大部分情况下会存在两个以上的线程竞争, 那么如果开启偏向锁, 反而会提升获取锁的资源消耗, 所以可以通过jvm参数 `UseBiasedLocking `来设置开启或者关闭偏向锁.

![image-20200605085715576](多线程挑战(synchronized).assets/image-20200605085715576.png)

### 升级为轻量级锁的过程

1. 线程在自己的栈帧中创建锁记录 LockRecord.
2. 将锁对象的对象头中的MarkWord复制到线创建的LockRecord中
3. 将锁对象中的Owner指针指向锁对象
4. 将锁对象的对象头的MarkWord替换为指向所记录的指针.

![image-20200605092337302](多线程挑战(synchronized).assets/image-20200605092337302.png)



![image-20200605092346953](多线程挑战(synchronized).assets/image-20200605092346953.png)

![image-20200605092808127](多线程挑战(synchronized).assets/image-20200605092808127.png)



### 重量级锁的加锁流程

![image-20200605093135358](多线程挑战(synchronized).assets/image-20200605093135358.png)









































