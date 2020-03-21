# Java Executor框架

> 在Java 5之后，并发编程引入了一堆新的启动、调度和管理线程的API。Executor框架便是Java 5中引入的，其内部使用了线程池机制，它在java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。
>
> 因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用Executor在构造器中。
>
> Executor作为灵活且强大的异步执行框架，其支持多种不同类型的任务执行策略，提供了一种标准的方法将任务的提交过程和执行过程解耦开发，基于生产者-消费者模式，其提交任务的线程相当于生产者，执行任务的线程相当于消费者，并用Runnable来表示任务，Executor的实现还提供了对生命周期的支持，以及统计信息收集，应用程序管理机制和性能监视等机制。

Executor框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable等。

## Executor和ExecutorService

Runnable与Callable接口都是对任务处理逻辑的抽象,将任务处理逻辑抽象成统一签名的方法Runnable.run()或Callable.call();

java.util.concurrent.Executor接口则是对任务的执行进行的抽象, 该接口仅定义了`void execute(Runnable command)`

command参数代表需要执行的任务, Executor接口使得任务的提交方(相当于生产者)只需要知道他调用Executor.execute方法可以使指定的任务被执行,而无需关心执行细节;

```java
import java.util.concurrent.Executor;

public class ExecutorTest {
    public static void main(String[] args) {
        Runnable run=()->{
            System.out.println(Thread.currentThread().getName()+
                    "线程执行！");
        };
        // 同步执行
        Executor executor= Runnable::run;
        executor.execute(run);
        // 异步执行
        Executor executor1=command -> {new Thread(command).start();};
        executor1.execute(run);
    }
}
// 结果:
main线程执行！
Thread-0线程执行！
```

ExecutorService：

- 是一个比Executor使用更广泛的子类接口，其提供了生命周期管理的方法，返回 Future 对象，以及可跟踪一个或多个异步任务执行状况返回Future的方法；
- 可以调用ExecutorService的shutdown（）方法来平滑地关闭 ExecutorService，调用该方法后，将导致ExecutorService停止接受任何新的任务且等待已经提交的任务执行完成(已经提交的任务会分两类：一类是已经在执行的，另一类是还没有开始执行的)，当所有已经提交的任务执行完毕后将会关闭ExecutorService。因此我们一般用该接口来实现和管理多线程。

- 关闭shutdown(): 在终止后，执行程序没有任务在执行，也没有任务在等待执行，并且无法提交新任务;
  - shutdown()方法在终止前允许执行以前提交的任务;
  - shutdownNow() 方法阻止等待任务的启动并试图停止当前正在执行的任务。
- 通过 ExecutorService.submit() 方法返回的 Future 对象，可以调用isDone（）方法查询Future是否已经完成。当任务完成时，它具有一个结果，你可以调用get()方法来获取该结果。你也可以不用isDone（）进行检查就直接调用get()获取结果，在这种情况下，get()将阻塞，直至结果准备就绪，还可以取消任务的执行。Future 提供了 cancel() 方法用来取消执行 pending 中的任务。