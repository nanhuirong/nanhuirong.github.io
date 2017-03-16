---
layout: post
title: java 集合类源码阅读
---
***

### CopyOnWriteArrayList


在List的衍生类中,包含ArrayList与LinkedList(都不是线程安全的),要实现一个线程安全的List必须调用
Collections.synchronizedList()方法,对于List的所有操作都进行隐式加锁.于此同时java提供了更加高
级的实现方式:CopyOnWriteArrayList(在读取数据的过程中不加锁,写入和删除操作需要进行加锁并进行数
组的深层拷贝),适用于读操作远远大于写操作的业务场景.

1.内部结构如下

```java

    final transient ReentrantLock lock = new ReentrantLock();
    private transient volatile Object[] array;
    
```

