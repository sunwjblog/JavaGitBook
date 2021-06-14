# JavaGitBook

### Java

#### 基础

1. [Java反射](javabase/reflect/什么是Java反射.md)

3. [代理模式总结](javabase/proxy/代理模式总结.md)

#### 容器

#### 多线程

1. [Java并发编程基础](thread/Java并发编程基础.md)

2. [线程间通信](thread/线程间通信.md)

3. [Java线程池](thread/线程池学习.md)

#### JVM

1. [JVM基础知识点](javabase/jvm/JVM知识点.md)

2. [JVM垃圾回收](javabase/jvm/JVM垃圾回收机制.md)

3. [类加载过程](javabase/jvm/类加载过程.md)

4. [类加载器](javabase/jvm/类加载器.md)

#### UML

1. [UML类图学习](javabase/uml/UML类学习总结.md)

#### 设计模式

1. [设计原则介绍](designpattern/设计原则.md)

   1. 创建型模式（Creational Pattern）
      1. [工厂模式](designpattern/creationalpattern/FactoryPattern.md)
      2. [抽象工厂模式](designpattern/creationalpattern/AbstractFactoryPattern.md)
      3. [单例模式](designpattern/creationalpattern/SingletonPattern.md)
      4. [建造者模式](designpattern/creationalpattern/BuilderPattern.md)
      5. [原型模式](designpattern/creationalpattern/ProtorypePattern.md)

   2. 结构型模式（Structural Pattern）
      1. [适配器模式](designpattern/structuralpattern/AdapterPattern.md)
      2. [桥接模式](designpattern/structuralpattern/BridgePattern.md)
      3. [过滤器模式](designpattern/structuralpattern/FilterCriteriaPattern.md)
      4. [组合模式](designpattern/structuralpattern/CompositePattern.md)
      5. [装饰器模式](designpattern/structuralpattern/DecoratorPattern.md)
      6. [外观模式](designpattern/structuralpattern/FacadePattern.md)
      7. [享元模式](designpattern/structuralpattern/FlyweightPattern.md)
      8. [代理模式](designpattern/structuralpattern/ProxyPattern.md)

   3. 行为型模式（Behavioral Pattern）
      1. [责任链模式](designpattern/behavioralpattern/ChainofResponsibilityPattern.md)
      2. [命令模式](designpattern/behavioralpattern/CommandPattern.md)
      3. [解释器模式](designpattern/behavioralpattern/InterpreterPattern.md)
      4. [迭代器模式](designpattern/behavioralpattern/IteratorPattern.md)
      5. [中介者模式](designpattern/behavioralpattern/MediatorPatter.md)
      6. [备忘录模式](designpattern/behavioralpattern/MementoPattern.md)
      7. [观察者模式](designpattern/behavioralpattern/ObserverPattern.md)
      8. [状态模式](designpattern/behavioralpattern/StatePattern.md)
      9. [策略模式](designpattern/behavioralpattern/StrategyPattern.md)
      10. [模版模式](designpattern/behavioralpattern/TemplatePattern.md)
      11. [访问者模式](designpattern/behavioralpattern/VisitorPattern)

#### Java新特性 JDK1.8

1. 

### 计算机基础

#### 计算机网络

1. [计算机网络知识](javabase/network/计算机网络知识点.md)

#### SHELL

1. [grep命令](linux/grep命令总结.md)

2. [find命令](linux/find命令总结.md)

3. [sed命令](linux/sed命令总结.md)

4. [awk命令](linux/awk命令总结.md)

5. [其他常用命令](linux/其他常用命令总结.md)

#### 常用算法

1. [二分查找算法](javaalgorithm/二分查找算法.md)

2. [冒泡排序算法](javaalgorithm/冒泡排序算法.md)

3. [插入排序算法](javaalgorithm/插入排序算法.md)

### MySQL数据库

1. [MySQL数据库索引总结](database/mysql/数据库索引学习总结.md)

2. [MySQL的可重复读](database/mysql/MySQL如何实现可重复读.md)

3. [MySQL的三大日志-binlog、redo log和undo log](database/mysql/MySQL的三大日志.md)

4. [MVCC](database/mysql/MVCC原理.md)

### 中间件

1. #### 缓存Redis

   1. [常用五大数据类型](redis/常用五大数据类型.md)
   2. [Redis配置文件介绍](redis/Redis配置文件介绍.md)
   3. [Redis分布式锁](redis/Redis分布式锁.md)
   4. [Redis常见面试题](redis/Redis常见面试题.md)
   5. [Redis安装mac](redis/Redis安装.md)

2. #### RPC

   1. [Dubbo](dubbo/Dubbo.md)
   2. [BIO、NIO和AIO](dubbo/BIO、NIO和AIO.md)

3. #### Zookeeper

   1. [zookeeper基础](zookeeper/zookeeper基础知识点.md)
   2. [zookeeper安装](zookeeper/zookeeper安装.md)

### 框架

1. Spring&Spring Boot
   1. [SpringMVC的执行过程](spring/SpringMVC.md)
   2. [Spring事务](spring/Spring事务学习.md)
   3. [SpringIoC](spring/Spring IoC学习总结.md)
   4. [SpringAOP](spring/SpringAOP学习总结.md)
   5. [Spring生命周期](spring/Spring生命周期.md)
   6. [Spring循环依赖](spring/Spring循环依赖.md)
   7. [Spring作用域](spring/Spring作用域.md)

2. Mybatis框架使用

   1. [代理模式](mybatis/代理设计模式.md)
   2. [Mybatis原理](mybatis/Mybatis原理.md)
   3. [Myatis入门](mybatis/Mybatis入门Demo.md)
   4. Mybatis的配置文件
      1. [全局配置文件](mybatis/Mybatis的全局配置文件.md)
      2. [Sql映射文件](mybatis/Sql映射文件.md)
      3. [动态sql](mybatis/动态sql.md)

   5. Mybatis-缓存机制
      1. [一级缓存](mybatis/一级缓存.md)
      2. [二级缓存](mybatis/二级缓存.md)

   6. [Mybatis-逆向工程](mybatis/Mybatis-逆向工程.md)

   7. [PageHelper插件进行分页](mybatis/PageHelper插件.md)

   8. [自定义TypeHandler处理枚举](mybatis/自定义TypeHandler处理枚举.md)

   9. [Mybatis学习代码库](https://github.com/sunwjblog/CodeRepository.git)

   10. [Mybatis常见面试题](mybatis/Mybatis常见面试题.md)

   11. [Mybatis遇到的问题](mybatis/Mybatis遇到的问题.md)

#### 开发工具的使用

1. [Git版本控制工具学习笔记](devtools/Git学习笔记.md)

2. Docker
   1. [Docker入门](docker/Docker入门.md)

### 读书笔记

1. [架构整洁之道-读书笔记](readbook/架构整洁之道-读书笔记.md)

2. [深入浅出的SpringBoot 2.x](readbook/深入浅出的SpringBoot 2.x.md)

### 简历

1. [简历模版](resume/简.md)

2. [面试问题梳理](resume/面试问题梳理.md)

