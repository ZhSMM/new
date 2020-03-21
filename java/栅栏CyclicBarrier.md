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

**构造方法**　

CyclicBarrier(int parties)
创建一个新的 CyclicBarrier ，当给定数量的线程（线程）等待它时，它将跳闸，并且当屏障跳闸时不执行预定义的动作。
CyclicBarrier(int parties, Runnable barrierAction)
创建一个新的 CyclicBarrier ，当给定数量的线程（线程）等待时，它将跳闸，当屏障跳闸时执行给定的屏障动作，由最后一个进入屏障的线程执行。

**方法**

- int await() 等待所有 parties已经在这个障碍上调用了 await 。
- int await(long timeout, TimeUnit unit) 等待所有 parties已经在此屏障上调用 await ，或指定的等待时间过去。
- int getNumberWaiting() 返回目前正在等待障碍的各方的数量。
- int getParties() 返回旅行这个障碍所需的parties数量。
- boolean isBroken() 查询这个障碍是否处于破碎状态。
- void reset() 将屏障重置为初始状态。

**与CountDownLatch的区别**

1. CountDownLatch是线程组之间的等待，即一个(或多个)线程等待N个线程完成某件事情之后再执行；而CyclicBarrier则是线程组内的等待，即每个线程相互等待，即N个线程都被拦截之后，然后依次执行。
2. CountDownLatch是减计数方式，而CyclicBarrier是加计数方式。
3. CountDownLatch计数为0无法重置，而CyclicBarrier计数达到初始值，则可以重置。
4. CountDownLatch不可以复用，而CyclicBarrier可以复用。

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

```java
// 错误的使用方式
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CyclicBarrierWithPoolTest {
    public static void main(String[] args) {
        // 同步辅助类，允许一组线程相互等待，直到公共屏障点
        class Run implements Runnable{
            private CyclicBarrier barrier;
            private String name;

            public Run(CyclicBarrier barrier, String name) {
                this.barrier = barrier;
                this.name = name;
            }

            @Override
            public void run() {
                try {
                    System.out.println(name+"到达预备点");
                    barrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println(name+"开始跑！");
            }
        }
        CyclicBarrier barrier=new CyclicBarrier(5,()->{
            System.out.println("暂停点执行！");
        });
        ExecutorService service= Executors.newFixedThreadPool(5);
        String[][] name=new String[2][5];
        name[0]= new String[]{"mj", "hj", "kk", "jo", "kiu"};
        name[1]= new String[]{"ml", "ho", "gg", "uu", "ttr"};
        for (int i = 0; i < 5; i++) {
            Thread t1=new Thread(new Run(barrier,name[0][i]));
            service.execute(t1);
        }
        for (int i = 0; i < 5; i++) {
            Thread t1=new Thread(new Run(barrier,name[1][i]));
            service.execute(t1);
        }
    }
}
// 输出：
kiu开始跑！
mj开始跑！
hj开始跑！
gg到达预备点
jo开始跑！
ml到达预备点
uu到达预备点
kk开始跑！
ttr到达预备点
ho到达预备点
暂停点执行！
ho开始跑！
gg开始跑！
uu开始跑！
ml开始跑！
ttr开始跑！
```