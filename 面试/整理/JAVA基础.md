### HashMap实现(蚂蚁金服)

引用Java基础中的HashMap [HashMap.md](../../JAVA基础/HashMap.md) 

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
* TreeMap(默认按照key升序排序), LinkedHashMap(记录了插入key的顺序)是有序的