# Mybatis原理

#### Mybatis架构设计图

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/Mybatis架构图.webp)

* ##### 功能架构解释

  如图所示，整体架构分为3个层次，分别是API接口层、数据处理层、框架支撑层等。

  * **API接口层：**提供给外部使用的接口api，开发人员通过这些本地api来操作数据库。接口层接收到调用请求，就会调用数据处理层来完成具体的数据库操作。
  * **数据处理层：**负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。
  * **框架支撑层：**负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

#### 框架架构

##### 框架架构图

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/Mybatis框架架构图.webp)

###### 框架架构讲解：

这张图从上往下看。MyBatis的初始化，会从mybatis-config.xml配置文件，解析构造成Configuration这个类，就是图中的红框。

1. 加载配置：配置来源于两个地方，一处是配置文件，一处是Java代码的注解，将SQL的配置信息加载成为一个个MappedStatement对象（包括了传入参数映射配置、执行的SQL语句、结果映射配置），存储在内存中。

2. SQL解析：当API接口层接收到调用请求时，会接收到传入SQL的ID和传入对象（可以是Map、JavaBean或者基本数据类型），Mybatis会根据SQL的ID找到对应的MappedStatement，然后根据传入参数对象对MappedStatement进行解析，解析后可以得到最终要执行的SQL语句和参数。

3. SQL执行：将最终得到的SQL和参数拿到数据库进行执行，得到操作数据库的结果。

4. 结果映射：将操作数据库的结果按照映射的配置进行转换，可以转换成HashMap、JavaBean或者基本数据类型，并将最终结果返回。

#### Mybatis核心类

1. ##### SqlSessionFactoryBuilder

   1. 每一个MyBatis的应用程序的入口是SqlSessionFactoryBuilder。

   2. 它的作用是通过XML配置文件创建Configuration对象（当然也可以在程序中自行创建），然后通过build方法创建SqlSessionFactory对象。没有必要每次访问Mybatis就创建一次SqlSessionFactoryBuilder，通常的做法是创建一个全局的对象就可以了。

      ```java
       public SqlSessionFactory getSqlSessionFactory() throws IOException{
      
              String resource = "mybatis-config.xml";
              // 加载配置文件
              InputStream inputStream = Resources.getResourceAsStream(resource);
              // 构建SqlSeesionFactory实例
              return new SqlSessionFactoryBuilder().build(inputStream);
          }
      ```



​			 org.apache.ibatis.session.Configuration 是mybatis初始化的核心。

​			 mybatis-config.xml中的配置，最后会解析xml成Configuration这个类。

2. ##### SqlSessionFactory对象由SqlSessionFactoryBuilder创建

3. ##### SqlSession

   SqlSession对象的主要功能是完成一次数据库的访问和结果的映射，它类似于数据库的session概念，由于不是线程安全的，所以SqlSession对象的作用域需限制方法内。SqlSession的默认实现类是DefaultSqlSession，它有两个必须配置的属性：Configuration和Executor。

   SqlSession ：默认创建DefaultSqlSession 并且开启一级缓存，创建执行器 、赋值。

4. ##### Executor

   Executor对象在创建Configuration对象的时候创建，并且缓存在Configuration对象里。Executor对象的主要功能是调用StatementHandler访问数据库，并将查询结果存入缓存中（如果配置了缓存的话）。

5. ##### StatementHandler

   StatementHandler是真正访问数据库的地方，并调用ResultSetHandler处理查询结果。

6. ##### ResultSetHandler

   处理查询结果。

#### Mybatis成员层次与职责

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/Mybatis成员层次与职责图.webp)

1. SqlSession 作为MyBatis工作的主要顶层API，表示和数据库交互的会话，完成必要数据库增删改查功能
2. Executor MyBatis执行器，是MyBatis 调度的核心，负责SQL语句的生成和查询缓存的维护
3. StatementHandler 封装了JDBC Statement操作，负责对JDBCstatement的操作，如设置参数、将Statement结果集转换成List集合。
4. ParameterHandler 负责对用户传递的参数转换成JDBC Statement 所需要的参数
5. ResultSetHandler *负责将JDBC返回的ResultSet结果集对象转换成List类型的集合；
6. TypeHandler 负责java数据类型和jdbc数据类型之间的映射和转换
7. MappedStatement MappedStatement维护了一条<select|update|delete|insert>节点的封
8. SqlSource 负责根据用户传递的parameterObject，动态地生成SQL语句，将信息封装到BoundSql对象中，并返回
9. BoundSql 表示动态生成的SQL语句以及相应的参数信息
10. Configuration MyBatis所有的配置信息都维持在Configuration对象之中