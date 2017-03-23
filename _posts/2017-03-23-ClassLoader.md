---
layout: post
title: java HotSpot
---

+ 绑定：将一个方法的调用和方法所在的类进行关联<br>
>静态绑定：程序编译期的绑定，只有final、static、private和构造<br>
>动态绑定：运行时绑定，几乎所有的方法都是后期绑定<br>


[福利传送门](http://wiki.jikexueyuan.com/project/java-vm/)
### java 编译和执行的三大机制
+ java源码编译机制
+ 类加载机制
+ 类执行机制

### 类加载机制

![类加载](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/jvmclass.gif)

1.类加载器
+ Bootstrap ClassLoader（启动类加载器，剩余的加载器必须由启动类加载器加载之后才能加载其他的类。）<br>
负责加载$JAVA_HOME中jre/lib/rt.jar里所有的 class，由 C++ 实现，不是 ClassLoader 子类。<br>
+ Extension ClassLoader<br>
负责加载Java平台中扩展功能的一些 jar 包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包。<br>
+ App ClassLoader<br>
负责记载 classpath 中指定的 jar 包及目录中 class。<br>
+ Custom ClassLoader<br>
属于应用程序根据自身需要自定义的 ClassLoader，如 Tomcat、jboss 都会根据 J2EE 规范自行实现 ClassLoader。<br>

对于任意一个类，需要由类加载器和类本身共同确定在java虚拟机中的唯一性。
2.类加载过程
+ 加载<br>
>通过一个类的全限定名来获取其定义的二进制字节流<br>
>将字节流所代表的静态存储结构转化为方法区的数据结构<br>
>在java堆中生成一个代表这个类的Class对象，作为对方法区中这些数据的访问入口<br>

+ 验证
+ 准备
+ 解析
+ 初始化
+ 使用
+ 卸载

其中加载、验证、准备、初始化的顺序固定, 解析的顺序不固定(可能涉及动态绑定, 此时会在初始化之后)


### 类执行机制
JVM 是基于栈的体系结构来执行 class 字节码的。线程创建后，都会产生程序计数器（PC）和栈（Stack），程序计数器存放下一条要执行的指令在方法内的偏移量，栈中存放一个个栈帧，每个栈帧对应着每个方法的每次调用，而栈帧又是有局部变量区和操作数栈两部分组成，局部变量区用于存放方法中的局部变量和参数，操作数栈中用于存放方法执行过程中产生的中间结果。栈的结构如下图所示：

![类加载](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/classrun.gif)

