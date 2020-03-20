# LinkedTransferQueue

LinkedTransferQueue是一个无界的阻塞队列，底层由链表实现。虽然和LinkedBlockingQueue一样也是链表实现的，但并发控制的实现上却很不一样，和SynchronousQueue类似，采用了大量的CAS操作，没有使用锁，由于是无界的，所以不会put生产线程不会阻塞，只会在take时阻塞消费线程，消费线程挂起时同样使用LockSupport.park方法。

LinkedTransferQueue相比于以上的队列还提供了一些额外的功能，它实现了TransferQueue接口，有两个关键方法transfer(E e)和tryTransfer(E  e)方法，transfer在没有消费时会阻塞，tryTransfer在没有消费时不会插入到队列中，也不会等待，直接返回false。

```java
private static final int NOW   = 0; // for untimed poll, tryTransfer
private static final int ASYNC = 1; // for offer, put, add
private static final int SYNC  = 2; // for transfer, take
private static final int TIMED = 3; // for timed poll, tryTransfer

//通过SYNC状态来实现生产阻塞
public void transfer(E e) throws InterruptedException {
    if (xfer(e, true, SYNC, 0) != null) {
        Thread.interrupted(); // failure possible only due to interrupt
        throw new InterruptedException();
    }
}
//通过NOW状态跳过添加元素以及阻塞
public boolean tryTransfer(E e) {
    return xfer(e, true, NOW, 0) == null;
}

//通过ASYNC状态跳过阻塞
public void put(E e) {
    xfer(e, true, ASYNC, 0);
}
//通过SYNC状态来实现消费阻塞
public E take() throws InterruptedException {
    E e = xfer(null, false, SYNC, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}

//生产消费调用同一个方法，通过e是否为空，haveData，how等参数来区分具体逻辑
private E xfer(E e, boolean haveData, int how, long nanos) {
    if (haveData && (e == null))
        throw new NullPointerException();
    Node s = null;                        // the node to append, if needed

    retry:
    for (;;) {                            // restart on append race
        //找出第一个可用节点
        for (Node h = head, p = h; p != null;) { // find & match first node
            boolean isData = p.isData;
            Object item = p.item;
            //队列为空时直接跳过
            if (item != p && (item != null) == isData) { // unmatched
                //节点类型相同，跳过
                if (isData == haveData)   // can't match
                    break;
                if (p.casItem(item, e)) { // match
                    for (Node q = p; q != h;) {
                        Node n = q.next;  // update by 2 unless singleton
                        if (head == h && casHead(h, n == null ? q : n)) {
                            h.forgetNext();
                            break;
                        }                 // advance and retry
                        if ((h = head)   == null ||
                            (q = h.next) == null || !q.isMatched())
                            break;        // unless slack < 2
                    }
                    LockSupport.unpark(p.waiter);
                    return LinkedTransferQueue.<E>cast(item);
                }
            }
            Node n = p.next;
            p = (p != n) ? n : (h = head); // Use head if p offlist
        }
        //插入节点或移除节点具体逻辑
        //tryTransfer方法会直接跳过并返回结果
        if (how != NOW) {                 // No matches available
            if (s == null)
                s = new Node(e, haveData);
            Node pred = tryAppend(s, haveData); //加入节点
            if (pred == null)
                continue retry;           // lost race vs opposite mode
            if (how != ASYNC)
                //自旋以及阻塞消费线程逻辑，和SynchronousQueue类似，先尝试自选，失败后挂起线程
                //transfer方法在没有消费线程时也会阻塞在这里
                return awaitMatch(s, pred, e, (how == TIMED), nanos);
        }
        return e; // not waiting
    }
}
```