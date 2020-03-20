# DelayQueue

DelayQueue也是一个无界队列，它是在PriorityQueue基础上实现的，先按延迟优先级排序，延迟时间短的排在前面。和PriorityBlockingQueue相似，底层也是数组，采用一个ReentrantLock来控制并发。由于是无界的，所以插入元素时不会阻塞，没有队列满的状态。能想到的最简单的使用场景一般有两个：一个是缓存过期，一个是定时执行的任务。但由于是无界的，缓存过期上一般使用的并不多。简单来看下put以及take方法：

```java
private final transient ReentrantLock lock = new ReentrantLock();
private final PriorityQueue<E> q = new PriorityQueue<E>();//优先级队列

public void put(E e) {
    offer(e);
}

public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        q.offer(e); //插入元素到优先级队列
        if (q.peek() == e) { //如果插入的元素在队列头部
            leader = null;
            available.signal(); //通知消费线程
        }
        return true;
    } finally {
        lock.unlock();
    }
}

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek(); //获取头部元素
            if (first == null)
                available.await(); //空队列阻塞
            else {
                long delay = first.getDelay(NANOSECONDS); //检查元素是否延迟到期
                if (delay <= 0)
                    return q.poll(); //到期则弹出元素
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay); //阻塞未到期的时间
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```