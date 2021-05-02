# 全局配置文件

#### Configration配置

* ##### properties配置

* ##### setting设置

* ##### typeAliases类型命名

* ##### typeHandlers类型处理器

* ##### objectFactory对象工厂

* ##### plugins插件

* ##### environment环境

  * ##### environment环境变量

    * ##### transactionManager事务管理器

    * ##### dataSource数据源

* ##### databasesIdProvider数据库厂商标识

* ##### mapper映射器

#### Properties配置

​		配置数据源信息，将数据源相关的配置单独配置到一个properties文件中，又或者一些与项目相关的配置也可以配置到properties文件中。

​		拿配置数据源为例，将项目中用到的数据源信息，配置到一个properties文件中，如：

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=1992Sunwj
```

通过<properties>标签将properties文件引入到mybatis-config.xml文件中

```xml

<configuration>

	<!-- 引入配置文件 -->
	<properties resource="dbconfig.properties"></properties>

	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>
</configuration>
```

#### Seeting配置

setting配置是Mybatis中极为重要的配置，它的配置会改变Mybatis的运行时的行为。

| 设置参数                 | 描述                                                         | 有效值               | 默认值        |
| ------------------------ | ------------------------------------------------------------ | -------------------- | ------------- |
| cacheEnabled             | 所有映射器中配置的缓存的全局开关                             | true/false           | true          |
| lazyLoadingEnabled       | 延迟加载开关，开启时，所有关联对象都会延迟加载。             | true/false           | false         |
| useColumnLabel           | 使用列标签代替列名。不同的驱动在这方面会有不同的表现         | true/false           | true          |
| defaultStatementTimeout  | 设置超时时间，它决定驱动等待数据库响应的秒数。               | Any positive integer | Not Set(null) |
| mapUnderscoreToCamelCase | 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名A_COLUMN到经典Java属性名aColumn的类似映射 | true/false           | false         |

```xml
<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/>
	</settings>
