---
layout: post
title: java 集合类源码阅读
---
***

## Java基础类库
+ java.lang
+ java.util
+ java.util.concurrent
+ java.io

![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/Collections.png)

### ArrayList 与 LinkedList

1.ArrayList
+ 基于数组实现, 初始大小是16, 最大大小是int的最大值(下标是int型的)

+ 读取效率比较高O(1), 适合随机读取, 由于扩容时需要进行数组的拷贝, 写入的效率不高

2.LinkedList
+ 基于双向链表实现, 读取效率不高, 但是写入效率比较高, 适合顺序读取操作

***

### CopyOnWriteArrayList CopyOnWriteArraySet


在List的衍生类中,包含ArrayList与LinkedList(都不是线程安全的),要实现一个线程安全的List必须调用Collections.synchronizedList()方法,对于List的所有操作都进行隐式加锁.于此同时java提供了更加高级的实现方式: CopyOnWriteArrayList(在读取数据的过程中不加锁, 写入和删除操作需要进行加锁并进行数组的深层拷贝),适用于读操作远远大于写操作的业务场景.

1.内部结构如下

```java
//显式定义锁
final transient ReentrantLock lock = new ReentrantLock();
private transient volatile Object[] array;
```

2.实现细节
+ set方法
添加元素时需要加锁,否则多线程添加会copy出多个副本
```java
    public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

+ get方法
读取时如果有多个线程并发写入, 读取的内容还是旧的, 因为写入的时候锁住的不是旧的对象.
```java
    public E get(int index) {
        return get(getArray(), index);
    }
```

3.内部理论
copyOnWrite其实是一种读写分离的理论.
CopyOnWriteArraySet与之原理类似, 都是线程安全的.
可以基于此原理实现Map.

4.总结
CopyOnWrite主要有两个缺点:
+ 内存占用问题:写入时内存存在两份数据, 可能会造成频繁的minor gc 和 full gc
+ 数据一致性问题:CopyOnWrite只能保证数据的最终一致性, 无法保证数据的实时一致性.如果要保证数据的时效性, 请不要使用该容器

***

### HashTable HashMap TreeMap LinkedHashMap ConcurrentHashMap
Map用于保存<key, value>型数据, 在java中有4种常见的形式: HashTable, HashMap, TreeMap, LinkedHashMap.

1.HashTable(该API已经被弃用, 过时的方法就不总结太深)
+ 线程安全, 效率比较低(替代方案:Collections.synchronizedMap(), ConcurrentHashMap)
+ hash映射方式: key.hashCode % table.length, 取余运算比较耗时
+ <key, value> 均不能为null, key如果为null, hashCode方法调用失败, 抛出空指针异常.

2.HashMap(在JDK1.8中其实采用位桶 + 链表/红黑树 的实现形式)
+ 非线程安全, 允许<key, value>为空, 当key为空时放入0号桶内部,
+ 数据结构:
```java
    //内部维护一个数组,
    transient Node<K,V>[] table;
    //初始桶大小16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    //最大桶大小
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认负载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    //链表最大长度, 超过采用红黑树
    static final int TREEIFY_THRESHOLD = 8;
    //修改次数, 快速失败机制, 其实就是个傻逼
    transient int modCount;
    //阈值, 超过进行扩容操作  = (capacity * loadFactor)
    int threshold;
    //数组的每一个元素其实是一个链表
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
    }
```
+ hash优化:其实主要是取余操作的优化, 前提是每次扩容的capacity必须是2的幂次, 最高16位与最低16为进行运算, 主要用于减少碰撞
```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    取余数操作其实就是 (capacity - 1) & hash  等价于 hash % capacity
```
+ 扩容操作

3.LinkedHashMap
![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/LinkedHashMap.png)

+ 在HashMap的基础上采用双向链保证元素的插入顺序

4.TreeMap
+ 内部采用红黑树实现, 元素默认升序
+ 非线程安全, 可采用Collections.synchronizedMap()实现并发操作

5.ConcurrentHashMap
+ 线程安全
+ 内部实现采用锁分段机制, 在桶上设置n把锁, 每把锁管理固定区域
+ 迭代器不会抛出ConcurrentModificationException异常

***
### HashSet TreeSet
1.HashSet
+ 非线程安全, 内部维护一个HashMap, 只使用key, 所有的value字段没有使用, 都指向同一个地址

2.TreeSet
+ 非线程安全, 内部维护一个TreeMap, 只使用Key, 所有的value字段没有使用
+ ConcurrentSkipListMap 同步SortedMap的替代品
+ ConcurrentSkipListSet 同步SortedSet的替代品

3.List与Set接口
+ List代表有序集合, 这里的顺序指插入顺序
+ Set代表无序集合

***

### Queue
1.BlockingQueue(多线程容器)
+ LinkedBlockingQueue:
+ ArrayBlockingQueue
+ PriorityBlockingQueue:按优先级排序的队列
+ SynchronousQueue: 维护一组线程,

2.TransferQueue
3.ConcurrentLinkedQueue
+ 传统的先进先出的队列

4.PriorityQueue
+ 非并发的优先队列

5.双端队列, 工作密取, 在生产者-消费者模式中, 每个消费者有一个双端队列, 如果完成自己队列的工作可以从其他队尾取元素, 适合即使生产者又是消费者的问题
+ ArrayDeque
+ LinkedBlockingDeque




