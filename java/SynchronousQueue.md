# SynchronousQueue

SynchronousQueue相比较之前的4个队列就比较特殊了，它是一个没有容量的队列，也就是说它内部时不会对数据进行存储，每进行一次put之后必须要进行一次take，否则相同线程继续put会阻塞。这种特性很适合做一些传递性的工作，一个线程生产，一个线程消费。内部分为公平和非公平访问两种模式，默认使用非公平，未使用锁，全部通过CAS操作来实现并发，吞吐量非常高。这里只对它的非公平实现下的take和put方法做下简单分析：

```java
//非公平情况下调用内部类TransferStack的transfer方法put
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, false, 0) == null) {
        Thread.interrupted();
        throw new InterruptedException();
    }
}
//非公平情况下调用内部类TransferStack的transfer方法take
public E take() throws InterruptedException {
    E e = transferer.transfer(null, false, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}

//具体的put以及take方法，只有E的区别，通过E来区别REQUEST还是DATA模式
E transfer(E e, boolean timed, long nanos) {
    SNode s = null; // constructed/reused as needed
    int mode = (e == null) ? REQUEST : DATA;

    for (;;) {
        SNode h = head;
        //栈无元素或者元素和插入的元素模式相匹配，也就是说都是插入元素
        if (h == null || h.mode == mode) {  
            //有时间限制并且超时
            if (timed && nanos <= 0) {      
                if (h != null && h.isCancelled())
                    casHead(h, h.next);  // 重新设置头节点
                else
                    return null;
            } 
            //未超时cas操作尝试设置头节点
            else if (casHead(h, s = snode(s, e, h, mode))) {
                //自旋一段时间后未消费元素则挂起put线程
                SNode m = awaitFulfill(s, timed, nanos);
                if (m == s) {               // wait was cancelled
                    clean(s);
                    return null;
                }
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        } 
        //栈不为空并且和头节点模式不匹配，存在元素则消费元素并重新设置head节点
        else if (!isFulfilling(h.mode)) { // try to fulfill
            if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                for (;;) { // loop until matched or waiters disappear
                    SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                        casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }
                    SNode mn = m.next;
                    if (m.tryMatch(s)) {
                        casHead(s, mn);     // pop both s and m
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                  // lost match
                        s.casNext(m, mn);   // help unlink
                }
            }
        }
        //节点正在匹配阶段 
        else {                            // help a fulfiller
            SNode m = h.next;               // m is h's match
            if (m == null)                  // waiter is gone
                casHead(h, null);           // pop fulfilling node
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))          // help match
                    casHead(h, mn);         // pop both h and m
                else                        // lost match
                    h.casNext(m, mn);       // help unlink
            }
        }
    }
}

//先自旋后挂起的核心方法
SNode awaitFulfill(SNode s, boolean timed, long nanos) {
    final long deadline = timed ? System.nanoTime() + nanos : 0L;
    Thread w = Thread.currentThread();
    //计算自旋的次数
    int spins = (shouldSpin(s) ?
                    (timed ? maxTimedSpins : maxUntimedSpins) : 0);
    for (;;) {
        if (w.isInterrupted())
            s.tryCancel();
        SNode m = s.match;
        //匹配成功过返回节点
        if (m != null)
            return m;
        //超时控制
        if (timed) {
            nanos = deadline - System.nanoTime();
            if (nanos <= 0L) {
                s.tryCancel();
                continue;
            }
        }
        //自旋检查，是否进行下一次自旋
        if (spins > 0)
            spins = shouldSpin(s) ? (spins-1) : 0;
        else if (s.waiter == null)
            s.waiter = w; // establish waiter so can park next iter
        else if (!timed)
            LockSupport.park(this); //在这里挂起线程
        else if (nanos > spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanos);
    }
}
```

代码中可以看到put以及take方法都是通过调用transfer方法来实现的，然后通过参数mode来区别，在生产元素时如果是同一个线程多次put则会采取自旋的方式多次尝试put元素，可能自旋过程中元素会被消费，这样可以及时put，降低线程挂起的性能损耗，高吞吐量的核心也在这里，消费线程一样，空栈时也会先自旋，自旋失败然后通过线程的LockSupport.park方法挂起。