```

#### typeAliases别名处理器

* 类型别名是为Java类型设置一个断的名字，可以方便我们引用某个类。

  ```xml
  <typeAliases>
  		<typeAlias type="com.sunwj.mybatis.bean.Employee" alias="empliyee"></typeAlias>
  	</typeAliases>
  ```

* 类很多的情况下，可以批量设置别名这个包下的每一个类创建一个默认的别名，就是简单类名小写

  ```xml
  <typeAliases>
  		<package name="com.sunwj.mybatis.bean"/>
  	</typeAliases>
  ```

* 也可以使用@Alias注解为其指定一个别名

  ```java
  @Alias("emp")
  public class Employee {...}
  ```

* Mybatis已经为许多常见的Java类型内建列相应的类型别名。它们都是大小写不敏感的，我们在起别名的时候千万不要占用已有的别名。

#### typeHandlers类型处理器

* 无论是Mybatis在预处理语句(PreparedStatement)中设置一个参数时，还是从结果集中取出一个值时，都会用类型处理器将获取的值以合适的方式转换成Java类型。

* 自定义类型处理
  * 实现org.apache.ibatis.type.TypeHandler接口或者集成org.apache.ibatis.type.BaseTypeHandler
  * 指定其映射某个JDBC类型
  * 在mybatis全局配置文件中注册

#### pulgins插件

* 插件是MyBatis提供的一个非常强大的机制，我们 可以通过插件来修改MyBatis的一些核心行为。插 件通过动态代理机制，可以介入四大对象的任何 一个方法的执行。
* Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
* ParameterHandler (getParameterObject, setParameters)
* ResultSetHandler (handleResultSets, handleOutputParameters)
* StatementHandler (prepare, parameterize, batch, update, query)

#### environments环境

* MyBatis可以配置多种环境，比如开发、测试和生 产环境需要有不同的配置。
* 每种环境使用一个environment标签进行配置并指 定唯一标识符
* 可以通过environments标签中的default属性指定 一个环境的标识符来快速的切换环境
* id:指定当前环境的唯一标识；
* transactionManager、和dataSource都必须有； 
* type: JDBC | MANAGED | 自定义 
  * JDBC:使用了 JDBC 的提交和回滚设置
  * MANAGED:不提交或回滚一个连接、让容器来管理 事务的整个生命周期
  * 自定义:实现TransactionFactory接口，type=全类名/ 别名 
* dataSource
  * type: UNPOOLED | POOLED | JNDI | 自定义
    * UNPOOLED:不使用连接池
    * POOLED:使用连接池
    * JNDI: 在EJB 或应用服务器这类容器中查找指定的数据源
    * 自定义:实现DataSourceFactory接口，定义数据源的 获取方式

```xml
<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>
```

#### databaseIdProvider环境

* MyBatis 可以根据不同的数据库厂商执行不同的语句

  ```xml
  <databaseIdProvider type="DB_VENDOR">
  		<!-- 为不同的数据库厂商起别名 -->
  		<property name="MySQL" value="mysql"/>
  		<property name="Oracle" value="oracle"/>
  		<property name="SQL Server" value="sqlserver"/>
  	</databaseIdProvider>
  ```

  ```xml
  # databaseId属性指向某个数据库
  <select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee">
  		select * from tbl_employee where id = #{id}
  	</select>
  	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee"
  		databaseId="mysql">
  		select * from tbl_employee where id = #{id}
  	</select>
  	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee"
  		databaseId="oracle">
  		select EMPLOYEE_ID id,LAST_NAME	lastName,EMAIL email 
  		from employees where EMPLOYEE_ID=#{id}
  ```

#### Mapper映射

* mapper逐个注册SQL映射文件

  ```xml
   <mappers>
   	<mapper resource="mybatis/mapper/PersonDao.xml"/>
   	<mapper resource="file///D:/UserDao.xml"/>
   	<mapper resource="com.sunwj.mybatis.mapper.PersonDaoAnnotation"/>
   </mappers>
  ```

* 或者使用批量注册

  ```xml
   <mappers>
   	<package name="com.atguigu.mybatis.dao"/>
   </mappers>
  ```



#### 完整的配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!--
		1、mybatis可以使用properties来引入外部properties配置文件的内容；
		resource：引入类路径下的资源
		url：引入网络路径或者磁盘路径下的资源
	  -->
	<properties resource="dbconfig.properties"></properties>
	
	
	<!-- 
		2、settings包含很多重要的设置项
		setting:用来设置每一个设置项
			name：设置项名
			value：设置项取值
	 -->
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/>
	</settings>
	
	
	<!-- 3、typeAliases：别名处理器：可以为我们的java类型起别名 
			别名不区分大小写
	-->
	<typeAliases>
		<!-- 1、typeAlias:为某个java类型起别名
				type:指定要起别名的类型全类名;默认别名就是类名小写；employee
				alias:指定新的别名
		 -->
		<!-- <typeAlias type="com.atguigu.mybatis.bean.Employee" alias="emp"/> -->
		
		<!-- 2、package:为某个包下的所有类批量起别名 
				name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写），）
		-->
		<package name="com.atguigu.mybatis.bean"/>
		
		<!-- 3、批量起别名的情况下，使用@Alias注解为某个类型指定新的别名 -->
	</typeAliases>
		
	<!-- 
		4、environments：环境们，mybatis可以配置多种环境 ,default指定使用某种环境。可以达到快速切换环境。
			environment：配置一个具体的环境信息；必须有两个标签；id代表当前环境的唯一标识
				transactionManager：事务管理器；
					type：事务管理器的类型;JDBC(JdbcTransactionFactory)|MANAGED(ManagedTransactionFactory)
						自定义事务管理器：实现TransactionFactory接口.type指定为全类名
				
				dataSource：数据源;
					type:数据源类型;UNPOOLED(UnpooledDataSourceFactory)
								|POOLED(PooledDataSourceFactory)
								|JNDI(JndiDataSourceFactory)
					自定义数据源：实现DataSourceFactory接口，type是全类名
		 -->
		 

		 
	<environments default="dev_mysql">
		<environment id="dev_mysql">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	
		<environment id="dev_oracle">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${orcl.driver}" />
				<property name="url" value="${orcl.url}" />
				<property name="username" value="${orcl.username}" />
				<property name="password" value="${orcl.password}" />
			</dataSource>
		</environment>
	</environments>
	
	
	<!-- 5、databaseIdProvider：支持多数据库厂商的；
		 type="DB_VENDOR"：VendorDatabaseIdProvider
		 	作用就是得到数据库厂商的标识(驱动getDatabaseProductName())，mybatis就能根据数据库厂商标识来执行不同的sql;
		 	MySQL，Oracle，SQL Server,xxxx
	  -->
	<databaseIdProvider type="DB_VENDOR">
		<!-- 为不同的数据库厂商起别名 -->
		<property name="MySQL" value="mysql"/>
		<property name="Oracle" value="oracle"/>
		<property name="SQL Server" value="sqlserver"/>
	</databaseIdProvider>
	
	
	<!-- 将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
	<!-- 6、mappers：将sql映射注册到全局配置中 -->
	<mappers>
		<!-- 
			mapper:注册一个sql映射 
				注册配置文件
				resource：引用类路径下的sql映射文件
					mybatis/mapper/EmployeeMapper.xml
				url：引用网路路径或者磁盘路径下的sql映射文件
					file:///var/mappers/AuthorMapper.xml
					
				注册接口
				class：引用（注册）接口，
					1、有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下；
					2、没有sql映射文件，所有的sql都是利用注解写在接口上;
					推荐：
						比较重要的，复杂的Dao接口我们来写sql映射文件
						不重要，简单的Dao接口为了开发快速可以使用注解；
		-->
		<!-- <mapper resource="mybatis/mapper/EmployeeMapper.xml"/> -->
		<!-- <mapper class="com.atguigu.mybatis.dao.EmployeeMapperAnnotation"/> -->
		
		<!-- 批量注册： -->
		<package name="com.atguigu.mybatis.dao"/>
	</mappers>
</configuration>
```



**Mapper映射文件**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.atguigu.mybatis.dao.EmployeeMapper">
<!-- 
namespace:名称空间;指定为接口的全类名
id：唯一标识
resultType：返回值类型
#{id}：从传递过来的参数中取出id值

public Employee getEmpById(Integer id);
 -->
 	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee">
		select * from tbl_employee where id = #{id}
	</select>
	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee"
		databaseId="mysql">
		select * from tbl_employee where id = #{id}
	</select>
	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee"
		databaseId="oracle">
		select EMPLOYEE_ID id,LAST_NAME	lastName,EMAIL email 
		from employees where EMPLOYEE_ID=#{id}
	</select>
</mapper>
```



**Properties配置文件**

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
jdbc.username=root
jdbc.password=1992Sunwj

orcl.driver=oracle.jdbc.OracleDriver
orcl.url=jdbc:oracle:thin:@localhost:1521:orcl
orcl.username=scott
orcl.password=123456
```

