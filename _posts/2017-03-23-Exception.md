---
layout: post
title: Exception详解
---
***

![Exception 继承](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/Exception.jpg)

+ 可查异常: 除了RuntimeException及其子类的其他Exception, java编译器一定会检查.
+ 不可查异常: RuntimeException 和 Error.<br>
>运行时异常;<br>
>编译异常.<br>


1.Throwable 两个重要的子类, 根本在于程序是否能处理.
+ Exception: 异常
+ Error: 错误

2.Error<br>
不做任何处理.<br>
程序无法处理的错误, 大多错误与code无关, 主要是JVM出了问题(包括Virtual MachineError, NoClassDefFoundError).

3.Exception<br>
程序本身可以处理的异常,

4.RuntimeException<br>
由java运行时系统自行抛出, 允许程序忽略该异常.
+ NullPointerException 空指针异常.当应用试图在要求使用对象的地方使用了null时,抛出该异常.譬如：调用null对象的实例方法,访问null对象的属性,计算null对象的长度,使用throw语句抛出null等等;
+ ArithmeticException 算术条件异常,譬如:整数除零等;
+ ArrayIndexOutOfBoundException 数组索引越界异常,当对数组的索引值为负数或大于等于数组大小时抛出.
+ java.lang.ClassNotFoundException 找不到类异常.当应用试图根据字符串形式的类名构造类,而在遍历CLASSPAH之后找不到对应名称的class文件时,抛出该异常
+ NegativeArraySizeException 数组长度为负异常.
+ ArrayStoreException 数组中包含不兼容的值抛出的异常.
+ SecurityException 安全性异常.
+ IllegalArgumentException 非法参数异常

5.异常处理机制: 抛出异常, 捕获异常.<br>
当运行时系统遍历调用栈而未找到合适的异常处理器,则运行时系统终止,同时,意味着Java程序的终止.<br>
+ 异常总是先抛出, 再捕获.
+ 允许忽略RuntimeException和Error.

6.IOException
+ IOException：操作输入流和输出流时可能出现的异常
+ EOFException   文件已结束异常
+ FileNotFoundException   文件未找到异常

7.其他异常
+ ClassCastException    类型转换异常类
+ ArrayStoreException  数组中包含不兼容的值抛出的异常
+ SQLException   操作数据库异常类
+ NoSuchFieldException   字段未找到异常
+ NoSuchMethodException   方法未找到抛出的异常
+ NumberFormatException    字符串转换为数字抛出的异常
+ StringIndexOutOfBoundsException 字符串索引超出范围抛出的异常
+ IllegalAccessException  不允许访问某类异常
+ InstantiationException  当应用程序试图使用Class类中的newInstance()方法创建一个类的实例，而指定的类对象无法被实例化时，抛出该异常

