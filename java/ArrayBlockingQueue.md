# ArrayBlockingQueue

ArrayBlockingQueue是一个底层用数组实现的有界阻塞队列，有界是指他的容量大小是固定的，不能扩充容量，在初始化时就必须确定队列大小。它通过可重入的独占锁ReentrantLock来控制并发，Condition来实现阻塞。

```java
//通过数组来存储队列中的元素
final Object[] items;

//初始化一个固定的数组大小，默认使用非公平锁来控制并发
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}

//初始化固定的items数组大小，初始化notEmpty以及notFull两个Condition来控制生产消费
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);//通过ReentrantLock来控制并发
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
```

可以看到ArrayBlockingQueue初始化了一个ReentrantLock以及两个Condition，用来控制并发下队列的生产消费。这里重点看下阻塞的put以及take方法：

```java
//插入元素到队列中
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly(); //获取独占锁
    try {
        while (count == items.length) //如果队列已满则通过await阻塞put方法
            notFull.await();
        enqueue(e); //插入元素
    } finally {
        lock.unlock();
    }
}

private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length) //插入元素后将putIndex+1，当队列使用完后重置为0
        putIndex = 0;
    count++;
    notEmpty.signal(); //队列添加元素后唤醒因notEmpty等待的消费线程
}

//移除队列中的元素
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly(); //获取独占锁
    try {
        while (count == 0) //如果队列已空则通过await阻塞take方法
            notEmpty.await(); 
        return dequeue(); //移除元素
    } finally {
        lock.unlock();
    }
}

private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length) //移除元素后将takeIndex+1，当队列使用完后重置为0
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal(); //队列消费元素后唤醒因notFull等待的消费线程
    return x;
}
```

在队列添加和移除元素的过程中使用putIndex、takeIndex以及count三个变量来控制生产消费元素的过程，putIndex负责记录下一个可添加元素的下标，takeIndex负责记录下一个可移除元素的下标，count记录了队列中的元素总量。队列满后通过notFull.await()来阻塞生产者线程，消费元素后通过notFull.signal()来唤醒阻塞的生产者线程。队列为空后通过notEmpty.await()来阻塞消费者线程，生产元素后通过notEmpty.signal()唤醒阻塞的消费者线程。

限时插入以及移除方法在ArrayBlockingQueue中通过awaitNanos来实现，在给定的时间过后如果线程未被唤醒则直接返回。

```java
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    checkNotNull(e);
    long nanos = unit.toNanos(timeout); //获取定时时长
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0) //指定时长过后，线程仍然未被唤醒则返回false
                return false;
            nanos = notFull.awaitNanos(nanos); //指定时长内阻塞线程
        }
        enqueue(e);
        return true;
    } finally {
        lock.unlock();
    }
}
```

还有一个比较重要的方法：drainTo，drainTo方法可以一次性获取队列中所有的元素，它减少了锁定队列的次数，使用得当在某些场景下对性能有不错的提升。

```java
public int drainTo(Collection<? super E> c, int maxElements) {
    checkNotNull(c);
    if (c == this)
        throw new IllegalArgumentException();
    if (maxElements <= 0)
        return 0;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock; //仅获取一次锁
    lock.lock();
    try {
        int n = Math.min(maxElements, count); //获取队列中所有元素
        int take = takeIndex;
        int i = 0;
        try {
            while (i < n) {
                @SuppressWarnings("unchecked")
                E x = (E) items[take];
                c.add(x); //循环插入元素
                items[take] = null;
                if (++take == items.length)
                    take = 0;
                i++;
            }
            return n;
        } finally {
            // Restore invariants even if c.add() threw
            if (i > 0) {
                count -= i;
                takeIndex = take;
                if (itrs != null) {
                    if (count == 0)
                        itrs.queueIsEmpty();
                    else if (i > take)
                        itrs.takeIndexWrapped();
                }
                for (; i > 0 && lock.hasWaiters(notFull); i--)
                    notFull.signal(); //唤醒等待的生产者线程
            }
        }
    } finally {
        lock.unlock();
    }
}
```

