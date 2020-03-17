# Set

- HashSet的性能总是比TreeSet好（尤其是常用操作为添加、查询元素的操作时），因为TreeSet需要额外的红黑算法树来维护集合元素的次序。只有需要使用一个排序的Set时，才应该使用TreeSet，否则应该使用HashSet；
- 对于普通的插入、删除操作，LinkedHashSet比HashSet要稍慢，这是由于需要维护链表所带来的开销，但是由于链表的存在，在遍历集合元素时，LinkedHashSet会更快；
- EnumSet是集合中性能最好的，但只能保存同一枚举类的枚举值为集合元素；
- HashSet、LinkedHashSet、TreeSet、EnumSet都是**线程不安全**的，可以使用Collections.synchronizedSortedSet方法来包装该Set集合。

## HashSet

```java
package top.songfang.connection.setTest;

import java.util.HashSet;

/**
 * 从以下可以看出，两个对象通过equals方法比较返回true，但两个对象的hashCode方法的返回不一样的值时，
 *   会导致HashSet将这两个对象保存在Hash表的不同位置，从而使对象可以添加成功，这与Set集合的规则有
 *   出入，因此需要明确，HashSet存在如下两个特征：
 *   1. equals()决定是否可以加入HashSet；
 *   2. 而hashCode决定存放的位置；
 *   3. 以上两个条件同时满足才允许一个新元素加入HashSet
 * 注意：
 *   1. 两个对象的hashCode相同，而equals返回值不同，HashSet会在这个位置使用链式结构来保存对各对象；
 *   2. 而HashSet快速访问元素时也是根据元素的HashCode值来快速定位元素，这种链式结构会导致性能下降；
 *   因此，需要将某个类对象保存到HashSet集合中时，在重写equals和hashCode方法时应尽量保证两个对象通过
 *   equals方法返回true时，它们的hashCode方法返回值也相等。
 */
public class HashSetTest {

    static class A{
        @Override
        public boolean equals(Object obj) {
            return true;
        }
    }
    static class B{
        @Override
        public int hashCode() {
            return 1;
        }
    }
    static class C{
        @Override
        public int hashCode() {
            return 2;
        }

        @Override
        public boolean equals(Object obj) {
            return true;
        }
    }

    public static void main(String[] args) {
        HashSet<Object> hashSet=new HashSet<>();
        // 往HashSet中分别添加两个A、B、C对象
        hashSet.add(new A());
        hashSet.add(new A());
        hashSet.add(new B());
        hashSet.add(new B());
        hashSet.add(new C());
        hashSet.add(new C());
        // 返回B@1,B@1,C@2,A@4554617c,A@1b6d3586
        System.out.println(hashSet);
    }
}
```

## LinkedHashSet

```java
package top.songfang.connection.setTest;

import java.util.LinkedHashSet;

/**
 * LinkedHashSet:
 *   特性：元素的顺序总是与元素的添加顺序一致，且不允许元素重复
 */
public class LinkedHashSetTest {
    public static void main(String[] args) {
        LinkedHashSet<String> books=new LinkedHashSet<>();
        books.add("java");
        books.add("jvm");
        books.add("jsp");
        books.add("Html");
        books.add("ASP");
        // [java, jvm, jsp, Html, ASP]
        System.out.println(books);
        books.remove("java");
        // [jvm, jsp, Html, ASP]
        System.out.println(books);
        books.add("ASP");
        // [jvm, jsp, Html, ASP]
        System.out.println(books);
    }
}
```

## TreeSet

