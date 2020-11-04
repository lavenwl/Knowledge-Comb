### HashMap实现(蚂蚁金服)

引用Java基础中的HashMap [HashMap.md](../../JAVA基础/HashMap.md) 

### hashmap put get 接口 集合类名 

node数组

### hashmap扩容机制

1. 什么时候扩容: 当新增元素的时候, 会判断当前容器的长度是否满足条件, 当>=总长度*阈值的时候, 会触resize()方法执行扩容机制
2. 新建一个容量为原来容量两倍的数组来存放新的数据, 然后对每一个桶内的链表进行迭代, 采用头插的方式将数据放入新的数组及链表中.
3. 1.8中做了优化, 1.7扩容后链表顺序会倒置,但1.8, 不会, 1.7会rehash, 1.8,采用位与运算来解决扩容索引定位问题, 不需要rehash.



### ConccurentHashMap 1.7、1.8实现(蚂蚁金服)

引用Java基础中的HashMap [HashMap.md](../../JAVA基础/HashMap.md) 

### IO与NIO的区别

IO, 为BIO, 为阻塞IO, NIO为 New IO, 是非阻塞IO, NIO是JDK1.4引入, 面向buffer, 而IO则是面向流.



参考[网页](https://blog.csdn.net/zengxiantao1994/article/details/88094910?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.add_param_isCf)

### ArrayList与LinkedList哪个是线程安全的

两个都不是线程安全的, 有个集合是Vector, 是线程安全的, 他在add方法添加了synchronized关键字, 保证了线程安全.

### HashMap既然线程不安全, 那为什么我们没有遇到相关的问题?





### 技术问题ArrayList与LinkedList的区别, ArrayList的内部数组的类型, 扩容机制

用户在追加数据时, Java会先计算容量(Capacity)是否合适, 如果不满足, 则创建一个新的数组, 将现有数组拷贝到新数组中, 新数组的长度是现有数组的1.5倍. 然后将原数组的变量指向新的数组. 同事size值+1, 在删除对象时, 将制定位置的后面对象往前移动一位, 然后把最后一位置空, 对象由垃圾回收期自动回收.

### HashMap的原理, 是否有序, 有序的集合是什么

* HashMap, HashTable是无序的
* TreeMap(默认按照key升序), LinkedHashMap(记录了插入key的顺序)是有序的

### 动态代理与静态代理

> 在讲解前我们先了解一下代理模式, 代理模式有三种角色: 代理接口(Subject), 代理类(ProxySubject), 委托类(RealSubject), 三种角色形成一个品字形的结构. 根据代理生成的时间不同, 可以分为静态代理和动态代理. 

* 静态代理

    所谓静态, 就是在程序运行前就已经存在了代理类字节码文件, 代理类和委托类的关系在运行前就确定了.

    > 优点: 业务类只需要关注业务逻辑本身, 保证了业务类的重用性, 这是代理的共同优点
    >
    > 缺点:
    >
    > 1. 代理对象的一个借口只服务于一种类型的对象, 如果代理的方法很多, 势必要求每一个方法进行代理, 静态代理的程序规模过大就无法胜任了
    > 2. 如果接口增加一个方法, 除了实现类需要实现这个方法外, 所有的代理类也需要实现这个方法.增加了代码的维护复杂度.

* 动态代理

    动态代理类的源码是在程序运行期间有JVM根据反射等级制动态生成的, 所以不存在代理类的字节码文件, 代理类和委托类的关系是在程序运行时确认的.

    > - **优点：**动态代理与静态代理相比较，最大的好处是接口中声明的所有方法都被转移到调用处理器一个集中的方法中处理（InvocationHandler.invoke）。这样，在接口方法数量比较多的时候，我们可以进行灵活处理，而不需要像静态代理那样每一个方法进行中转。在本示例中看不出来，因为invoke方法体内嵌入了具体的外围业务（记录任务处理前后时间并计算时间差），实际中可以类似Spring AOP那样配置外围业务。 
    > - **缺点：**诚然，Proxy 已经设计得非常优美，但是还是有一点点小小的遗憾之处，那就是它始终无法摆脱仅支持 interface 代理的桎梏，因为它的设计注定了这个遗憾。回想一下那些动态生成的代理类的继承关系图，它们已经注定有一个共同的父类叫 Proxy。Java 的继承机制注定了这些动态代理类们无法实现对 class 的动态代理，原因是多继承在 Java 中本质上就行不通。 

