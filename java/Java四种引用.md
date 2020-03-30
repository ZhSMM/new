# Java四种引用

Java中的操作对象的标志符为引用（reference）。

jdk1.2之前：如果 reference 类型的数据中存储的数值代表的是另外一块内存的起始地址，就称为这块内存代表着一个引用。

jdk GC：根据对象是否存在引用指向它而判断是否需要进行垃圾回收。

jdk1.2之后，引用分四种类型：

- 强引用（Strong Reference）
- 软引用（Soft Reference）
- 弱引用（Weak Reference）
- 虚引用（Phantom Reference）

## 强引用

Java中的默认声明就是强引用。示例：

```java
Object obj = new Object(); //只要obj还指向Object对象，Object对象就不会被回收
obj = null;  //手动置null
```

只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足时，JVM也会直接抛出OutOfMemoryError，不会去回收。如果想中断强引用与对象之间的联系，可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了。

## 软引用

软引用是用来描述一些非必需但仍有用的对象。**在内存足够的时候，软引用对象不会被回收，只有在内存不足时，系统则会回收软引用对象，如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常**。这种特性常常被用来实现缓存技术，比如网页缓存，图片缓存等。
在 JDK1.2 之后，用java.lang.ref.SoftReference类来表示软引用。

## 弱引用

弱引用的引用强度比软引用要更弱一些，**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收**。

在 JDK1.2 之后，用 java.lang.ref.WeakReference 来表示弱引用。

## 虚引用

引用强度最弱的引用，这种引用比较特殊：被虚引用不会影响其所指对象的生命周期，即Java对象是否被回收与指向它的虚引用无关，也不能通过虚引用来得到其指向的对象（即get方法的返回值为null）。

虚引用的作用：配合引用队列（ReferenceQueue）来使用。当某个虚引用指向的对象被回收时，我们可以在其引用队列中得到这个虚引用的对象作为其所指向的对象被回收的一个通知。

引用队列（ReferenceQueue）：与软引用、弱引用、虚引用配合使用，当垃圾回收器准备回收一个对象时，如果发现它还有引用，那么就会在回收对象之前，把这个引用加入到与之关联的引用队列中去。程序可以通过判断引用队列中是否已经加入了引用，来判断被引用的对象是否将要被垃圾回收，这样就可以在对象被回收之前采取一些必要的措施。

## 测试

虚拟机参数设置：

-Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+PrintGCDetails  -XX:+UseParallelGC

```java
public class TestReference {
    
    // 引用测试类
    static class ReferenceTest {
        static final int _1M = 1024*1024;
        // 引用队列，当某个引用所指向的对象被回收时这个引用本身会被添加到其对应的引用队列中
        // 其泛型为其中存放的引用要指向的对象类型
        ReferenceQueue<Object> referenceQueue = new ReferenceQueue<>();

        // 强引用测试
        void testStrongReference() {
            ArrayList<byte[]> strongReferences = new ArrayList<>();
            try {
                while (true) {
                    strongReferences.add(new byte[_1M]);
                }
            } catch (OutOfMemoryError e) {
                e.printStackTrace();
            }
        }

        // 软引用测试
        void testSoftReference() {
            ArrayList<SoftReference> softReferences = new ArrayList<>();
            try {
                while (true) {
                    softReferences.add(new SoftReference<>(new byte[_1M], referenceQueue));
                }
            } catch (OutOfMemoryError e) {
                e.printStackTrace();
            }
        }

        // 弱引用测试
        void testWeakReference() {
            ArrayList<WeakReference> weakReferences = new ArrayList<>();
            try {
                while (true) {
                    weakReferences.add(new WeakReference<>(new byte[_1M], referenceQueue));
                }
            } catch (OutOfMemoryError e) {
                e.printStackTrace();
            }
        }

        // 虚引用测试
        void testPhantomReference() {
            ArrayList<PhantomReference<byte[]>> phantomReferences = new ArrayList<>();
            try {
                while (true) {
                    phantomReferences.add(new PhantomReference<>(new byte[_1M], referenceQueue));
                }
            } catch (OutOfMemoryError e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ReferenceTest test=new ReferenceTest();
        test.testStrongReference();
    }
}
```

