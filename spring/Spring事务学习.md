# Spring事务

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/Spring-tx.png)

# Spring事务的分类

### 编程式事务管理

通过 `TransactionTemplate`或者`TransactionManager`手动管理事务，实际应用中很少使用，但是对于你理解 Spring 事务管理原理有帮助。

### 声明式事务管理

推荐使用（代码侵入性最小），实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）。

# 事务属性

## 1.传播特性

1. PROPAGATION_REQUIRED 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务
2. PROPAGATION_REQUIRED_NEW 表示当前方法必须运行在它自己的事务中。不管当前事务是否存在，它都会启动一个新的事务。如果存在当前事务，在该方法执行期间，当前事务会被挂起。
3. PROPAGATION_SUPPORTS 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会加入到当前事务中运行
4. PROPAGATION_NOT_SUPPORTED 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。
5. PROPAGATION_MANDATORY 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常
6. PROPAGATION_NESTED 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当事务进行单独地提交或回滚。如果内层事务抛出异常，则外层事务也会回滚。如果外层事务异常，内层事务已经执行提交的话不会再回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。
7. PROPAGATION_NEVER 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常

## 2.隔离级别

1. ISOLATION_DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别。
2. ISOLATION_READ_UNCOMMITTED 这是事务最低的隔离级别，它充许令外一个事务可以看到这个事务未提交的数据。
3. ISOLATION_READ_COMMITTED 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。
4. ISOLATION_REPEATABLE_READ 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻像读（mysql的innodb存储引擎在这个隔离级别已经解决了幻读的问题）。
5. ISOLATION_SERIALIZABLE 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻像读。

## 3.是否只读

1. readOnly默认为false，当设置为true时，那么JDBC驱动程序和数据库就有可能根据这种情况对该事务进行一些特定的优化，比方说不安排相应的数据库锁，以减轻事务对数据库的压力。
2. 当readOnly为true时，只能进行查询操作，不能增加、修改、删除数据。

## 4.事务超时

1. 事务不能运行太长的时间，合理适当的设置超时时间事务可回滚。
2. 事务的超时时间是指事务开始后到最后一个sql执行完成的时间，再最后一个sql执行前超时则事务回滚，如果再最后一个sql执行后超时了这时候事务不再回滚。

## 5.事务回滚

1. 默认情况下，事务只有遇到运行时异常（RuntimeException）时才会回滚。
2. 一般检查时异常或者被try...catch住的时候不会回滚。
3. 可以申明事务在遇到特定的检查型异常时回滚，需要配置相应的异常类型，使用rollbackFor或者rollbackForClassName。
4. 也可以申明事务遇到特定的异常不回滚，使用noRollbackForClassName。

---

## 常见的面试问题

### 同一个类内

1、一个Service类里的两个方法，分别是A方法，B方法，都有打注解@Transactional。问当调用A方法时，A方法中先调用B方法，B方法执行成功后往下执行，但A方法抛异常了，此时A方法和B方法会发生什么情况？

两个方法都会回滚，不会执行成功，因为Spring事务在同一个Service下，不会新创建事务，会把两个事务并为一个事务处理，所以A方法回滚，同时B方法也会回滚。

2、同类场景，若A方法未打注解@Transactional，B方法有打注解，同样的执行方式，会发生什么情况？

A方法没有事务，会新启动一个事务，然而B又是在一个事务中运行的，所以A B方法是互不影响的，所以B会执行成功，但是A发生异常会回滚。

3、同类场景，若A方法打注解@Transactional，B方法未打注解，同样的执行方式，会发生什么情况？

A方法存在事务，所以该方法要在当前事务下运行，A方法发生异常回滚，所以B也会跟着回滚

4、同类场景，若A方法未打注解，B方法未打注解，同样的执行方式，会发生什么情况？

A方法会新启动一个事务，B方法也会新启动一个事务，所以B会成功，A会回滚

### 不同一个类内

#### 代码

```java
Class A {
    @Transactional(propagation=propagation.PROPAGATION_REQUIRED)
    public void aMethod {
        //do something
        B b = new B();
        b.bMethod();
    }
}

Class B {
    @Transactional(propagation=propagation.PROPAGATION_REQUIRED)
    public void bMethod {
       //do something
    }
}
```

1、根据PROPAGATION_REQUIRED特性是，如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

解释：

1. 如果外部方法没有开启事务的话，`Propagation.REQUIRED`修饰的内部方法会新开启自己的事务，且开启的事务相互独立，互不干扰。
2. 如果外部方法开启事务并且被`Propagation.REQUIRED`的话，所有`Propagation.REQUIRED`修饰的内部方法和外部方法均属于同一事务 ，只要一个方法回滚，整个事务均回滚。

所以以上测试场景，`aMethod()`和`bMethod()`使用的都是`PROPAGATION_REQUIRED`传播行为的话，两者使用的就是同一个事务，只要其中一个方法回滚，整个事务均回滚。

2、如果我们上面的`bMethod()`使用`PROPAGATION_REQUIRES_NEW`事务传播行为修饰，`aMethod`还是用`PROPAGATION_REQUIRED`修饰的话，这样调用会出现什么情况？

答：如果`aMethod()`发生异常回滚，`bMethod()`不会跟着回滚，因为 `bMethod()`开启了独立的事务；

