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

3.hibernate主键生成策略<br>
[主键生成策略](http://www.cnblogs.com/hoobey/p/5508992.html)<br>

![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/hibernate/hibernate核心类和接口.png)

4.事务
+ 全局事务<br>
>又称JTA，跨数据库。<br>
+ 本地事务
>在同一数据库内。<br>

5.Session
>线程不安全，持久化接口，短生命周期对象，封装JDBC链接对象，作为一个事务工厂，
维护domain对象的持久化内容。<br>
+ get 与load<br>
>get：直接返回实体类，如果查不到数据返回null。先到缓存中查，如果不存在，立即去DB
中查找。<br>
>load：返回实体代理对象，如果数据不存在抛出异常。先到缓存（session缓存/二级缓存）
中查找，如果找不到，当需要使用时去数据库查询（支持延时加载）<br>
>确定对象一定存在，用load，否则用get。<br>

6.hibernate缓存
+ 一级缓存<br>
>又称session查询，存储SQL语句和domain对象。<br>
+ 二级缓存<br>
>load查找到的对象会缓存在二级缓存中。<br>

7.Query接口
>其实Criteria接口类似于Query接口，官方推荐Query接口进行查询。<br>
><br>
+ <br>
><br>
+ <br>
><br>
+ <br>
><br>

8.对象映射关系
+ one-one<br>
>1）基于主键：<br>
>2) 基于外键：<br>
+ one-many<br>
><br>
+ many-one<br>
><br>
>懒加载：查询一个对象，只默认返回对象的普通属性，当使用对象属性时，再次向DB发出查询。<br>
> 1）关闭懒加载。<br>
> 2）Hibernate.initialize()<br>
+ many-many<br>
>尽量通过一对多的关系进行避免，Hibernate会将多对多转换成两个一对多的关系。<br>

![](https://raw.githubusercontent.com/nanhuirong/nanhuirong.github.io/master/_posts/hibernate/hibernate对象状态.gif)<br>

9.Hibernate对象状态
+ 瞬时（自由）状态<br>
>通过new创建的对象，特点是不和Session进行关联，在DB中不存在和对象关联的记录。<br>
+ 持久状态<br>
>已经被保存到DB的实体对象，并且还处在hibernate的缓存管理中。持久对象总是与 Session 和 Transaction 相关联，在一个 Session 中，对持久对象的改变不会马上对数据库进行变更，而必须在 Transaction 终止，也就是执行 commit() 之后，才在数据库中真正运行 SQL 进行变更，持久对象的状态才会与数据库进行同步。在同步之前的持久对象称为脏 (dirty) 对象。<br>
+ 托管（游离）状态<br>
>当一个持久化对象脱离hibernate的缓存管理（session关闭）时，处于游离状态（与自由状态的区别在于在DB中可能存在对应的记录）<br>

10.级联（cascade）
>当进行一个操作时，另外的操作自动完成。常用的cascade:none,all,save-update,delete, lock,refresh,evict,replicate,persist,merge,delete-orphan(one-to-many)。<br>
