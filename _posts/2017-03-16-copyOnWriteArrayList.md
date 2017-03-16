---
layout: post
title: java 集合类源码阅读
---
***

### CopyOnWriteArrayList


在List的衍生类中,包含ArrayList与LinkedList(都不是线程安全的),要实现一个线程安全的List必须调用
Collections.synchronizedList()方法,对于List的所有操作都进行隐式加锁.于此同时java提供了更加高
级的实现方式: CopyOnWriteArrayList(在读取数据的过程中不加锁, 写入和删除操作需要进行加锁并进行数
组的深层拷贝),适用于读操作远远大于写操作的业务场景.

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



