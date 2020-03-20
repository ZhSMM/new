# LinkedBlockingQueue

LinkedBlockingQueue是一个底层用单向链表实现的有界阻塞队列，和ArrayBlockingQueue一样，采用ReentrantLock来控制并发，不同的是它使用了两个独占锁来控制消费和生产。put以及take方法源码如下：

```java
public void put(E e) throws InterruptedException {
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    //因为使用了双锁，需要使用AtomicInteger计算元素总量，避免并发计算不准确
    final AtomicInteger count = this.count; 
    putLock.lockInterruptibly();
    try {
        while (count.get() == capacity) {
            notFull.await(); //队列已满，阻塞生产线程
        }
        enqueue(node); //插入元素到队列尾部
        c = count.getAndIncrement(); //count + 1
        if (c + 1 < capacity) //如果+1后队列还未满，通过其他生产线程继续生产
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0) //只有当之前是空时，消费队列才会阻塞，否则是不需要通知的
        signalNotEmpty(); 
}

private void enqueue(Node<E> node) {
    //将新元素添加到链表末尾，然后将last指向尾部元素
    last = last.next = node;
}

public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await(); //队列为空，阻塞消费线程
        }
        x = dequeue(); //消费一个元素
        c = count.getAndDecrement(); //count - 1
        if (c > 1) // 通知其他等待的消费线程继续消费
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity) //只有当之前是满的，生产队列才会阻塞，否则是不需要通知的
        signalNotFull();
    return x;
}

//消费队列头部的下一个元素，同时将新头部置空
private E dequeue() {
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h; // help GC
    head = first;
    E x = first.item;
    first.item = null;
    return x;
}
```

可以看到LinkedBlockingQueue通过takeLock和putLock两个锁来控制生产和消费，互不干扰，只要队列未满，生产线程可以一直生产，只要队列不为空，消费线程可以一直消费，不会相互因为独占锁而阻塞。

看过了LinkedBlockingQueue以及ArrayBlockingQueue的底层实现，会发现一个问题，正常来说消费者和生产者可以并发执行对队列的吞吐量会有比较大的提升，那么为什么ArrayBlockingQueue中不使用双锁来实现队列的生产和消费呢？我的理解是ArrayBlockingQueue也能使用双锁来实现功能，但由于它底层使用了数组这种简单结构，相当于一个共享变量，如果通过两个锁，需要更加精确的锁控制，这也是为什么JDK1.7中的ConcurrentHashMap使用了分段锁来实现，将一个数组分为多个数组来提高并发量。LinkedBlockingQueue不存在这个问题，链表这种数据结构头尾节点都相对独立，存储上也不连续，双锁控制不存在复杂性。