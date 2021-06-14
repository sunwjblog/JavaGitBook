# MVCC

| Session1                       | Session2                    |
| ------------------------------ | --------------------------- |
| Select c1 from t1;return c1=10 |                             |
| Start transaction;             |                             |
| update t1 set c1=20;           |                             |
|                                | Start transaction;          |
|                                | select c1 from t1;return ?  |
| Commit;                        |                             |
|                                | select c1 from t1;return ?; |

**分析：**

* 在事务隔离级别为READ-UNCOMMITTED的情况下，在session1提交前后，session2查询看到的都是修改后的结果a=20；
* 在事务隔离级别为READ-COMMITTED的情况下，在session1提交前看到的还是a=10；提交后看到的是a=20；
* 在事务隔离级别为REPEATABLE-READ、SERIALIZABLE的情况下，在session1提交前后，session2查询看到的都是修改前的结果a=10。



为什么修改数据后，数据库查询的结果还是之前的数据呢？数据库其实是借助于undo和MVCC来实现的。如图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/MVCC1.png)

当插入一条数据时，在记录上对应的回滚指针为null，如图所示：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/MVCC2.png)

当更新记录时，将原记录放入Undo表空间中，我们查询看到的未修改的数据就是从Undo表空间中返回的，如果存在多个数据的版本就会构成一个链表。

在MySQL中会根据记录上的回滚段指针及事务ID判断记录是否可见的。具体的逻辑如下：

1. 在每个事物开始时，都会将当前系统中所有的活跃事务（”活跃“指的是，启动了但还没提交）拷贝到一个列表（Read View）中。当读取一行记录时，会根据行记录上的TRX_ID值与Read View中的最大值TRX_ID值、最小值TRX_ID值的比较判断是否可见。
2. 比较TRX_ID值是否小于Read View中的最小TRX_ID值，如果是，则说明此事务早于Read View中的所有事务结束，可以输出返回；如果不是，则判断TRX_ID值是否大于Read View中的最大值TRX_ID值。
   1. 如果是，则根据行记录上的回滚段指针找到回滚段中的对应记录且取出TRX_ID值赋给当前行的TRX_ID，并重新执行比较操作（说明此行记录在事务开始之后发生了变化）。
   2. 如果不是，则判断TRX_ID值是否在Read View中。如果在Read View中，则根据行记录上的回滚段指针找到回滚段中的对应记录且取出TRX_ID值（说明此行记录在事务开始时处于活跃状态）；如果不是，则返回记录。

如图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/可重复读原理2.png)



tips：对于不同的事务隔离级别，可见性的实现也不一样

* 对于READ-COMMITTED隔离级别，事务内的每一条查询语句都会重新创建Read View，这样就会产生不可重复读的现象。
* 对于REPEATABLE-READ隔离级别，事务内第一条语句执行时会创建Read View，在事务结束这段时间内每一次查询都不会重新创建Read-View，从而实现了可重复读。
