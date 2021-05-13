# Mybatis常见面试题

#### 1、Hibernate和Mybatis的区别

* ##### Common

  * 都是对jdbc的封装，都是持久层的框架，都用于dao层的开发。

* ##### Diff

  * Mybatis是一个半自动化映射的框架，配置Java对象与sql语句执行结果的对应关系，多表关联关系配置简单。
  * Hibernate是一个圈表映射的框架，配置java对象与数据库表的对应关系，多表关联关系配置复杂。
  * Hibernate对SQL语句进行了封装，提供了日志、缓存、级联（级联比Mybatis强大）等特性，此外还提供了HQL操作数据库，数据库无关性支持好，但会多消耗性能。如果项目需要支持多种数据库，代码开发量少，但SQL语句优化困难。
  * Mybatis需要手动编写SQL，支持动态SQL、处理列表、动态生成表名、支持存储过程。开发工作量相对大一些，直接使用SQL语句操作数据库，不支持数据库无关性，但sql语句优化容易。

#### 2、Mapper编写有哪几种方式？

* ##### 第一种：接口实现类继承 SqlSessionDaoSupport：使用此种方法需要编写mapper 接口，mapper 接口实现类、mapper.xml 文件

* ##### 第二种：使用 org.mybatis.spring.mapper.MapperFactoryBean

* ##### 第三种：使用 mapper 扫描器

#### 3、Mybatis是否可以映射Enum枚举类？

当然可以，不仅可以映射枚举类，Mybatis可以映射任何对象到列表的一列上。

映射方式为自定义一个 TypeHandler，实现 TypeHandler 的 setParameter()和

getResult()接口方法。

TypeHandler 有两个作用，一是完成从 javaType 至 jdbcType 的转换，

二是完成 jdbcType 至 javaType 的转换，体现为 setParameter()和 getResult()两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果。

#### 4、Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？

1. Mybatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association指的就是一对一，collection 指的就是一对多查询。在 Mybatis 配置文件中，可以配置是否启用延迟加载 lazyLoadingEnabled=true|false。
2. 它的原理是，使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 a.getB().getName()，拦截器 invoke()方法发现 a.getB()是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName()方法的调用。这就是延迟加载的基本原理。

#### 5、#{}和${}的区别是什么？

\#{}是预编译处理，${}是字符串替换。 

Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值； 

Mybatis在处理${}时，就是把${}替换成变量的值。 

使用#{}可以有效的防止SQL注入，提高系统安全性。

#### 6、Mybatis的一级、二级缓存

* 一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就将清空，默认打开一级缓存。

* 二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap 存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现Serializable序列化接口(可用来保存对象的状态),可在它的映射文件中配置`<cache/>`

* 对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。

#### 7、Mybatis常用注解

[@Insert ](https://github.com/souyunku/DevBooks/blob/master/Insert)： 插入sql , 和xml insert sql语法完全一样

[@Select ](https://github.com/souyunku/DevBooks/blob/master/Select)： 查询sql, 和xml select sql语法完全一样

[@Update ](https://github.com/souyunku/DevBooks/blob/master/Update)： 更新sql, 和xml update sql语法完全一样

[@Delete ](https://github.com/souyunku/DevBooks/blob/master/Delete)： 删除sql, 和xml delete sql语法完全一样

[@Param ](https://github.com/souyunku/DevBooks/blob/master/Param)： 入参

[@Results ](https://github.com/souyunku/DevBooks/blob/master/Results)：结果集合

[@Result ](https://github.com/souyunku/DevBooks/blob/master/Result)： 结果

#### 8、Mybatis是如何进行分页的？分页插件的原理是什么？

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而不是物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，田间对应的物理分页语句和物理分页参数。

例如：select cloumn1 ... from Job,拦截sql后重写为：select t.column... from （select \* from Job）t limit 0，10

#### 9、通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应，请问，这个 Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

Dao 接口，就是人们常说的 `Mapper`接口，接口的全限名，就是映射文件中的 namespace 的值，接口的方法名，就是映射文件中`MappedStatement`的 id 值，接口方法内的参数，就是传递给 sql 的参数。`Mapper`接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个`MappedStatement`，举例：`com.mybatis3.mappers.StudentDao.findStudentById`，可以唯一找到 namespace 为`com.mybatis3.mappers.StudentDao`下面`id = findStudentById`的`MappedStatement`。在 MyBatis 中，每一个`<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个`MappedStatement`对象。

Dao接口里的方法可以重载，但是Mybatis的XML里面的ID不能重复。

**Mybatis 的 Dao 接口可以有多个重载方法，但是多个接口对应的映射必须只有一个，否则启动会报错。**

#### 10、简述MyBatis的插件运行原理，以及如何编写一个插件

MyBatis 仅可以编写针对 `ParameterHandler`、`ResultSetHandler`、`StatementHandler`、`Executor` 这 4 种接口的插件，MyBatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 `InvocationHandler` 的 `invoke()`方法，当然，只会拦截那些你指定需要拦截的方法。

实现 MyBatis 的 Interceptor 接口并复写` intercept()`方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

#### 11、MyBatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？

MyBatis 动态 sql 可以让我们在 Xml 映射文件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能，MyBatis 提供了 9 种动态 sql 标签 `trim|where|set|foreach|if|choose|when|otherwise|bind`。

其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

#### 参考：

* [Mybatis常见面试题](https://github.com/Snailclimb/JavaGuide/blob/master/docs/system-design/framework/mybatis/mybatis-interview.md)