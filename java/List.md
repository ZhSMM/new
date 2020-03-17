# List

1. java提供的List是一个线性表接口，ArrayList（基于数组）和LinkedList（基于链表）属于两种典型实现；

2. ArrayList底层基于数组，基于一块连续内存，所以随机访问性能最好；

3. LinkedList底层基于链表，在元素插入、删除方面性能较好；

4. 迭代操作时，以链表为底层的集合比以数组为底层的集合性能好；

    ```java
    import java.util.ArrayList;
    import java.util.List;

    /**
     * ArrayList：
     *   底层有定长数组实现，支持动态扩容，指定初始initialCapacity可以减少重新分配
     *   次数，提升性能。ArrayList提供以下方法重新分配Object[]数组：
     *   1. ensureCapacity(int minCapacity):将ArrayList集合Object[]数组长度增加到
     *     minCapacity；
     *   2. trimToSize()：调整ArrayList集合的Object[]数组长度为当前元素的个数，可以
     *     使用此方法减少对象所占用的内存空间。
     */
    public class ArrayListTest {
        public static void main(String[] args) {
            ArrayList<String> books=new ArrayList<>(9);
            books.add("轻量级java实战");
            books.add("疯狂java讲义");
            books.add("HTML教程");
            books.add("疯狂java讲义");
            books.add("HTML教程");
            System.out.println(books);
            System.out.println(books.size());
            // 调整集合大小
            books.trimToSize();
            System.out.println(books.size());

            // 将新字符串插入到第二个位置
            books.add(1,new String("傻瓜式java"));
            System.out.println(books);

            // 删除第三个元素
            System.out.println(books.remove(2));

            // 返回指定元素在集合中的位置
            System.out.println(books.indexOf("HTML教程"));  //2
            System.out.println(books.lastIndexOf("HTML教程"));//4
            System.out.println(books.indexOf("ff"));//-1

            // 替换指定元素字符串
            books.set(2,"Hello java");
            System.out.println(books);

            // 截取子集和[1,4)
            List<String> subBooks=books.subList(1,4);
            for (String item:subBooks) {
                System.out.println(item);
            }

            //
        }
    }
    ```
    
5. Collection接口继承了Iterable接口，说明List集合类具有可遍历性；

    ```java
    // Iterable接口方法：
    // 1.boolean hasNext():是否存在下一个未遍历元素
    // 2.Object next():返回集合中下一元素
    // 3.void remove():删除集合中上一次返回next方法的元素
    
    import java.util.ArrayList;
    import java.util.Iterator;
    import java.util.List;
    
    public class IterableTest {
        public static void main(String[] args) {
            // 创建集合
            List<String> list= new ArrayList<>(9);
            list.add("One");
            list.add("Two");
            list.add("Three");
            list.add("Four");
            list.add("Five");
            // 获得迭代器
            Iterator<String> it= list.iterator();
            while (it.hasNext()){
                // next返回Object，进行转换
                String item=(String) it.next();
                System.out.println(item);
                if (item.equals("Four")){
                    // 从集合中删除上一次next返回的元素
                    it.remove();
                }
            }
            for (String item:list) {
                System.out.println(item);
            }
        }
    }
    
    // ListIterator:继承Iterator，专门操作List，除了以上方法，还有：
    // 1.boolean hasPrevious():返回迭代器关联的集合是否还有上有一个元素
    // 2.Object previous():返回该迭代器的上一个元素（向前迭代）
    // 3.void add():在指定位置上插入一个元素
    ```

    