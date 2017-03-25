---
layout: post
title: java内存泄漏
---

***

1.内存泄漏理论研究
>内存泄露是指分配出去的内存没有被回收回来，由于失去了对该内存区域的控制，因而造成了资源的浪费。Java 中一般不会产生内存泄露，因为有垃圾回收器自动回收垃圾，但这也不绝对，当我们 new 了对象，并保存了其引用，但是后面一直没用它，而垃圾回收器又不会去回收它，这边会造成内存泄露<br>

+ 内存泄漏原因，一般会出现OutOfMemoryError<br>
>有可能引起内存泄漏，一旦类自己管理内存，就应该警惕内存泄漏<br>
>缓存是另外一个内存泄漏的常见来源（可以用WeakHashMap代表缓存，缓存应该定时清除一下无用的项，由一个后台线程负责）<br>
>监听器和其他回调（最佳实现时支持弱引用）

+ 内存泄漏example

```java
//引起内存泄漏的例子
public class Stack {
    private Object[] elements;
    private int size;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    private void ensureCapacity(){
        if (elements.length == size){
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }

    public void push(Object obj){
        ensureCapacity();
        elements[size++] = obj;
    }

    public Object pop(){
        if (size == 0){
            throw new EmptyStackException();
        }
//        return elements[--size];会引起内存泄漏
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
}
```

2.JVM内存泄漏
+ 特征：<br>
>程序在不断重复某个操作时，内存一直处于动态的增长之中，并且到达JVM声明的最大内存而不释放任何heap内存。<br>
>某个流程已经处理完成，但是过程中增加的临时内存在介绍后却没有被垃圾回收<br>
>应用程序运行一段时间以后就得到OOM(Out of Memory)异常<br>

+ 图解<br>
>JVM正常活动图解<br>
![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/leakage-normal.gif)<br>
>JVM异常活动图解<br>
![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/leakage-nonormal.gif)<br>