强引用测试输出：

- PSYoungGen 出现Allocation Failure（内存空间不足，分配失败），触发一次MinorGC，由于强引用，无法回收内存，启动内存担保机制，将PSYoungGen中的数据暂存到ParOldGen（永久代），可以看到虽然进行了回收，但总空间减少的很少8002K->6976K；

  `[GC (Allocation Failure) [PSYoungGen: 8002K->824K(9216K)] 8002K->6976K(19456K), 0.0021287 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] `

- 触发Full GC（Ergonomics），将年轻代腾空转移到永久代；

  `[Full GC (Ergonomics) [PSYoungGen: 824K->0K(9216K)] [ParOldGen: 6152K->6763K(10240K)] 6976K->6763K(19456K), [Metaspace: 3227K->3227K(1056768K)], 0.0072017 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] `

- 触发Full GC（Allocation Failure）报错OOM（java.lang.OutOfMemoryError）；

  ```
  [Full GC (Ergonomics) [PSYoungGen: 7325K->4096K(9216K)] [ParOldGen: 6763K->9832K(10240K)] 14089K->13928K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0087087 secs] [Times: user=0.00 sys=0.02, real=0.01 secs] 
  [Full GC (Ergonomics) [PSYoungGen: 7482K->7168K(9216K)] [ParOldGen: 9832K->9831K(10240K)] 17314K->17000K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0064862 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
  [Full GC (Allocation Failure) [PSYoungGen: 7168K->7168K(9216K)] [ParOldGen: 9831K->9813K(10240K)] 17000K->16981K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0082264 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
  ```

软引用测试输出：

- PSYoungGen 出现Allocation Failure（内存空间不足，分配失败），触发一次MinorGC，由于强引用，无法回收内存，启动内存担保机制，将PSYoungGen中的数据暂存到ParOldGen（永久代），可以看到虽然进行了回收，但总空间减少的很少8002K->6976K；

  `[GC (Allocation Failure) [PSYoungGen: 8002K->824K(9216K)] 8002K->6976K(19456K), 0.0021849 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] `

- 触发Full GC（Ergonomics）；

- 触发Full GC（Allocation Failure），软引用被回收，程序继续运行；

```
[GC (Allocation Failure) [PSYoungGen: 8002K->824K(9216K)] 8002K->6976K(19456K), 0.0021849 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 824K->0K(9216K)] [ParOldGen: 6152K->6764K(10240K)] 6976K->6764K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0067752 secs] [Times: user=0.16 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 7325K->4096K(9216K)] [ParOldGen: 6764K->9832K(10240K)] 14089K->13928K(19456K), [Metaspace: 3229K->3229K(1056768K)], 0.0088843 secs] [Times: user=0.11 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 7482K->7168K(9216K)] [ParOldGen: 9832K->9832K(10240K)] 17315K->17000K(19456K), [Metaspace: 3229K->3229K(1056768K)], 0.0063086 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 7168K->0K(9216K)] [ParOldGen: 9832K->598K(10240K)] 17000K->598K(19456K), [Metaspace: 3229K->3229K(1056768K)], 0.0056542 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 7289K->7168K(9216K)] [ParOldGen: 9814K->9814K(10240K)] 17104K->16983K(19456K), [Metaspace: 3229K->3229K(1056768K)], 0.0051924 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 7168K->0K(9216K)] [ParOldGen: 9814K->599K(10240K)] 16983K->599K(19456K), [Metaspace: 3229K->3229K(1056768K)], 0.0035842 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
......................(无穷无尽)..........................
```

弱引用测试输出：

- 可以看到，软引用在触发Minor GC时被回收；

```
17780K->9464K(19968K), 0.0003857 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 8500K->160K(9728K)] 17836K->9504K(19968K), 0.0003044 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 8532K->160K(9728K)] 17876K->9520K(19968K), 0.0003631 secs] [Times: user=0.00 
............................................
[Full GC (Ergonomics) [PSYoungGen: 128K->0K(9728K)] [ParOldGen: 10216K->2084K(10240K)] 10344K->2084K(19968K), [Metaspace: 3761K->3761K(1056768K)], 0.0100210 secs] [Times: user=0.16 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [PSYoungGen: 8372K->32K(9728K)] 10457K->3140K(19968K), 0.0005201 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
......................(无穷无尽)..........................
```