```java
package top.songfang.connection.setTest;

import java.util.Comparator;
import java.util.TreeSet;

/**
 *   1. CompareTo、equals：关注比较字段的使用；
 *   2. 自然排序、定制排序、Comparator关注使用什么顺序进行排序。
 *
 * TreeSet采用红黑树的数据结构来存储集合元素，支持两种排序方式:
 *   1. 自然排序
 *     TreeSet会调用集合元素的compareTo(Object o)方法来比较元素之间的大小关系，
 *     然后将集合元素按升序排列，即自然排序。如果试图把一个对象添加到TreeSet时，
 *     则该对象必须实现Comparable接口，否则程序会抛出异常。
 *  2. 定制排序
 *     TreeSet的自然排序是根据集合元素的大小，TreeSet将它们以升序排列。如果我们
 *     需要实现定制排序，则可以通过Comparator接口的帮助。该接口包含一个int CompareTo(T o1,T o2)
 *     方法，用于比较大小。
 *  将一个对象加入TreeSet集合中时，TreeSet会调用该对象的compareTo(Object obj)方法与容器中的
 *  其他方法进行比较，然后根据红黑树的结构找到它的存储位置。如果两个对象compareTo方法比较相等，
 *  则新对象无法加入TreeSet集合。
 *  注意：当需要把一个对象放入TreeSet中时，重写该对象对应类的equals方法时，应该保证该方法的compareTo
 *  方法有一致的结果。即两个对象equals方法比较为true，则这两个对象通过compareTo比较结果应为0（即相等）
 */
public class TreeSetTest {
    public static void main(String[] args) {
        TreeSet<Integer> nums=new TreeSet<>();
        nums.add(10);
        nums.add(-5);
        nums.add(36);
        nums.add(82);
        // 返回集合中的所有元素
        System.out.println(nums);
        // 返回集合里第一个元素
        System.out.println(nums.first());
        // 返回集合里最后一个元素
        System.out.println(nums.last());
        // 返回小于36的子集，不包含36 ：[-5, 10]
        System.out.println(nums.headSet(36));
        // 返回大于36的子集，包含36 ：[36, 82]
        System.out.println(nums.tailSet(36));
        // 返回大于10（包含），小于（不包含）36的子集 ：[10]
        System.out.println(nums.subSet(10,36));

        // 测试定制排序
        TreeSet<M> ts=new TreeSet<M>(new Comparator<M>() {
            @Override
            public int compare(M o1, M o2) {
                return Integer.compare(o1.age, o2.age);
            }
        });
        ts.add(new M(5));
        ts.add(new M(10));
        ts.add(new M(100));
        System.out.println(ts);
    }
    static class M{
        int age;

        public M(int age) {
            this.age = age;
        }

        @Override
        public String toString() {
            return "M[" +
                    "age=" + age +
                    ']';
        }
    }
}
```

## EnumSet

```java
package top.songfang.connection.setTest;

import java.util.EnumSet;

/**
 * EnumSet: 以枚举类型为集合元素
 */
enum Season{
    SPRING("春天"),
    SUMMER("夏天"),
    FALL("秋天"),
    WINTER("冬天");
    String SEASON;

    Season(String SEASON) {
        this.SEASON = SEASON;
    }

    public String getSEASON() {
        return SEASON;
    }

    public void setSEASON(String SEASON) {
        this.SEASON = SEASON;
    }
}
public class EnumSetTest {
    public static void main(String[] args) {
        // 创建一个EnumSet集合，元素为Season枚举
        EnumSet<Season> enumSet=EnumSet.allOf(Season.class);
        // [SPRING, SUMMER, FALL, WINTER]
        System.out.println(enumSet);
        // 创建空集合
        EnumSet<Season> enumSet1=EnumSet.noneOf(Season.class);
        // []
        System.out.println(enumSet1);
        // 手动添加元素
        enumSet1.add(Season.SPRING);
        enumSet1.add(Season.WINTER);
        // [SPRING, WINTER]
        System.out.println(enumSet1);
        // 以指定枚举值创建集合
        EnumSet<Season> enumSet2=EnumSet.of(Season.WINTER,Season.FALL);
        // [FALL, WINTER]
        System.out.println(enumSet2);
        // 以范围值创建
        EnumSet<Season> enumSet3=EnumSet.range(Season.SUMMER,Season.WINTER);
        // [SUMMER, FALL, WINTER]
        System.out.println(enumSet3);
        // 以剩余值创建
        EnumSet enumSet4=EnumSet.complementOf(enumSet3);
        // [SPRING]
        System.out.println(enumSet4);
    }
}
```