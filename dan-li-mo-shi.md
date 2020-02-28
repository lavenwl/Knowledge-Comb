### 什么是单例模式

是指确保一个类在任何情况下都绝对只有一个实例，并提供一个全局访问点。属于创建型模式

### 饿汉式单例模式

> 优点：没有任何锁，执行效率高
>
> 缺点：类加载的时候初始化，浪费内存空间
>
> 适用：单例对象较少的情况

```java
// 单例模式的第一种写法， 结构简单易懂
public class HungrySingleton{
    private static final HungrySingleton instance = new HungrySingletion();
    
    private HungrySingleton() {}
    
    public static HungrySingleton getInstance() {
        return instance;
    }
}

// 单例模式的第二种写法, 使用了静态代码块类加载时执行的特性
public class HungrySingleton{
    private static final HungrySingleton instance;
    
    static {
        instance = new HungrySingleton();
    }
    
    private HungrySingleton();
    
    public static HungrySingleton getInstance() {
        return instance;
    }
}
```

### 懒汉式单例模式

> 优点：调用时加载， 节省空间
>
> 缺点：
>
> 适用：

```java
// 单线程下的懒汉式单例模式代码
public class LazySimpleSingleton{
    private static final LazySingleton instance;
    
    private LazySimpleSingleton() {}
    
    public static LazySimpleSingleton getInstance() {
        if (instance == null) {
            instance = new LazySimpleSingleton();
        }
        return instance;
    }
}
```



