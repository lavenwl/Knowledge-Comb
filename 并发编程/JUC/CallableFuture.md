```java
public class CallableDemo implements Callable<String> {
    @Override
    public String call() throws Exception {
        Thread.sleep(5000);
        return "hello world";
    }


    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(3);
        CallableDemo callableDemo = new CallableDemo();
        FutureTask futureTask = (FutureTask) service.submit(callableDemo);
//        FutureTask futureTask = new FutureTask(callableDemo);
//        new Thread(futureTask).start();
        System.out.println(futureTask.get());
    }
}
```

