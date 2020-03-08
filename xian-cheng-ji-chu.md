实现多线程的方式

> 继承Thread类，实现Runable接口，使用ExecutorService，Callable，Future实现带有结果返回的多线程

* 继承Thread类创建线程

```java
public class MyThread extends Thread {
    public void run() {
        System.out.println("MyThread.run()");
    }
}

MyThread t1 = new MyThread();
MyThread t2 = new MyThread();
t1.start();
t2.start();
```



