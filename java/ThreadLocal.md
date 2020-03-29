# ThreadLocal

多线程中解决线程安全问题：

- 产生原因：多个线程对同一临界区共享资源进行操作；

- 解决问题的常用方式：synchronized或lock控制临界区资源的同步；
- 缺点：会使未获的锁的线程进行阻塞等待。

java.lang.ThreadLocal：通过每个线程保存一份变量副本从而实现内存的线程隔离。

## 实现原理

每一个线程，其执行均是依靠Thread类的实例的start方法来启动线程，然后CPU来执行线程。每一个Thread类的实例的运行即为一个线程。若要每个线程（每个Thread实例）的变量空间隔离，则需要将这个变量的定义声明在Thread这个类中。这样，每个实例都有属于自己的这个变量的空间，则实现了线程的隔离。

1. 在Thread类中声明一个公共的类变量ThreadLocalMap，用以在Thread的实例中预占空间：

   `ThreadLocal.ThreadLocalMap threadLocals = null;`

2. 在ThreadLocal中创建一个内部类ThreadLocalMap，这个Map的key是ThreadLoca对象，value是set进去的ThreadLocal中泛型类型的值：

   `private void set(ThreadLocal<?> key, Object value) {...}`

3. 在new ThreadLocal时，只是简单的创建了个ThreadLocal对象，与线程还没有任何关系；

4. 真正产生关系的是在向ThreadLocal对象中set值得时候：

   - 首先从当前的线程中获取ThreadLocalMap，如果为空，则初始化当前线程的ThreadLocalMap；
   - 然后将值set到这个Map中去，如果不为空，则说明当前线程之前已经set过ThreadLocal对象了（这样用一个ThreadHashMap来存储当前线程的若干个可以线程间隔离的变量，key是ThreadLocal对象，value是要存储的值（类型是ThreadLocal的泛型））；

   ```java
    public void set(T value) {
           Thread t = Thread.currentThread();
           ThreadLocalMap map = getMap(t);
           if (map != null)
               map.set(this, value);
           else
               createMap(t, value);
       }
   ```

5. 从ThreadLocal中获取值 ：还是先从当前线程中获取ThreadLocalMap,然后使用ThreadLocal对象(key)去获取这个对象对应的值(value)；

   ```java
   public T get() {
       Thread t = Thread.currentThread();
       ThreadLocalMap map = getMap(t);
       if (map != null) {
           ThreadLocalMap.Entry e = map.getEntry(this);
           if (e != null) {
               @SuppressWarnings("unchecked")
               T result = (T)e.value;
               return result;
           }
       }
       return setInitialValue();
   }
   ```

6. 简单总结

   ThreadLocal实现线程隔离的原理：通过在ThreadLocal类中声明ThreadLocalMap这个类，然后在使用ThreadLocal对象set值的时候将当前线程（Thread实例）进行map初始化，并将ThreadLocal对应的值放入map中，下次get的时候，也是使用ThreadLocal的对象（key）去当前线程的map中获取值（value）。

## ThreadLocalMap

从源码上看，ThreadLocalMap虽然叫做Map，但这个类并没有实现Map这个接口，只是定义在ThreadLocal中的一个静态内部类。只是因为在存储的时候也是以key-value的形式作为方法的入参暴露出去，所以称为map。

`static class ThreadLocalMap {...}`

1. ThreadLocalMap的创建，在使用ThreadLocal对象set值的时候，会创建ThreadLocalMap的对象，可以看到，入参就是KV，key是ThreadLocal对象，value是一个Entry对象，存储kv（HashMap是使用Node作为KV对象存储）。Entry的key是ThreadLocal对象，vaule是set进去的具体值。

   ```java
   void createMap(Thread t, T firstValue) {
       t.threadLocals = new ThreadLocalMap(this, firstValue);
   }
   ```

