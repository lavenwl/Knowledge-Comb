HashMap

hashmap是Java集合类, 以Key-Value形式存储数据.允许为null

jdk1.8之前底层结构是以数组加链表实现

> 数组: 查询效率高, 但插入与删除效率低
>
> 链表: 查询效率低, 但插入与删除效率高

jdk1.8及之后如果链表长度大于8, 则将链表转换为红黑树结构存储

> 当链表数量大于8时会转换为红黑树结构存储
>
> 当链表数量小于6时会转回链表格式存储
>
> 红黑树: 是一个相对平衡的二叉查找树

如果不指定hashmap数组的默认长度是16, 如果数组的占用比例大于默认负载因子0.75, 则自动扩容.

> 数组的长度是2的幂次. 即使用户自己制定, 最终的大小也是可以满足用户制定数量的最小2幂次.
>
> 扩容会重新计算下标索引.消耗性能, 可以对已知大小的集合设置初始长度

hashmap中的数组各元素称为桶, 对应的数组下标称为桶号, 通过对key值进行哈希计算得到hash值后对数组长度做类似取模计算得到具体的桶号

> 哈希算法: hash(key); 也称散列算法
>
> 桶号计算: index(hash, N+1)
>
> 真实的计算并不是取模, 而是位运算实现

在添加数据的时候会出现`哈希碰撞与下标冲突`

> 哈希碰撞: 如果两个对象不同, 但是哈希值相同, 则出现哈希冲突
>
> 下标冲突: 有两个不同的对象进行哈希运算并确认的下标相同, 则为下标冲突
>
> 关系: 下标冲突不一定是哈希碰撞, 哈希碰撞一定是下标冲突

散列表可以解决哈希碰撞与下标冲突的问题

> 散列表: 数组+链表
>
> 1.8之前, 新插入的元素都是放在了链表的头部位置, 但这种操作在高并发的环境下容易死锁
>
> 1.8之后, 新插入的元素放在了链表的尾部

HashMap有四个构造方法

```java
HashMap()    //无参构造方法
HashMap(int initialCapacity)  //指定初始容量的构造方法 
HashMap(int initialCapacity, float loadFactor) //指定初始容量和负载因子
HashMap(Map<? extends K,? extends V> m)  //指定集合，转化为HashMap
```

与hashtable的区别

* hashMap是非线程安全的, hashTable是现成安全的

    > hashtable实现方法里面都添加了synchronized关键字来保证线程同步, 效率比hashmap慢, 建议使用hashmap如果需要在多线程环境下使用hashmap需要使用`Collections.synchronizedMap()`方法获取一个线程安全的集合

* HashMap可以使用null作为key, HashTable不可以使用null作为Key

    > HashMap以null作为key时, 总是存储在table数组的第一个节点上

* HashMap是对Map接口的实现, HashTable实现了Map接口和Dictionary抽象类

* HashMap的初始化容量为16, HashTable的初始化容量为11, 两者默认都是0.75

* HashMap扩容时是当前容量翻倍capacity*2, HashTable扩容时是容量翻倍+1

* HashTable计算hash是直接使用key的hashcode对数组的长度取模, HashMap是二次取模.

HashMap为什么线程不安全

调用resize()方法的时候会出现环形链表......



concurrentHashMap [参考](https://blog.csdn.net/weixin_44460333/article/details/86770169)