虚引用测试输出：

虚引用确实是引用强度最弱的，但是还有一点是虚引用根本不会影响对象的声明周期，也就是说某个对象是否被 JVM 的垃圾回收动作回收和这个对象是否被虚引用所指向和被多少个虚引用所指向没有任何关系，既然其不会影响对象的生命周期，那么使用和不使用虚引用指向对象对这个对象是否被 JVM 回收是没有任何区别的，那么我们就可以将其看做没有使用虚引用时的代码，此时效果自然和直接使用强引用一样。

```
[GC (Allocation Failure) [PSYoungGen: 8002K->808K(9216K)] 8002K->6960K(19456K), 0.0043642 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 808K->0K(9216K)] [ParOldGen: 6152K->6764K(10240K)] 6960K->6764K(19456K), [Metaspace: 3227K->3227K(1056768K)], 0.0066343 secs] [Times: user=0.16 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 7325K->4096K(9216K)] [ParOldGen: 6764K->9832K(10240K)] 14089K->13928K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0079754 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 7482K->7168K(9216K)] [ParOldGen: 9832K->9832K(10240K)] 17314K->17000K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0080927 secs] [Times: user=0.16 sys=0.00, real=0.01 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 7168K->7168K(9216K)] [ParOldGen: 9832K->9814K(10240K)] 17000K->16982K(19456K), [Metaspace: 3228K->3228K(1056768K)], 0.0079906 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 9216K, used 7371K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 89% used [0x00000000ff600000,0x00000000ffd32f18,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 9814K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 95% used [0x00000000fec00000,0x00000000ff595978,0x00000000ff600000)
 Metaspace       used 3259K, capacity 4500K, committed 4864K, reserved 1056768K
  class space    used 353K, capacity 388K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at referenceTest.TestReference$ReferenceTest.testPhantomReference(TestReference.java:59)
	at referenceTest.TestReference.main(TestReference.java:69)
```

## 引用队列

**GC 线程回收对象 -> 将相关指向这个对象的引用加入到其引用队列（如果有）-> 更新引用入队状态（`isEnqueued` 方法返回 true）-> 在 Java 代码中可以得到引用队列中的已经入队的引用（即得到要回收对象的对应引用对象，作为对象回收的一个通知）。**

测试：

```java
public class TestReferenceQueue {
    ReferenceQueue<byte[]> referenceQueue=new ReferenceQueue<>();
    void testReferenceNotify(){
        WeakReference<byte[]> weakReference=new WeakReference<>(new byte[1024],referenceQueue);
        // 后面的 ReferenceQueue.remove 方法会阻塞调用线程，因此开子线程进行操作
        Thread thread=new Thread(()->{
            try {
                for (Reference pr;(pr=referenceQueue.remove())!=null;){
                    System.out.println(pr+" 引用指向对象被回收！");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        /* 
		 因为 ReferenceQueue 对象的 remove 方法是阻塞线程的，因此子线程需设置守护线程，
		 否则如果 ReferenceQueue 中没有可取出的引用对象会导致线程一直阻塞，程序不能退出
		*/
        thread.setDaemon(true);
        thread.start();
        // 启动垃圾回收动作，将弱引用指向的对象回收
        System.gc();
    }

    public static void main(String[] args) {
        TestReferenceQueue test=new TestReferenceQueue();
        test.testReferenceNotify();
    }
}
```

原理：

