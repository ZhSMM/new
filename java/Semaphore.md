# Semaphore

用来控制并发线程的数量。

```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.Semaphore;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 *  Semaphore是一种在多线程环境下使用的设施，该设施负责协调各个线程，以保证它们能够正确、合理的使用公
 *  共资源的设施，也是操作系统中用于控制进程同步互斥的量。Semaphore是一种计数信号量，用于管理一组资源，
 *  内部是基于AQS的共享模式。它相当于给线程规定一个量从而控制允许活动的线程数。
 * 常用方法：
 * Semaphore(int permits):构造方法，创建具有给定许可数的计数信号量并设置为非公平信号量。
 * Semaphore(int permits,boolean fair):构造方法，当fair等于true时，创建具有给定许可数的计数信号量并设置为公平信号量。
 * void acquire():从此信号量获取一个许可前线程将一直阻塞。相当于一辆车占了一个车位。
 * void acquire(int n):从此信号量获取给定数目许可，在提供这些许可前一直将线程阻塞。比如n=2，就相当于一辆车占了两个车位。
 * void release():释放一个许可，将其返回给信号量。就如同车开走返回一个车位。
 * void release(int n):释放n个许可。
 * int availablePermits()：当前可用的许可数。
 */
public class SemaphoreTest {
    private static final Semaphore SEMAPHORE=new Semaphore(3,true);
    private static final ThreadPoolExecutor POOL_EXECUTOR=new
            ThreadPoolExecutor(5,10,
            60, TimeUnit.SECONDS,new LinkedBlockingQueue<Runnable>());
    public static void main(String[] args) {
        class InformationThread extends Thread{
            private final String name;
            private final int age;

            public InformationThread(String name,int age) {
                this.name = name;
                this.age = age;
            }

            @Override
            public void run() {
                try{
                    SEMAPHORE.acquire();
                    System.out.println(Thread.currentThread().getName()+
                            " name:"+name+" age:"+age+" 时间："+System.currentTimeMillis());
                    Thread.sleep(1000);
                    System.out.println("当前可使用的许可数："+SEMAPHORE.availablePermits());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    SEMAPHORE.release();
                }
            }
        }
        String[] name={"李明","王五","张杰","王强","赵二","李四","张三"};
        int[] age= {26,27,33,45,19,23,41};
        for (int i = 0; i < 7; i++) {
            Thread t1=new InformationThread(name[i],age[i]);
            POOL_EXECUTOR.execute(t1);
        }
    }
}
```