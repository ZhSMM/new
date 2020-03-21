# CompletionService

CompletionService与ExecutorService类似都可以用来执行线程池的任务，ExecutorService继承了Executor接口，而CompletionService则是一个接口，那么为什么CompletionService不直接继承Executor接口呢？

主要是Executor的特性决定的，Executor框架不能完全保证任务执行的异步性，那就是如果需要实现任务（task）的异步性，只要为每个task创建一个线程就实现了任务的异步性。

代码往往包含`new Thread(task).start()`。这种方式的问题在于，它没有限制可创建线程的数量（在ExecutorService可以限制），不过，这样最大的问题是在高并发的情况下，不断创建线程异步执行任务将会极大**增大线程创建的开销**、**造成极大的资源消耗**和**影响系统的稳定性**。另外，Executor框架还支持同步任务的执行，就是在execute方法中调用提交任务的run()方法就属于同步调用。

一般情况下，如果需要判断任务是否完成，思路是得到Future列表的每个Future，然后反复调用其get方法，并将timeout参数设为0，从而通过轮询的方式判断任务是否完成。为了更精确实现任务的异步执行以及更简便的完成任务的异步执行，可以使用CompletionService。

## 实现原理

CompletionService实际上可以看做是Executor和BlockingQueue的结合体。CompletionService在接收到要执行的任务时，通过类似BlockingQueue的put和take获得任务执行的结果。CompletionService的一个实现是ExecutorCompletionService，ExecutorCompletionService把具体的计算任务交给Executor完成。

在实现上，ExecutorCompletionService在构造函数中会创建一个BlockingQueue（使用的基于链表的无界队列LinkedBlockingQueue），该BlockingQueue的作用是保存Executor执行的结果。当计算完成时，调用FutureTask的done方法。当提交一个任务到ExecutorCompletionService时，首先将任务包装成QueueingFuture，它是FutureTask的一个子类，然后改写FutureTask的done方法，之后把Executor执行的计算结果放入BlockingQueue中。QueueingFuture的源码如下：

```java
private class QueueingFuture extends FutureTask<Void> {
    QueueingFuture(RunnableFuture<V> task) {
        super(task, null);
        this.task = task;
    }
    protected void done() { completionQueue.add(task); }
    private final Future<V> task;
}
```

从代码可以看到，CompletionService将提交的任务转化为QueueingFuture，并且覆盖了done方法，在done方法中就是将任务加入任务队列中。这点与之前对Executor框架的分析是一致的。

## 示例

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.*;

public class ExecutorServiceTest {
    // 线程池
    private final static ExecutorService executorService= Executors.
            newFixedThreadPool(3);

    static class MyCallable implements Callable<String>{
        private String username;
        private Random random=new Random();

        public MyCallable(String username) {
            this.username = username;
        }

        @Override
        public String call() throws Exception {
            System.out.println(username);
            Thread.sleep(random.nextInt(500));
            return "return:"+username;
        }
    }
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        List<MyCallable> list=new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            MyCallable myCallable=new MyCallable("userName"+i);
            list.add(myCallable);
        }
        CompletionService completionService=new
                ExecutorCompletionService(executorService);
        for (int i = 0; i < 5; i++) {
            completionService.submit(list.get(i));
        }
        for (int i = 0; i < 5; i++) {
            System.out.println("等待打印"+i+"返回值");
            System.out.println(completionService.take().get());
        }
    }
}
```