2. 其实ThreadLocalMap存储是一个Entry类型的数组，key提供了hashcode用来计算存储的数组地址（散列法解决冲突）:

   - 创建Entry数组（初始容量16）
   - 然后获取到key（ThreadLocal对象）的hashcode(是一个自增的原子int型)；

       ```java
       private final int threadLocalHashCode = nextHashCode();

       private static int nextHashCode() {
           return nextHashCode.getAndAdd(HASH_INCREMENT);
       }

       private static AtomicInteger nextHashCode = new AtomicInteger();
       ```

   - 使用【**hashcode 模(%) 数组长度**】的方式得到要将key存储到数组的哪一位。

   - 设置数组的扩容阈值，用以后续扩容

     ```java
     ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
         table = new Entry[INITIAL_CAPACITY];
         int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
         table[i] = new Entry(firstKey, firstValue);
         size = 1;
         setThreshold(INITIAL_CAPACITY);
     }
     ```

   - 创建ThreadLcoalMap对象只有在当前线程第一次插入kv的时候发生，如果是第二次插入kv，则会进行第三步

3. 这个set的过程其实就是根据ThreadLocal的hashcode来计算存储在Entry数组的位置

   - 利用ThreadLocal的【**hashcode 模(%) 数组长度**】的方式获取存储在数组的位置

   - 如果当前位置已存在值，则向右移一位，如果也存在值，则继续右移，直到有空位置出现为止

   - 将当前的value存储上面两部得到的索引位置（上面这两步就是散列法的实现）

   - 校验是否扩容，如果当前数组的中存储的值得数量大于阈值（数组长度的2/3），则扩容一倍，并将原来的数组的值重新hash至新数组中（这个过程其实就是HashMap的扩容过程）

     ```java
     private void set(ThreadLocal<?> key, Object value) {
         Entry[] tab = table;
         int len = tab.length;
         int i = key.threadLocalHashCode & (len-1);
     
         for (Entry e = tab[i];
              e != null;
              e = tab[i = nextIndex(i, len)]) {
             ThreadLocal<?> k = e.get();
     
             if (k == key) {
                 e.value = value;
                 return;
             }
     
             if (k == null) {
                 replaceStaleEntry(key, value, i);
                 return;
             }
         }
         tab[i] = new Entry(key, value);
         int sz = ++size;
         if (!cleanSomeSlots(i, sz) && sz >= threshold)
             rehash();
     }
     ```

## ThreadLocalMap和HashMap的比较

相同点：两个map都是最终用数组作为存储结构，使用key做索引，value是真正存储在数组索引上的值；

不同点：解决key冲突的方式

- HashMap采用链表法
- ThreadLocalMap采用散列法（又称开放地址法）

为什么不采用HashMap作为ThreadLocal的存储结构：

1. 引入链表，徒增了数据结构的复杂度，并且链表的读取效率较低；
2. 更加灵活。包括方法的定义和数组的管理，更加适合当前场景；
3. 不需要HashMap的额外的很多方法和变量，需要一个更加纯粹和干净map，来存储自己需要的值，减少内存的损耗。

## ThreadLocal的生命周期

1. 正常情况
   正常情况下（当然会有非正常情况），在线程退出的时候会将threadLocals这个变量置为null，等待JVM去自动回收。

   > 注意：Thread这个方法只是用以系统能够显示的调用退出线程，线程在结束的时候是不会调用这个方法，启动的线程是非守护线程，会在线程结束的时候由jvm自动进行空间的释放和回收。

   ```java
   private void exit() {
       if (group != null) {
           group.threadTerminated(this);
           group = null;
       }
       /* Aggressively null out all reference fields: see bug 4006245 */
       target = null;
       /* Speed the release of some of these resources */
       threadLocals = null;
       inheritableThreadLocals = null;
       inheritedAccessControlContext = null;
       blocker = null;
       uncaughtExceptionHandler = null;
   }
   ```

2. 非正常情况

   由于现在多线程一般都是由线程池管理，而线程池的线程一般都是复用的，这样会导致线程一直存活，而如果使用ThreadLocal大量存储变量，会使得空间开始膨胀

3. 需要自己来管理ThreadLocal的生命周期，在ThreadLocal使用结束以后及时调用remove（）方法进行清理。



