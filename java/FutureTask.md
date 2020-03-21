# FutureTask

一个可取消的异步计算。FutureTask提供了对Future的基本实现，可以调用方法去开始和取消一个计算，可以查询计算是否完成并且获取计算结果。只有当计算完成时才能获取到计算结果，一旦计算完成，计算将不能被重启或者被取消，除非调用runAndReset方法。
除了实现了Future接口以外，FutureTask还实现了Runnable接口，因此FutureTask交由Executor执行，也可以直接用线程调用执行(futureTask.run())。

根据FutureTask的run方法执行的时机，FutureTask可以处于以下三种执行状态：

1. 未启动：在FutureTask.run()还没执行之前，FutureTask处于未启动状态。当创建一个FutureTask对象，并且run()方法未执行之前，FutureTask处于未启动状态。
2. 已启动：FutureTask对象的run方法启动并执行的过程中，FutureTask处于已启动状态。
3. 已完成：FutureTask正常执行结束，或者FutureTask执行被取消(FutureTask对象cancel方法)，或者FutureTask对象run方法执行抛出异常而导致中断而结束，FutureTask都处于已完成状态。

- 当FutureTask处于未启动或者已启动的状态时，调用FutureTask对象的get方法会将导致调用线程阻塞。当FutureTask处于已完成的状态时，调用FutureTask的get方法会立即放回调用结果或者抛出异常。

- 当FutureTask处于未启动状态时，调用FutureTask对象的cancel方法将导致线程永远不会被执行；当FutureTask处于已启动状态时，调用FutureTask对象cancel(true)方法将以中断执行此任务的线程的方式来试图停止此任务;当FutureTask处于已启动状态时，调用FutureTask对象cancel(false)方法将不会对正在进行的任务产生任何影响；当FutureTask处于已完成状态时，调用FutureTask对象cancel方法将返回false；

**使用**:

1. 可以把FutureTask交给Executor执行；
2. 也可以通ExecutorService.submit（…）方法返回一个FutureTask，然后执行FutureTask.get()方法或FutureTask.cancel（…）方法。
3. 除此以外，还可以单独使用FutureTask。
   - 当一个线程需要等待另一个线程把某个任务执行完后它才能继续执行，此时可以使用FutureTask。
   - 假设有多个线程执行若干任务，每个任务最多只能被执行一次。当多个线程试图同时执行同一个任务时，只允许一个线程执行任务，其他线程需要等待这个任务执行完后才能继续执行。

## Future

FutureTask实现了Future接口，Future接口有5个方法：
1、boolean cancel(boolean mayInterruptIfRunning)
尝试取消当前任务的执行。如果任务已经取消、已经完成或者其他原因不能取消，尝试将失败。如果任务还没有启动就调用了cancel(true)，任务将永远不会被执行。如果任务已经启动，参数mayInterruptIfRunning将决定任务是否应该中断执行该任务的线程，以尝试中断该任务。
如果任务不能被取消，通常是因为它已经正常完成，此时返回false，否则返回true
2、boolean isCancelled()
如果任务在正常结束之前被被取消返回true
3、boolean isDone()
正常结束、异常或者被取消导致任务完成，将返回true
4、V get()
等待任务结束，然后获取结果，如果任务在等待过程中被终端将抛出InterruptedException，如果任务被取消将抛出CancellationException，如果任务中执行过程中发生异常将抛出ExecutionException。
5、V get(long timeout, TimeUnit unit)
任务最多在给定时间内完成并返回结果，如果没有在给定时间内完成任务将抛出TimeoutException。

## FutureTask状态转换

FutureTask任务的运行状态，最初为NEW。运行状态仅在set、setException和cancel方法中转换为终端状态。在完成过程中，状态可能呈现出瞬时值INTERRUPTING(仅在中断运行程序以满足cancel(true)的情况下)或者COMPLETING(在设置结果时)状态时。从这些中间状态到最终状态的转换使用成本更低的有序/延迟写，因为值是统一的，需要进一步修改。
state：表示当前任务的运行状态，FutureTask的所有方法都是围绕state开展的，state声明为volatile，保证了state的可见性，当对state进行修改时所有的线程都会看到。

- NEW：表示一个新的任务，初始状态
- COMPLETING：当任务被设置结果时，处于COMPLETING状态，这是一个中间状态。
- NORMAL：表示任务正常结束。
- EXCEPTIONAL：表示任务因异常而结束
- CANCELLED：任务还未执行之前就调用了cancel(true)方法，任务处于CANCELLED
- INTERRUPTING：当任务调用cancel(true)中断程序时，任务处于INTERRUPTING状态，这是一个中间状态。
- INTERRUPTED：任务调用cancel(true)中断程序时会调用interrupt()方法中断线程运行，任务状态由INTERRUPTING转变为INTERRUPTED；

可能的状态过渡：
 1、NEW -> COMPLETING -> NORMAL：正常结束
 2、NEW -> COMPLETING -> EXCEPTIONAL：异常结束
 3、NEW -> CANCELLED：任务被取消
 4、NEW -> INTERRUPTING -> INTERRUPTED：任务出现中断

```java
import java.util.concurrent.*;

public class FutureTaskTest {
    public static void main(String[] args) {
        class Task implements Callable<Integer>{
            private final int n;

            public Task(int n) {
                this.n = n;
            }

            @Override
            public Integer call() throws Exception {
                int left=1;
                int right=1;
                if (n<3) return 1;
                for (int i = 2; i < n; i++) {
                    right+=left;
                    left=right-left;
                }
                return right;
            }
        }
        // 实例化任务，传递参数
        Task task=new Task(20);
        // 将任务放进FutureTask里
        FutureTask<Integer> futureTask=new FutureTask<>(task);
        // 使用ExecutorService来执行FutureTask
        ExecutorService service=Executors.newFixedThreadPool(2);
        service.submit(futureTask);
        service.shutdown();
        try {
            System.out.println(futureTask.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}

```

