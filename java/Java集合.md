# Java集合

Collection：除了hashmap实现的是Map接口外，是其他的最上层接口。

iterator：迭代器，是实现iterable接口。

comparable：比较操作

## List接口

- ArrayList：
  - 底层数组实现
  - 1.5倍扩容策略
  - 初始容量的设定会影响性能
  - 线程不安全
- vector：
  - 底层数组实现
  - 2倍扩容
  - 线程安全
- linkedlist：
  - 双链表实现
  - 自带按索引访问的api
- copyonwritelist

## Map

- hashmap是数组和链表的组合结构，数组是一个Entry数组，entry是k-V键值对类型，所以一个entry数组存着很entry节点，一个entry的位置通过key的hashcode方法，再进行hash（移位等操作），最后与表长-1进行相与操作，其实就是取hash值到的后n - 1位，n代表表长是2的n次方；
- hashmap的默认负载因子是0.75，阈值是16 * 0.75 = 12；初始长度为16；
- hashmap的增删改查方式比较简单，都是遍历，替换。有一点要注意的是key相等时，替换元素，不相等时连成链表；
- 除此之外，1.8jdk改进了hashmap，当链表上的元素个数超过8个时自动转化成红黑树，节点变成树节点，以提高搜索效率和插入效率到logn；
- 还有一点值得一提的是，hashmap的扩容操作，由于hashmap非线程安全，扩容时如果多线程并发进行操作，则可能有两个线程分别操作新表和旧表，导致节点成环，查询时会形成死锁。chm避免了这个问题。
- 扩容时会将旧表元素移到新表，原来的版本移动时会有rehash操作，每个节点都要rehash，非常不方便，而1.8改成另一种方式，对于同一个index下的链表元素，由于一个元素的hash值在扩容后只有两种情况，要么是hash值不变，要么是hash值变为原来值+2^n次方，这是因为表长翻倍，所以hash值取后n位，第一位要么是0要么是1，所以hash值也只有两种情况。这两种情况的元素分别加到两个不同的链表。这两个链表也只需要分别放到新表的两个位置即可，是不是很酷。
- hashmap1.7版本链表使用的是节点的头插法，扩容时转移链表仍然使用头插法，这样的结果就是扩容后链表会倒置，而hashmap.1.8在插入时使用尾插法，扩容时使用头插法，这样可以保证顺序不变。

## CHM

- concurrenthashmap也稍微提一下把，chm1.7使用分段锁来控制并发，每个segment对应一个segmentmask，通过key的hash值相与这个segmentmask得到segment位置，然后在找到具体的entry数组下标。所以chm需要维护多个segment，每个segment对应一段数组。分段锁使用的是reetreetlock可重入锁实现，查询时不加锁。
- 1.8则放弃使用分段锁，改用cas+synchronized方式实现并发控制，查询时不加锁，插入时如果没有冲突直接cas到成功为止，有冲突则使用synchronized插入。

## Set

set就是hashmap将value固定为一个object，只存key元素，包装成一个entry即可，其他不变。

## Linkedhashmap

在原来hashmap基础上将所有的节点依据插入的次序另外连成一个链表。用来保持顺序，可以使用它实现lru缓存，当访问命中时将节点移到队头，当插入元素超过长度时，删除队尾元素即可。

使用的时候先继承linkedhashmap或者直接使用linkedhashmap作为成员变量，然后重写removeEldestEntry方法即可，注意传入size参数，判断当元素个数超过size时返回true，表示可以删除就行了。

## collections和Arrays工具类

两个工具类分别操作集合和数组，可以进行常用的排序，合并等操作。

## comparable和comparator

实现comparable接口可以让一个类的实例互相使用compareTo方法进行比较大小，可以自定义比较规则，comparator则是一个通用的比较器，比较指定类型的两个元素之间的大小关系。

## treemap和treeset

主要是基于红黑树实现的两个数据结构，可以保证key序列是有序的，获取sortedset就可以顺序打印key值了。其中涉及到红黑树的插入和删除，调整等操作，比较复杂，这里就不细说了。

另外我们也可以获取逆序的set集合。