```java
// 弱引用为例，WeakReference继承Reference类
public class WeakReference<T> extends Reference<T> {
    
    public WeakReference(T referent) {
        super(referent);
    }

    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}
// Reference类源码：在 Reference 类中的 enqueue 方法（这个方法本身会被 GC 线程调用）
// 中发现其直接调用了对应引用队列（ReferenceQueue）的 enqueue 方法
public abstract class Reference<T> {

    private T referent;         /* Treated specially by GC */

    volatile ReferenceQueue<? super T> queue;

    /* When active:   NULL
     *     pending:   this
     *    Enqueued:   next reference in queue (or this if last)
     *    Inactive:   this
     */
    @SuppressWarnings("rawtypes")
    Reference next; // 引用所处的状态不同时，该属性保存了不同的信息
    
	// ...
    
    /**
     * 获取当前引用所指向的对象的方法，如果所指向对象已经被 GC 回收，那么返回 null
     */
    public T get() {
        return this.referent;
    }

    /**
     * 清除该引用所指向的对象，该方法会在 GC 回收该引用指向的对象后被 GC 调用，
     * 之后，通过该引用对象的 get 方法得到的返回值为 null, 该方法不应该被程序员主动调用
     */
    public void clear() {
        this.referent = null;
    }

    /* -- Queue operations -- */

    /**
     * 判断当前引用是否已经进入对应的引用队列，
     * 如果构造该引用对象时没有指定对应的引用队列，那么该方法始终返回 false
     */
    public boolean isEnqueued() {
        return (this.queue == ReferenceQueue.ENQUEUED);
    }

    /**
     * 如果当前引用对象的引用队列属性（构造时由参数指定）不为 null, 
     * 那么当这个引用所指向的对象被 GC 回收之后会由 GC 调用这个方法，
     * 代表将该引用进入对应的引用队列（即该引用指向的对象被回收）
     */
    public boolean enqueue() {
        return this.queue.enqueue(this);
    }

    /* -- Constructors -- */
    Reference(T referent) {
        this(referent, null);
    }

    Reference(T referent, ReferenceQueue<? super T> queue) {
        this.referent = referent;
        this.queue = (queue == null) ? ReferenceQueue.NULL : queue;
    }
}

// ReferenceQueue 类的这个方法：
public class ReferenceQueue<T> {

    /**
     * Constructs a new reference-object queue.
     */
    public ReferenceQueue() { }

    // ...

    static private class Lock { };
    private Lock lock = new Lock();
    private volatile Reference<? extends T> head = null;
    private long queueLength = 0;

    // 引用对象本身入队列的过程就是一个向单向链表中插入节点的过程
    boolean enqueue(Reference<? extends T> r) { /* Called only by Reference class */
        synchronized (lock) { // 保证线程安全
            // Check that since getting the lock this reference hasn't already been
            // enqueued (and even then removed)
            ReferenceQueue<?> queue = r.queue;
            if ((queue == NULL) || (queue == ENQUEUED)) {
                return false;
            }
            assert queue == this;
            r.queue = ENQUEUED; // 更新引用入队状态
           	// 前插法插入链表节点
            r.next = (head == null) ? r : head;
            head = r;
            queueLength++;
            if (r instanceof FinalReference) {
                sun.misc.VM.addFinalRefCount(1);
            }
            lock.notifyAll();
            return true;
        }
    }

    /**
     * 返回当前引用队列中的第一个引用对象，如果不存在则返回 null
     * 该方法不会阻塞线程
     */
    public Reference<? extends T> poll() {
        if (head == null)
            return null;
        synchronized (lock) {
            return reallyPoll();
        }
    }

    /**
     * 返回当前引用队列中第一个可用的引用对象，如果没有，则阻塞线程一定时间（参数指定）
     * 阻塞时间过后，如果当前队列中仍然没有可用的引用对象，那么抛出中断异常（InterruptedException）
     */
    public Reference<? extends T> remove(long timeout)
        throws IllegalArgumentException, InterruptedException
    {
        if (timeout < 0) {
            throw new IllegalArgumentException("Negative timeout value");
        }
        synchronized (lock) {
            Reference<? extends T> r = reallyPoll();
            if (r != null) return r;
            long start = (timeout == 0) ? 0 : System.nanoTime();
            for (;;) {
                lock.wait(timeout);
                r = reallyPoll();
                if (r != null) return r;
                if (timeout != 0) {
                    long end = System.nanoTime();
                    timeout -= (end - start) / 1000_000;
                    if (timeout <= 0) return null;
                    start = end;
                }
            }
        }
    }

    /**
     * 阻塞调用线程，直到当前引用队列中存在可用的引用对象，将该引用对象从引用队列中移除并返回该引用对象
     */
    public Reference<? extends T> remove() throws InterruptedException {
        return remove(0);
    }
}
```

