# 栅栏CyclicBarrier

CyclicBarrier是java推出的一个并发编程工具，它用在多个线程之间协同工作。线程约定到达某个点，到达这个点之后的线程都停下来，直到最后一个线程也到达了这个点之后，所有的线程才会得到释放。常用的场景是：多个worker线程，每个线程都在循环地做一部分工作，并在最后用cyclicBarrier.await()设下约定点，当最后一个线程做完了工作也到达约定点后，所有线程得到释放，开始下一轮工作。也就是下面这样：

```java
while(!done()){
    // working
    cyclicBarrier.await();
}
```

- 使用CyclicBarrier实现等待的线程被称为**参与方（Party）**，参与方只需要实现CyclicBarrier.await()就可以实现等待。从应用代码看，参与方是并发执行CyclicBarrier.await()，但是，CyclicBarrier内部维护了一个显示锁，使得总是可以在所有参与方中分出一个最后执行CyclicBarrier.await()的线程，该线程被称为**最后一个线程**。
- 除最后一个线程外，参与方执行CyclicBarrier.await()均会使该线程被暂停（线程生命周期状态变为waiting），最后一个线程的await会导致使用相应CyclicBarrier实例的其他所有参与方被唤醒，且最后一个线程自身不会被暂停。
- 与CountDownLatch不同，CyclicBarrier实例可以重复使用。

**使用范围：**

- 使迭代算法并发化；
- 在测试代码中模拟高并发；

示例：

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

// 单词使用
public class CyclicBarrierTest {
    private static final int threadNum=5;
    public static class WorkerThread implements Runnable{
        CyclicBarrier barrier;

        public WorkerThread(CyclicBarrier barrier) {
            this.barrier = barrier;
        }

        @Override
        public void run() {
            try {
                System.out.println("ID:"+Thread.currentThread().getId()+" 名称："
                        +Thread.currentThread().getName()+"的线程等待点");
                // 线程在这里等待
                barrier.await();
                System.out.println("ID:"+Thread.currentThread().getId()+" 名称："
                        +Thread.currentThread().getName()+"继续工作！");
            } catch (InterruptedException|BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier=new CyclicBarrier(threadNum, new Runnable() {
            // 所有线程到达barrier时执行
            @Override
            public void run() {
                System.out.println("Barrier方法！");
            }
        });
        for (int i = 0; i < 5; i++) {
            new Thread(new WorkerThread(cyclicBarrier)).start();
        }
    }
}

// 输出：
ID:12 名称：Thread-0的线程等待点
ID:14 名称：Thread-2的线程等待点
ID:13 名称：Thread-1的线程等待点
ID:15 名称：Thread-3的线程等待点
ID:16 名称：Thread-4的线程等待点
Barrier方法！
ID:16 名称：Thread-4继续工作！
ID:12 名称：Thread-0继续工作！
ID:13 名称：Thread-1继续工作！
ID:14 名称：Thread-2继续工作！
ID:15 名称：Thread-3继续工作！
```

