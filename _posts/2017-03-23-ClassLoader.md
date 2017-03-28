---
layout: post
title: java HotSpot
---

+ 绑定：将一个方法的调用和方法所在的类进行关联<br>
>静态绑定：程序编译期的绑定，只有final、static、private和构造<br>
>动态绑定：运行时绑定，几乎所有的方法都是后期绑定<br>

+ 双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。<br>

[福利传送门](http://wiki.jikexueyuan.com/project/java-vm/)
### java 编译和执行的三大机制
+ java源码编译机制
+ 类加载机制
+ 类执行机制

### 类加载机制

![类加载](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/jvmclass.gif)

1.类加载器

![类加载器层次](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/loader.png)

+ Bootstrap ClassLoader（启动类加载器，剩余的加载器必须由启动类加载器加载之后才能加载其他的类。）<br>
负责加载$JAVA_HOME中jre/lib/rt.jar里所有的 class，由 C++ 实现，不是 ClassLoader 子类。<br>
+ Extension ClassLoader<br>
负责加载Java平台中扩展功能的一些 jar 包，包括$JAVA_HOME中jre/lib/*.jar或-Djava.ext.dirs指定目录下的jar包。<br>
+ App ClassLoader<br>
负责记载 classpath 中指定的 jar 包及目录中 class。<br>
+ Custom ClassLoader<br>
属于应用程序根据自身需要自定义的 ClassLoader，如 Tomcat、jboss 都会根据 J2EE 规范自行实现 ClassLoader。<br>

对于任意一个类，需要由类加载器和类本身共同确定在java虚拟机中的唯一性。

这种层次关系被称为类加载器的双亲委派模型，他们之间的父子关系不是通过继承来实现的，而是使用组合关系来复用父加载器中的代码。
2.类加载过程
+ 加载<br>
>通过一个类的全限定名来获取其定义的二进制字节流<br>
>将字节流所代表的静态存储结构转化为方法区的数据结构<br>
>在java堆中生成一个代表这个类的Class对象，作为对方法区中这些数据的访问入口<br>
+ 验证(确保 Class 文件中的字节流包含的信息符合当前虚拟机的要求，而且不会危害虚拟机自身的安全。)<br>
>文件格式验证：验证字节流是否符合 Class 文件格式的规范，并且能被当前版本的虚拟机处理，该验证的主要目的是保证输入的字节流能正确地解析并存储于方法区之内。经过该阶段的验证后，字节流才会进入内存的方法区中进行存储，后面的三个验证都是基于方法区的存储结构进行的。<br>
>元数据验证：对类的元数据信息进行语义校验（其实就是对类中的各数据类型进行语法校验），保证不存在不符合 Java 语法规范的元数据信息。<br>
>字节码验证：该阶段验证的主要工作是进行数据流和控制流分析，对类的方法体进行校验分析，以保证被校验的类的方法在运行时不会做出危害虚拟机安全的行为。<br>
>符号引用验证：这是最后一个阶段的验证，它发生在虚拟机将符号引用转化为直接引用的时候（解析阶段中发生该转化，后面会有讲解），主要是对类自身以外的信息（常量池中的各种符号引用）进行匹配性的校验。<br>
>出错会VerifyError<br>
+ 准备(正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。)<br>
>这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在 Java 堆中。<br>
>这里所设置的初始值通常情况下是数据类型默认的零值（如 0、0L、null、false 等），而不是被在 Java 代码中被显式地赋予的值。<br>
+ 解析(将常量池中的符号引用转化为直接引用的过程。包含静态、私有方法、构造器和父类方法[编译可知，运行不可变])
+ 初始化(执行类构造器方法的过程)
+ 使用
+ 卸载

其中加载、验证、准备、初始化的顺序固定, 解析的顺序不固定(可能涉及动态绑定, 此时会在初始化之后)<br>
如果没有找到类或者接口的二进制表示，对应NoClassDefFound，类的格式出现语法错误会ClassFormatError、UnSupportedClassVersionError，类的继承层次有错会ClassCircularityError，如果索引用的直接接口本身并不是接口或者直接超类实际上是接口会IncompatibleClassChangeError<br>



### 类执行机制
JVM 是基于栈的体系结构来执行 class 字节码的。线程创建后，都会产生程序计数器（PC）和栈（Stack），程序计数器存放下一条要执行的指令在方法内的偏移量，栈中存放一个个栈帧，每个栈帧对应着每个方法的每次调用，而栈帧又是有局部变量区和操作数栈两部分组成，局部变量区用于存放方法中的局部变量和参数，操作数栈中用于存放方法执行过程中产生的中间结果。栈的结构如下图所示：

![类加载](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/classrun.gif)

### Class文件结构
[class文件结构详解](http://wiki.jikexueyuan.com/project/java-vm/class.html)


### 语法糖
包含泛型、变长参数、条件编译、自动装拆箱、内部类等。虚拟机不支持这些语法、在编译阶段被还原成简单的基础语法结构。

1.泛型，JDK1.5引入
