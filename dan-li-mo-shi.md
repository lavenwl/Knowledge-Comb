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
// 单线程下的懒汉式单例模式代码, 多线程存在并发问题
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

// 多线程下的单例模式代码，锁定了整个方法，性能下降
public class LazySimpleSingleton{
    private static final LazySimpleSingleton instance;

    private LazySimpleSingleton() {}

    // 通过锁定方法实现
    public synchronized static LazySimpleSingleton getInstance() {
        if (instance == null) {
            instance = new LazySimpleSingleton();
        }
        return instance;
    }

    // 通过锁定代码块实现
    public static LazySimpleSingleton getInstance2() {
        synchronized(LazySimpleSingleton.class) {
            if (instance == null) {
                instance = new LazySimpleSingleton();
            }
            return instance;
        }
    }
}

// 双重检查锁单例模式， 一定程度提高了性能
public class LazyDoubleCheckSingleton{
    private static final LazyDoubleCheckSingletion instance;

    private LazyDoubleCheckSingletion(){}

    public static LazyDoubleCheckSingleton getInstacne() {
        if(instance == null) { // 判断是否加锁， 提高性能
            synchronized(LazyDoubleCheckSingleton.class) {
                if(instance == null) { // 判断是否创建， 保证单例
                    instance = new LazyDoubleCheckSingleton();
                }
            }
        }
        return instance;
    }   
}

// 静态内部类单例模式，利用java特性，避免浪费内存和synchronized的性能问题
public class LazyInnerClassSingleton{
    private static final LazyInnerClassSingleton instance;

    private LazyInnerClassSingleton() {}

    public static LazyInnerClassSingleton getInstance() {
        return InnerClass.instance;        
    }

    // 默认不加载
    private static class InnerClass{
        private static final LazyInnerClassSingleton instance = new LazyInnerClassSingleton();
    }
}
```

### 如何解决反射破坏单例的问题

```java
// 以静态内部类实现方式为例， 解决反射破坏单例的问题，只需要在构造器内添加判断
public class LazyInnerClassSingleton{
    private static final LazyInnerClassSingleton instance;

    private LazyInnerClassSingleton() {
        if (InnerClass.instance != null) {
            throw new RuntimeException("不可以e创建多个实例")
        }
    }

    public static LazyInnerClassSingleton getInstance() {
        return InnerClass.instance;        
    }

    // 默认不加载
    private static class InnerClass{
        private static final LazyInnerClassSingleton instance = new LazyInnerClassSingleton();
    }
}
```

### 如何解决序列化破坏单例的问题

> 新增readResolve\(\)方法返回实例解决了单例模式被破坏的问题，但是实际上实例化了两次， 只不过新创建的对象没有被返回而已。

```java
// 以懒汉式单例模式代码为例说明如何解决序列化破坏单例的问题， 重写一个方法
public class HungrySingleton{
    private static final HungrySingleton instance = new HungrySingletion();

    private HungrySingleton() {}

    public static HungrySingleton getInstance() {
        return instance;
    }

    private Object readResolve(){
        return instance;
    }
}
```

### 注册式单例模式

注册式单例模式分为两种：枚举式单例模式和容器式单例模式。

* 枚举式单例模式

> 枚举式单例模式在静态代码块中给INSTANCE复制，是饿汉单例模式
>
> JDK底层对枚举类型做了特殊判断，保证序列化与反射不会影响单例实现

```java
public enum EnumSingleton{
    INSTANCE;
    
    private Object data;
    
    public Object getData(){
        return data;
    }
    
    public void setData(Object data) {
        this.data = data;
    }
    
    public static EnumSingleton getInctance() {
        return INSTANCE;
    }
}
```

* 容器式单例模式

> 适用于实例非常多的情况，便于管理
>
> 但他是非线程安全的。

```java
public class ContainerSingleton {
    private ContainerSingleton() {}
    private static Map<String, Object> ioc = new ConcurrentHashMap<String, Object>();
    public static Object getBean(String, className) {
        synchronized (ioc) {
            if(!ioc.containsKey(className)) {
                Object obj = null;
                try{
                    obj = Class.forName(className).newInstance();
                    ioc.put(className, obj);
                } catch(Exception e) {
                    e.printStackTrace();
                }
                return obj;
            } else {
                return ioc.get(className);
            }
        }
    }
}
```



