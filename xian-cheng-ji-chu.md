### 实现多线程的方式

> 继承Thread类，实现Runnable接口，使用ExecutorService，Callable，Future实现带有结果返回的多线程

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

* 实现Runnable接口创建线程
* > 如果自己的类已经继承另外一个类， 就无法直接继承Thread类。

```java
public class MyThrea extends OtherClass implements Runnable {
    public void run() {
        System.out.println("MyThread.run()");
    }
}
```

* 实现Callable接口通过FutureTask包装器来创建Thread线程

```java
public class CallableDemo implements Callable<String> {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(1);
        CallableDemo callableDemo = new CallableDemo();
        Future<String> future = executeorService.submit(callableDemo);
        System.out.println(future.get());
        executeorService.shudown();
    }
    @Override
    public String call() throws Exception {
        int a = 1;
        int b = 2;
        System.out.println(a + b);
        return "执行结果：" + (a + b); 
    }
}
```

### 线程的生命周期

> 一个线程6种状态： new, runnable, blocked, waiting, time\_waiting, terminated

* NEW：初始状态， 线程被构建，但还没调用start方法
* RUNNABLE：运行状态， JAVA线程中表操作系统中的就绪和运行两种状态统称为“运行中“
* BLOCKED：阻塞状态，线程进入等待状态， 放弃CPU使用权
  * 等待阻塞：运行线程执行wait方法，JVM会把当前线程放入到等待队列。
  * 同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁已被其他线程锁占用了， 那么JVM会把当前线程放入到锁池中
  * 其他阻塞：运行的线程执行Thread.sleep或者t.join方法， 或者发出了I/O请求时，JVM会把当前线程设置为阻塞状态， 当sleep结束，join线程终止，io处理完毕则线程恢复
* TIME\_WAITING：超时等待状态，超时以后自动返回
* TERMINATED：终止状态， 表示当前线程执行完毕

![](/assets/thread-life-cycle.png)



