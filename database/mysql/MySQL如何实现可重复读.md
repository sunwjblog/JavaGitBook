# MySQL的可重复读

## 什么是可重复读

可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。

可以理解为，在可重复读的隔离级别下，事务启动的时候就给当前读 “拍了个快照”，注意，这个快照时基于整个库的。

如果一个库有 100G，那么我启动一个事务，MySQL就要拷贝 100G 的数据出来，这个过程得多慢啊。可是，我平时的事务执行起来很快啊。

实际上，我们并不需要拷贝出这 100G 的数据。我们来看下”快照“是怎么实现的。

## 快照

InnoDB 里面每个事务都有一个唯一的事务 ID，叫作**transaction id**。它在事务开始的时候向 InnoDB 的事务系统申请的，**是按申请顺序严格递增的**。

##### 每条记录在更新的时候都会同时记录一条 undo log，这条 log 就会记录上当前事务的 transaction id，记为 row trx_id。记录上的最新值，通过回滚操作，都可以得到前一个状态的值。

如下图所示，一行记录被多个事务更新之后，最新值为 k=22。假设事务A在 trx_id=15 这个事务**提交后启动**，事务A 要读取该行时，就通过 undo log，计算出该事务启动瞬间该行的值为 k=10。

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/可重复读实现原理.png)

在可重复读隔离级别下，一个事务在启动时，InnoDB 会为事务构造一个数组，用来保存这个事务启动瞬间，当前正在”活跃“的所有事务ID。”活跃“指的是，启动了但还没提交。

##### 数组里面事务 ID 为最小值记为低水位，当前系统里面已经创建过的事务 ID 的最大值加 1 记为高水位。

这个视图数组和高水位，就组成了**当前事务的一致性视图（read-view）**。

这个视图数组把所有的 row trx_id 分成了几种不同的情况。

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/可重复读原理2.png)

如果 trx_id 小于低水位，表示这个版本在事务启动前已经提交，可见；

如果 trx_id 大于高水为，表示这个版本在事务启动后生成，不可见；

如果 trx_id 大于低水位，小于高水位，分为两种情况：

1. 若 trx_id 在数组中，表示这个版本在事务启动时还未提交，不可见；
2. 若 trx_id 不在数组中，表示这个版本在事务启动时已经提交，可见。

## 例子

初始化语句

```mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `k` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
insert into t(id, k) values(1,1),(2,2);
```

下表为事务A, B, C 的执行流程

| 事务A                                       | 事务B                                       | 事务C                          |
| ------------------------------------------- | ------------------------------------------- | ------------------------------ |
| START TRANSACTION WITH CONSISTENT SNAPSHOT; |                                             |                                |
|                                             | START TRANSACTION WITH CONSISTENT SNAPSHOT; |                                |
|                                             |                                             | UPDATE t SET k=k+1 WHERE id=1; |
|                                             | UPDATE t SET k=k+1 WHERE id=1;              |                                |
|                                             | SELECT k FROM t WHERE id=1;                 |                                |
| SELECT k FROM t WHERE id=1;                 |                                             |                                |
| COMMIT;                                     |                                             |                                |
|                                             | COMMIT;                                     |                                |

假设事务A, B, C 的 trx_id 分别为 100, 101, 102。事务A开始前活跃的事务 ID 只有 99，并且 id=1 这一行数据的 trx_id=90。

根据假设，我们得出事务启动瞬间的视图数组：事务A：[99, 100]，事务B：[99, 100, 101]，事务C：[99, 100, 101, 102]。

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/可重复读的原理3.png)

事务C通过更新语句，把 k 更新为 2，此时trx_id=102；

事务B通过更新语句，把 k 更新为 3，此时trx_id=101；

事务B通过查询语句，查询到最新一条记录为3，trx_id=101，满足隔离条件，返回 k=3；

事务A通过查询语句：

1. 查询到最新一条记录为3，trx_id=101，比高水位大，不可见；
2. 通过 undo log，找到上一个历史版本，trx_id=102，比高水位大，不可见；
3. 继续找上一个历史版本，trx_id=90，比低水位小，可见。

#### 为啥事务B更新的时候能看到事务C的修改？

假设事务B在更新的看不到事务C的修改，是什么个情况？

1. 事务B查询到最新一条记录为2，trx_id=102，比高水位大，不可见；
2. 通过 undo log，找到上一个版本，trx_id=90，比低水位小，可见；
3. 返回记录 k=1，执行 k=k+1，把 k 更新为2，此时 trx_id=101。

