# PriorityBlockingQueue

PriorityBlockingQueue是一个底层由数组实现的无界队列，并带有排序功能，同样采用ReentrantLock来控制并发。由于是无界的，所以插入元素时不会阻塞，没有队列满的状态，只有队列为空的状态。通过这两点特征其实可以猜测它应该是有一个独占锁（底层数组）和一个Condition（只通知消费）来实现的。put以及take方法源码分析如下：

```java
public void put(E e) {
    offer(e);
}

public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    int n, cap;
    Object[] array;
    //无界队列，队列长度不够时会扩容
    while ((n = size) >= (cap = (array = queue).length))
        tryGrow(array, cap);
    try {
        //通过comparator来实现优先级排序
        Comparator<? super E> cmp = comparator;
        if (cmp == null)
            siftUpComparable(n, e, array);
        else
            siftUpUsingComparator(n, e, array, cmp);
        size = n + 1;
        notEmpty.signal(); //和ArrayBlockingQueue一样，每次添加元素后通知消费线程
    } finally {
        lock.unlock();
    }
    return true;
}

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    E result;
    try {
        while ( (result = dequeue()) == null)
            notEmpty.await(); //队列为空，阻塞消费线程
    } finally {
        lock.unlock();
    }
    return result;
}
```

