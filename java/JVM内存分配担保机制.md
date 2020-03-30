# JVM内存分配担保机制

JVM内存分配担保机制：指在新生代无法分配内存的时候，把新生代的对象转移到老年代，把新对象放入到腾空的新生代。

## 客户端模式的担保机制

使用客户端模式下的Serial+Serial Old收集器进行垃圾回收：java -XX:+UseSerialGC

设置jvm的参数：

java -Xms20M -Xmx20M -Xmn10M -XX:+printGCDetails -XX:SurvivorRatio=8 -XX:+UseSerialGC

- -Xms20M：最大堆内存20M；
- -Xmx20M：初始堆内存20M；
- -Xmn10M：新生代10M；
- -XX:+printGCDetails：打印GC细节；
- -XX:SurvivorRatio=8：Survivor与eden区的比率1：8，即eden区8M；
- -XX:+UseSerialGC：设置Serial GC；
- -XX：-HandlePromotionFailure(忽略)：担保机制开启，jdk1.5默认关闭，之后默认开启；

示例：

```java
public class TestHandlePromotionFailure {
    private static final int _1MB=1024*1024; // 1M内存

    public static void testHandlePromotion(){
        byte[]  allocation1,
                allocation2,
                allocation3,
                allocation4;
        allocation1=new byte[2*_1MB];
        allocation2=new byte[2*_1MB];
        allocation3=new byte[2*_1MB];
        allocation4=new byte[4*_1MB];
    }
    public static void main(String[] args) {
        testHandlePromotion();
    }
}
```

输出：

```
[GC (Allocation Failure) [DefNew: 8002K->619K(9216K), 0.0054738 secs] 8002K->6763K(19456K), 0.0055213 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
Heap
 def new generation   total 9216K, used 4797K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  51% used [0x00000000fec00000, 0x00000000ff014930, 0x00000000ff400000)
  from space 1024K,  60% used [0x00000000ff500000, 0x00000000ff59ae20, 0x00000000ff600000)
  to   space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
 tenured generation   total 10240K, used 6144K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
   the space 10240K,  60% used [0x00000000ff600000, 0x00000000ffc00030, 0x00000000ffc00200, 0x0000000100000000)
 Metaspace       used 3230K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

由日志`[GC (Allocation Failure) [DefNew: 8002K->619K(9216K), 0.0054738 secs] 8002K->6763K(19456K), 0.0055213 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] `可以发现在分配allocation4的时候发生了一次Minor GC，让新生代由8002k到619k，但整个堆的占用只是由8002k到6763k，并没有多大的变化，这是因为前三个2M的对象都还存活，所以回收器并没有发现可回收的对象。

而触发gc的原因是前三个对象占用了6M的eden space，而allocation4需要4m的内存空间，而new generation   total 9216K，所以新生代无法装下allocation4对象，于是发生了一次Minor GC，而虚拟机发现eden的三个对象无法全部放入Survivor空间（可用内存1M）。

于是启动了内存分配的担保机制，将eden的三个对象转入老年代，然后将allocation4放入eden区。所以结果就是eden区占用51%（4M/8M），老年代被占用60%（6M/10M）。

## 服务端模式下的担保机制

使用服务端模式（**Parallel Scavenge+Serial Old的组合**）来测试担保机制实现：

修改GC组合为：-XX:+UseParallelGC

输出：

```
Heap
 PSYoungGen      total 9216K, used 8166K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 99% used [0x00000000ff600000,0x00000000ffdf9b80,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
  to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
 ParOldGen       total 10240K, used 4096K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 40% used [0x00000000fec00000,0x00000000ff000010,0x00000000ff600000)
 Metaspace       used 3230K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

将allocation4改为3M的输出：

```
[GC (Allocation Failure) [PSYoungGen: 8002K->840K(9216K)] 8002K->6992K(19456K), 0.0060401 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Ergonomics) [PSYoungGen: 840K->0K(9216K)] [ParOldGen: 6152K->6762K(10240K)] 6992K->6762K(19456K), [Metaspace: 3224K->3224K(1056768K)], 0.0070114 secs] [Times: user=0.14 sys=0.00, real=0.01 secs] 
Heap
 PSYoungGen      total 9216K, used 3154K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 38% used [0x00000000ff600000,0x00000000ff914930,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 6762K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 66% used [0x00000000fec00000,0x00000000ff29aa40,0x00000000ff600000)
 Metaspace       used 3230K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K
```

由上述输出可以看出：**当我们使用Server模式下的ParallelGC收集器组合（Parallel Scavenge+Serial  Old的组合）下，担保机制的实现和之前的Client模式下（SerialGC收集器组合）有所变化。在GC前还会进行一次判断，如果要分配的内存>=Eden区大小的一半，那么会直接把要分配的内存放入老年代中。否则才会进入担保机制。**