如果是这种情况，事务C可能就蒙了：“啥子情况，我的更新怎么就丢了”。事务B覆盖了事务C的更新。

所以，InnoDB在更新时运用一条规则：**更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读“ （current read）。**

因此，事务B在更新时要拿到最新的数据，在此基础上做更新。紧接着，事务B在读取的时候，查询到最新的记录为3， trx_id=101 为当前事务ID，可见。

我们再假设另一种情况：

事务B在更新之后，事务C紧接着更新，事务B回滚了，事务C成功提交。

| 事务B                                       | 事务C                                       |
| ------------------------------------------- | ------------------------------------------- |
| START TRANSACTION WITH CONSISTENT SNAPSHOT; |                                             |
|                                             | START TRANSACTION WITH CONSISTENT SNAPSHOT; |
| UPDATE t SET k=k+1 WHERE id=1;              |                                             |
|                                             | UPDATE t SET k=k+1 WHERE id=1;              |
|                                             | SELECT k FROM t WHERE id=1;                 |
| ROLLBACK;                                   |                                             |
|                                             | COMMIT;                                     |

如果按照当前读的定义，会发生以下事故，假设当前 K=1：

1. 事务B把 k 更新为 2；
2. 事务C读取到当前最新值，k=2，更新为3；
3. 事务B回滚；
4. 事务C提交。

这时候，事务C发现自己想要执行的是 +1 操作，结果变成了 ”+2“ 操作。

InnoDB 肯定不允许这种情况的发生，事务B在执行更新语句时，会给该行加上行锁，直到事务B结束，才会释放这个锁。

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/可重复读的实现原理4.png)

## 快照读和当前读

### 什么是快照读和当前读？

* 当前读：select …. lock in share mode，select …. for update 或 update、delete、insert 加了锁的增删改查的记录。读取的是最新版本，并且对读取的记录加锁，阻塞其他事务同时改动相同记录，避免出现安全问题。
* 快照读：不加锁的非阻塞读，单纯的select操作
  * Read Committed隔离级别：**每次select都生成一个快照读**
  * Read Repeatable隔离级别：**开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读**

出现快照读，是为了提高并发性能的考虑；快照读的实现是基于多版本并发控制及mvcc可以认为是行级锁的一个变种，但是它在很多情况下避免了加锁操作，因此开销更低，由于是基于多版本控制，所以快照读读到的数据有可能是数据的历史版本，非当前版本。

### 例子：

4个session，2个是rc隔离级别（session1，session2），2个是rr隔离级别（session3，session4）

​	session1开启事务，查询sql：select * from t where id = 2，得出预期结果

​	session2开启事务，更新sql：update t set balance = 600 where id=2，执行成功

​	session1中使用快照读和当前读分别读取数据

​			快照读：select * from t where id=2 ==>balance=600

​			当前读：select * from t where id=2 lock in share mode ==>balance=600

​	结论：在rc隔离级别下，快照读和当前读读的到数据版本是一样的。

​	session3和session4操作中，分别按照session1和session2的操作重新操作一遍。	

​		session3开启事务，查询sql：select * from t where id = 2，得出预期结果==>600

​		注意其中session4中的更新sql为：update t set balance = 300 where id=2

​		session3中使用快照读和当前读分别读取数据

​			快照读：select * from t where id=2 ==>balance=600

​			当前读：select * from t where id=2 lock in share mode ==>balance=300

​		结论：在rr隔离级别下，快照读有可能会读到数据的历史版本数据，未读到数据当前最新数据，当前读总是可以会获得数据的当前版本

​		另外一种情况：

​		先不在session3中查询数据，先在session4中更新数据，更新sql：update t set balance = 100 where id=2

​		然后回到session3中使用快照读和当前读分别读取数据

​			快照读：select * from t where id=2 ==>balance=100

​			当前读：select * from t where id=2 lock in share mode ==>balance=100

​	结论：在rr隔离级别下，事务首次调用快照读的地方很关键，创建快照的时机决定读取数据的版本。

## 小结

1. InnoDB 的行数据有多个版本，每个版本都有 row trx_id。
2. 事务根据 undo log 和 trx_id 构建出满足当前隔离级别的一致性视图。
3. 可重复读的核心是一致性读，而事务更新数据的时候，只能使用当前读，如果当前记录的行锁被其他事务占用，就需要进入锁等待。



## 参考

[MySQL是如何实现可重复读的?](https://juejin.cn/post/6844904180440629262)

