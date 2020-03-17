# Map

## HashMap/Hashtable

- 使用自定义类作为HashMap、Hashable的key时，如果重写该类的equals和hashCode方法，则应该保证2个方法的判断标准一致-----当两个key通过equals()方法比较返回true时，两个key的hashCode()的返回值也应该相同。

```java
import java.util.Hashtable;

class A{
    int count;

    public A(int count) {
        this.count = count;
    }

    // 根据count比较两个对象是否相等
    @Override
    public boolean equals(Object obj) {
        if (obj==this){
            return true;
        }
        if (obj!=null && obj.getClass()==A.class){
            A a=(A)obj;
            return this.count==a.count;
        }
        return false;
    }

    // 根据count来计算hashCode的值
    @Override
    public int hashCode() {
        return this.count;
    }
}
class B{
    // 重写equals()方法，B对象与任何对象通过equals()方法比较时均相等
    @Override
    public boolean equals(Object obj) {
        return true;
    }
}
public class HashTableTest {
    public static void main(String[] args) {
        Hashtable hashtable=new Hashtable();
        hashtable.put(new A(60000),"疯狂java讲义");
        hashtable.put(new A(500),"java多线程");
        hashtable.put(new A(87562),"java编程入门");
        hashtable.put(new A(6223),new B());
        System.out.println(hashtable);

        // 只要两个对象通过equals比较返回true，
        // Hashtable就认为他们是相等的value，
        // 由于Hashtable中有一个B对象
        // 它与任何对象比较均相等,所以返回true
        System.out.println(hashtable.containsValue("Hello"));

        // 只要两个A对象的count相等，它们通过equals比较返回true，
        //  且hashCode相等；
        //  Hashtable即认为他们是相同的key，所以输出true
        System.out.println(hashtable.containsKey(new A(6223)));

        // 删除最后一个K-V对
        hashtable.remove(new A(6223));

        // 遍历Hashtable所有key组成的set集合
        // 从而遍历Hashtable每个k-v对
        for (Object key:hashtable.keySet()) {
            System.out.print(key+"--->");
            System.out.println(hashtable.get(key));
        }
    }
}
```

## LinkedHashMap

- 判断TreeMap中两个元素相等的标准：
  - 两个key通过compareTo()方法返回0；
  - equals()返回true；
- Set与Map的关系：先实现了HashMap、TreeMap等集合，然后通过包装了一个所有的value都为null的Map集合实现了Set集合类。
- HashMap和HashTable效率大致相同，通常HashMap比Hashtable快一点，因为Hashtable需要额外的线程同步控制；
- TreeMap比HashMap和Hashtable慢，尤其在插入、删除k-v时更慢，因为TreeMap底层采用红黑树管理k-v对；
- TreeMap好处：k-v对总是处于有序状态，无序进行专门排序操作。

```java
import java.util.LinkedHashMap;
import java.util.SortedMap;
import java.util.TreeMap;

/**
 * LinkedHashMap:按插入顺序保存；
 * TreeMap:按顺序保存
 */
class R implements Comparable{
    private int count;

    public R(int count) {
        this.count = count;
    }

    @Override
    public String toString() {
        return "R{" +
                "count=" + count +
                '}';
    }

    // 根据count值判断两对象是否相等
    @Override
    public boolean equals(Object obj) {
        if(obj==this){
            return true;
        }
        if(obj!=null && obj.getClass()==R.class){
            R r=(R)obj;
            return r.count==this.count;
        }
        return false;
    }

    // 根据count属性值来判断两个对象的大小
    @Override
    public int compareTo(Object o) {
        R r=(R)o;
        return Integer.compare(this.count,r.count);
    }
}
public class LinkedHashMapTest {
    public static void main(String[] args) {
        LinkedHashMap<String,Integer> scores=new LinkedHashMap<>();
        scores.put("语文",80);
        scores.put("英语",88);
        scores.put("数学",90);
        for (String key:scores.keySet()) {
            System.out.println(key+"--->"+scores.get(key));
        }

        TreeMap<R, String > tm=new TreeMap<>();
        tm.put(new R(10),"轻量级java");
        tm.put(new R(4),"HTML入门");
        tm.put(new R(-8),"ASP学习");
        System.out.println(tm);
        // 返回第一个Entry对象
        System.out.println(tm.firstEntry());
        // 返回最后一个Entry的key
        System.out.println(tm.lastKey());
        // 返回该TreeMap的比new R(4)大的最小key值,不含new R(4)
        System.out.println(tm.higherKey(new R(4)));
        // 返回比new R(4)小的最大entry,不包含new R(4)
        System.out.println(tm.lowerEntry(new R(4)));
        // 返回子TreeMap,[-8,4)
        SortedMap<R,String> subTm=tm.subMap(new R(-8),new R(4));
        System.out.println(subTm);

    }
}
```