---
layout: post
title: java 集合类源码阅读
---
***

![GitHub](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/javaCollectionFrame.png)
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
```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
        //已经没有办法扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //采用2倍扩容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```



