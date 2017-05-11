---
layout: post
title: Hibernate
---

![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/hibernate/Hibernate产品介绍.png)

1.Hibernate简介
>基于JDBC的轻量级封装的ORM框架，使用反射机制，充当项目的持久化层。<br>

+ 分层架构<br>
>1）表现层<br>
>2）业务层<br>
>3）持久层：最直接的方式是使用JDBC和SQL<br>
>4）数据库<br>
+ ORM<br>
>对象关系映射，利用描述对象和数据库之间映射的元数据，自动把java对象持久化到关系数据库的表。<br>

2.Hibernate jar
+ hibernate core<br>
>hibernate 核心jar<br>
+ hibernate annotation<br>
><br>
+ cglib-asm.jar<br>
>实现PO字节码的动态生成<br>
+ odmg.jar<br>
>odmg是一个orm规范<br>
+ antlr<br>
>语言转换工,Hibernate利用它实现 HQL 到 SQL的转换<br>
+ dom4j-1.6.jar<br>
>dom4j XML 解析器<br>
+ hibernate entityManager<br>
><br>
+ hibernate shards<br>
><br>
+ hibernate search<br>
><br>
+ hibernate validator<br>
><br>
+ <br>
><br>
+ <br>
><br>

3.hibernate核心组建
+ Configuration类<br>
>它用来读取Hibernate的配置文件，并生成SessionFactory对象。Hibernate的配置文件有全局配置文件（hibernate.properties或hibernate.cfg.xml）和映射文件（*.hbm.xml）。<br>
+ SessionFactory接口<br>
>这是Hibernate的关键对象，它是单个数据库映射关系经过编译后的内存镜像，它也是线程安全的。它是生成Session的工厂，本身要应用到ConnectionProvider，该对象可以在进程和集群的级别上，为那些事务之间可以重用的数据提供可选的二级缓存。<br>
+ Session接口<br>
>它是应用程序和持久存储层之间交互操作的一个单线程对象。它也是Hibernate持久化操作的关键对象，所有的持久化对象必须在Session的管理下才能够进行持久化操作。此对象的生存周期很短，其隐藏了JDBC连接，也是Transaction 的工厂。Session对象有一个一级缓存，现实执行Flush之前，所有的持久化操作的数据都在缓存中Session对象处。<br>
+ Transaction接口<br>
>代表一次原子操作，它具有数据库事务的概念。但它通过抽象，将应用程序从底层的具体的JDBC、JTA和CORBA事务中隔离开。在某些情况下，一个Session 之内可能包含多个Transaction对象。虽然事务操作是可选的，但是所有的持久化操作都应该在事务管理下进行，即使是只读操作。<br>
+ ConnectionProvider<br>
>它是生成JDBC的连接的工厂，同时具备连接池的作用。他通过抽象将底层的DataSource和DriverManager隔离开。这个对象无需应用程序直接访问，仅在应用程序需要扩展时使用。<br>

4.hibernate主键生成策略<br>
[主键生成策略](http://www.cnblogs.com/hoobey/p/5508992.html)<br>