但是，如果 `bMethod()`抛出了未被捕获的异常并且这个异常满足事务回滚规则的话,`aMethod()`同样也会回滚，因为这个异常被 `aMethod()`的事务管理机制检测到了。

为什么是这种情况？

因为`PROPAGATION_REQUIRED_NEW `表示当前方法必须运行在它自己的事务中。不管当前事务是否存在，它都会启动一个新的事务相互独立，互不干扰。如果存在当前事务，在该方法执行期间，当前事务会被挂起。

3、如果我们下面的`bMethod()，bMethod2()`使用`PROPAGATION_NESTED`事务传播行为修饰，`aMethod`还是用`PROPAGATION_REQUIRED`修饰的话，这样调用会出现什么情况？

如代码

```java
Class A {
    @Transactional(propagation=propagation.PROPAGATION_REQUIRED)
    public void aMethod {
        //do something
        B b = new B();
        b.bMethod();
        b.bMethod2();
    }
}

Class B {
    @Transactional(propagation=propagation.PROPAGATION_NESTED)
    public void bMethod {
       //do something
    }
    @Transactional(propagation=propagation.PROPAGATION_NESTED)
    public void bMethod2 {
       //do something
    }
}
```

答：如果 `aMethod()` 回滚的话，`bMethod()`和`bMethod2()`都要回滚，而`bMethod()`回滚的话，并不会造成 `aMethod()` 和`bMethod()2`回滚。

为什么会出现这种情况？

因为`PROPAGATION_NESTED`传播特性是如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于`TransactionDefinition.PROPAGATION_REQUIRED`。

解释：

1. 在外部方法未开启事务的情况下`Propagation.NESTED`和`Propagation.REQUIRED`作用相同，修饰的内部方法都会新开启自己的事务，且开启的事务相互独立，互不干扰。
2. 如果外部方法开启事务的话，`Propagation.NESTED`修饰的内部方法属于外部事务的子事务，外部主事务回滚的话，子事务也会回滚，而内部子事务可以单独回滚而不影响外部主事务和其他子事务。

注意：

```java
// 同一个Service类中，spring并不重新创建新事务，如果是两不同的Service，就会创建新事务了。
// 跨Service调用方法时，都会经过org.springframework.aop.framework.CglibAopProxy.DynamicAdvisedInterceptor.intercept()方法，只有经过此处，才能对事务进行控制。
```

## 数据库查询事务

MySQL InnoDB 存储引擎的默认支持的隔离级别是 **REPEATABLE-READ（可重读）**。我们可以通过`SELECT @@tx_isolation;`命令来查看，MySQL 8.0 该命令改为`SELECT @@transaction_isolation;`

```mysql
mysql> SELECT @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
```

### @Transactional 注解使用详解

#### `@Transactional` 的作用范围

1. **方法** ：推荐将注解使用于方法上，不过需要注意的是：**该注解只能应用到 public 方法上，否则不生效。**
2. **类** ：如果这个注解使用在类上的话，表明该注解对该类中所有的 public 方法都生效。
3. **接口** ：不推荐在接口上使用。

#### `@Transactional` 的常用配置参数

**`@Transactional` 的常用配置参数总结（只列巨额 5 个我平时比较常用的）：**

| 属性名      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| propagation | 事务的传播行为，默认值为 REQUIRED，可选的值在上面介绍过      |
| isolation   | 事务的隔离级别，默认值采用 DEFAULT，可选的值在上面介绍过     |
| timeout     | 事务的超时时间，默认值为-1（不会超时）。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| readOnly    | 指定事务是否为只读事务，默认值为 false。                     |
| rollbackFor | 用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型。 |

#### `@Transactional` 事务注解原理

**`@Transactional` 的工作机制是基于 AOP 实现的，AOP 又是使用动态代理实现的。如果目标对象实现了接口，默认情况下会采用 JDK 的动态代理，如果目标对象没有实现了接口,会使用 CGLIB 动态代理。**

`createAopProxy()` 方法 决定了是使用 JDK 还是 Cglib 来做动态代理，源码如下：

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {

	@Override
	public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
		if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
			Class<?> targetClass = config.getTargetClass();
			if (targetClass == null) {
				throw new AopConfigException("TargetSource cannot determine target class: " +
						"Either an interface or a target is required for proxy creation.");
			}
			if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
				return new JdkDynamicAopProxy(config);
			}
			return new ObjenesisCglibAopProxy(config);
		}
		else {
			return new JdkDynamicAopProxy(config);
		}
	}
  .......
}
```

如果一个类或者一个类中的 public 方法上被标注`@Transactional` 注解的话，Spring 容器就会在启动的时候为其创建一个代理类，在调用被`@Transactional` 注解的 public 方法的时候，实际调用的是，`TransactionInterceptor` 类中的 `invoke()`方法。这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务。

`TransactionInterceptor` 类中的 `invoke()`方法内部实际调用的是 `TransactionAspectSupport` 类的 `invokeWithinTransaction()`方法。

## 参考

参考代码： [Spring-tx-demo](https://github.com/star9500/spring-tx-demo)

参考文章：[Spring事务总结](https://github.com/Snailclimb/JavaGuide/blob/master/docs/system-design/framework/spring/Spring%E4%BA%8B%E5%8A%A1%E6%80%BB%E7%BB%93.md)

