# Sql映射文件

#### 映射文件内部元素

映射文件指导着Mybatis如何进行数据库增删改查，有很重要的意义。

* cache 命名空间的二级缓存配置
* cache-ref 其他命名空间缓存配置的引用
* resultMap 自定义结果集映射
* parameterMap 已废弃，老式风格的参数映射
* sql 抽取可重用语句块
* insert 映射插入语句
* update 映射更新语句
* delete 映射删除语句
* select 映射查询语句

#### insert、update、delete元素

标签属性语义

| Id              | 命名空间中的唯一标识符                                       |
| --------------- | ------------------------------------------------------------ |
| parameterType   | 将要传入语句的参数的完全限定类名或别名，这个属性是可选的，因为Mybatis可以通过TypeHandler推断出具体传入语句的参数类型，默认值为unset。 |
| flushCache      | 将其设置为true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句） |
| timeout         | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数，默认值为unset（依赖驱动） |
| statementType   | STATEMENT，PREPARED或CALLABLE的一个，这会让Mybatis分别使用Statement，PrepareStatement或CallableStatement，默认值：PREPARED |
| useGenerateKeys | (仅对insert和update有用)这会令Mybatis使用JDBC的getGeneratedKeys方法来取出由数据库内部生成的主键（比如：像MySQL和SQL Server这样的关系数据库管理系统的自动递增字段），默认值：false |
| keyColumn       | （仅对insert和update有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置，如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| KeyProperty     | （仅对insert和update有用）唯一标记一个属性，Mybatis会通过getGenerateKeys的返回值或者通过insert语句的selectKey子元素设置它的键值，默认：unset |
| databaseId      | 如果配置了databaseIdProvider，Mybatis会加载所有的不带databaseId或匹配当前databaseId的语句，如果带或者不带的语句都有，则不带的会被忽略。 |

#### 主键生成方式

* 若数据库支持自动生成主键的字段(比如MySQL 和 SQL Server)，则可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上。

  ```xml
  <!-- public void addEmp(Employee employee); -->
  	<!-- parameterType：参数类型，可以省略， 
  	获取自增主键的值：
  		mysql支持自增主键，自增主键值的获取，mybatis也是利用statement.getGenreatedKeys()；
  		useGeneratedKeys="true"；使用自增主键获取主键值策略
  		keyProperty；指定对应的主键属性，也就是mybatis获取到主键值以后，将这个值封装给javaBean的哪个属性
  	-->
  	<insert id="addEmp" parameterType="com.atguigu.mybatis.bean.Employee"
  		useGeneratedKeys="true" keyProperty="id" databaseId="mysql">
  		insert into tbl_employee(last_name,email,gender) 
  		values(#{lastName},#{email},#{gender})
  	</insert>
  ```

* 而对于不支持自增型主键的数据库(例如 Oracle)，则可以使用 selectKey 子元素: selectKey 元素将会首先运行，id 会被设置，然 后插入语句会被调用

  ```xml
  <insert id="addEmp" databaseId="oracle">
  		<!-- 
  		keyProperty:查出的主键值封装给javaBean的哪个属性
  		order="BEFORE":当前sql在插入sql之前运行
  			   AFTER：当前sql在插入sql之后运行
  		resultType:查出的数据的返回值类型
  		
  		BEFORE运行顺序：
  			先运行selectKey查询id的sql；查出id值封装给javaBean的id属性
  			在运行插入的sql；就可以取出id属性对应的值
  		AFTER运行顺序：
  			先运行插入的sql（从序列中取出新值作为id）；
  			再运行selectKey查询id的sql；
  		 -->
  		<selectKey keyProperty="id" order="BEFORE" resultType="Integer">
  			<!-- 编写查询主键的sql语句 -->
  			<!-- BEFORE-->
  			select EMPLOYEES_SEQ.nextval from dual 
  			<!-- AFTER：
  			 select EMPLOYEES_SEQ.currval from dual -->
  		</selectKey>
  		
  		<!-- 插入时的主键是从序列中拿到的 -->
  		<!-- BEFORE:-->
  		insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
  		values(#{id},#{lastName},#{email<!-- ,jdbcType=NULL -->}) 
  		<!-- AFTER：
  		insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
  		values(employees_seq.nextval,#{lastName},#{email}) -->
  	</insert>
  ```

* ##### selectKey

  | keyProperty   | selectKey语句结果应该被设置的目标属性                        |
  | ------------- | ------------------------------------------------------------ |
  | keyColumn     | 匹配属性的返回结果集中的列名称                               |
  | resultType    | 结果的类型，Mybatis通常可以推算出来，但是为了更加确定写上也不会有什么问题。Mybatis允许任何简单类型用作主键的类型，包括字符串 |
  | order         | 可以被设置为BEFORE或AFTER。如果设置为BEFORE，那么它会首先选择主键，设置keyProperty然后执行插入语句。如果设置为AFTER，那么先执行插入语句，然后selectKey元素 |
  | statementType | 与前面相同，Mybatis支持STATEMENT，PREPARED和CALLABLE语句的映射类型，分别代表PreparedStatement和CallableStatement类型 |

#### 参数传递与处理

* ##### 参数传递

  * 单个参数
  * 多个参数
  * 命名参数
  * POJO
  * Map

* ##### 参数处理

  * 参数可以指定一个特殊的数据类型

    *  javaType 通常可以从参数对象中来去确定
    *  如果 null 被当作值来传递，对于所有可能为空的列， jdbcType 需要被设置
    *  对于数值类型，还可以设置小数点后保留的位数:
    *  mode 属性允许指定 IN，OUT 或 INOUT 参数。如果参数 为 OUT 或 INOUT，参数对象属性的真实值将会被改变， 就像在获取输出参数时所期望的那样。

  * 参数位置支持的属性

    javaType、jdbcType、mode、numericScale、resultMap、typeHandler、jdbcTypeName

  * 实际上通常被设置的是：可能为空的列指定jdbcType
  * \#{key}:获取参数的值，预编译到SQL中。安全。
  * ${key}:获取参数的值，拼接到SQL中。有SQL注入问 题。ORDER BY ${name}

#### select元素

* Select元素来定义查询操作
* Id：唯一标识符
  * 用来引用这条语句，需要和接口的方法名一致
* parameterType：参数类型。
  * 可以不传，Mybatis会根据TypeHandler自动推断
* resultType：返回值类型
  * 别名或者全类名，如果返回的是集合，定义集合中元素的类型。不能和resultMap同时使用。
* 其他属性

| parameterType | 将会传入这条语句的参数类的完全限定名或别名。这个属性是可选的，因为Mybatis可以通过TypeHandler推断出具体传入语句的参数，默认值为unset |
| ------------- | ------------------------------------------------------------ |
| resultType    | 从这条语句中返回的期望类型的类的完全限定名或别名，注意如果是集合，那应该是结婚可以包含的类型，而不能是集合本身，该属性和resultMap不能同时使用 |
| resultMap     | 外部resultMap的命名引用，和resultType属性不能同时使用。      |
| flushCache    | 将其设置为true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false |
| useCache      | 将其设置为true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true |
| timeout       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数，默认值为unset（依赖驱动） |
| fetchSize     | 影响驱动程序每次批量返回的结果行数，默认值为unset（依赖驱动） |
| statementType | STATEMENT，PREPARED或CALLABLE的一个，这会让Mybatis分别使用Statement，PrepareStatement或CallableStatement，默认值：PREPARED |
| resultSetType | FORWARD_INLY,SCROLL_SENSITIVE 或SCROLL_INSENITIVE中的一个，默认值为unset（依赖驱动） |
| databaseId    | 如果配置了databaseIdProvider，Mybatis会加载所有的不带databaseId或匹配当前databaseId的语句，如果带或者不带的语句都有，则不带的会被忽略。 |
| resultOrdered | 这个设置仅针对嵌套结果select语句适用；如果为true，就假设包含了嵌套结果集或是分组，这样当返回一个主结果行，就不会发生又对前面结果集引用的情况，这就使得在获取嵌套的结果集的时候不至于导致内存不够用，默认值：false |
| resultSets    | 这个设置仅对结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称逗号分隔的。 |

#### resultMap

* constructor - 类在实例化时, 用来注入结果到构造方法中

  * idArg - ID 参数; 标记结果作为 ID 可以帮助提高整体效能
  * arg - 注入到构造方法的一个普通结果

* id – 一个 ID 结果; 标记结果作为 ID 可以帮助提高整体效能

* result – 注入到字段或 JavaBean 属性的普通结果

* association – 一个复杂的类型关联;许多结果将包成这种类型

  * 嵌入结果映射 – 结果映射自身的关联,或者参考一个

* collection – 复杂类型的集

  * 嵌入结果映射 – 结果映射自身的集,或者参考一个

* discriminator – 使用结果值来决定使用哪个结果映射

  * case – 基于某些值的结果映射
    * 嵌入结果映射–这种情形结果也映射它本身,因此可以包含很多相同的元 素,或者它可以参照一个外部的结果映射。

* ##### id&result

  * id和result映射一个单独列的值到简单数据类型 (字符串,整型,双精度浮点数,日期等)的属性或字段。

| Property    | 映射到列结果的字段或属性，例如：“username” 或 “address.street.number”。 |
| ----------- | ------------------------------------------------------------ |
| column      | 数据表的列名。通常和resultSet.getString(columnName)的返回值一致 |
| javaType    | 一个Java类的完全限定名，或一个类型别名。如果映射到一个JavaBean，Mybatis通常可以断定类型 |
| jdbcType    | JDBC类型是仅仅需要对插入，更新和删除操作可能为空的列进行处理 |
| typeHandler | 类型处理器，使用这个属性，可以覆盖默认的类型处理器。这个属性值是类的完全限定名或者是一个类型处理器的实现，或者是类名别名。 |

* ##### association

  * 复杂对象映射

  * POJO中的属性可能会是一个对象

  * 我们可以使用联合查询，并以级联属性的方式封装对象

    ```xml
    <!--
    		联合查询：级联属性封装结果集
    	  -->
    	<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp">
    		<id column="id" property="id"/>
    		<result column="last_name" property="lastName"/>
    		<result column="gender" property="gender"/>
    		<result column="did" property="dept.id"/>
    		<result column="dept_name" property="dept.departmentName"/>
    	</resultMap>
    ```

  * 使用association标签定义对象的封装规则

  * Association-嵌套结果集

    ```xml
    <!-- 
    		使用association定义关联的单个对象的封装规则；
    	 -->
    	<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyDifEmp2">
    		<id column="id" property="id"/>
    		<result column="last_name" property="lastName"/>
    		<result column="gender" property="gender"/>
    		
    		<!--  association可以指定联合的javaBean对象
    		property="dept"：指定哪个属性是联合的对象
    		javaType:指定这个属性对象的类型[不能省略]
    		-->
    		<association property="dept" javaType="com.atguigu.mybatis.bean.Department">
    			<id column="did" property="id"/>
    			<result column="dept_name" property="departmentName"/>
    		</association>
    	</resultMap>
    ```

  * Association-分段查询

    Select：调用目标的方法查询当前属性值

    column：将指定列的值传入目标方法

    ```xml
    <!-- 使用association进行分步查询：
    		1、先按照员工id查询员工信息
    		2、根据查询员工信息中的d_id值去部门表查出部门信息
    		3、部门设置到员工中；
    	 -->
    	 
    	 <!--  id  last_name  email   gender    d_id   -->
    	 <resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmpByStep">
    	 	<id column="id" property="id"/>
    	 	<result column="last_name" property="lastName"/>
    	 	<result column="email" property="email"/>
    	 	<result column="gender" property="gender"/>
    	 	<!-- association定义关联对象的封装规则
    	 		select:表明当前属性是调用select指定的方法查出的结果
    	 		column:指定将哪一列的值传给这个方法
    	 		
    	 		流程：使用select指定的方法（传入column指定的这列参数的值）查出对象，并封装给property指定的属性
    	 	 -->
     		<association property="dept" 
    	 		select="com.atguigu.mybatis.dao.DepartmentMapper.getDeptById"
    	 		column="d_id">
     		</association>
    	 </resultMap>
    ```

  * association-分段查询与延迟加载

    ```xml
    <settings>		
    		<!--显示的指定每个我们需要更改的配置的值，即使他是默认的。防止版本更新带来的问题  -->
    		<setting name="lazyLoadingEnabled" value="true"/>
    		<setting name="aggressiveLazyLoading" value="false"/>
    	</settings>
    ```

* Collection-集合类型与嵌套结果集

  ```xml
  <!--嵌套结果集的方式，使用collection标签定义关联的集合类型的属性封装规则  -->
  	<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDept">
  		<id column="did" property="id"/>
  		<result column="dept_name" property="departmentName"/>
  		<!-- 
  			collection定义关联集合类型的属性的封装规则 
  			ofType:指定集合里面元素的类型
  		-->
  		<collection property="emps" ofType="com.atguigu.mybatis.bean.Employee">
  			<!-- 定义这个集合中元素的封装规则 -->
  			<id column="eid" property="id"/>
  			<result column="last_name" property="lastName"/>
  			<result column="email" property="email"/>
  			<result column="gender" property="gender"/>
  		</collection>
  	</resultMap>
  
  <!-- public Department getDeptByIdPlus(Integer id); -->
  	<select id="getDeptByIdPlus" resultMap="MyDept">
  		SELECT d.id did,d.dept_name dept_name,
  				e.id eid,e.last_name last_name,e.email email,e.gender gender
  		FROM tbl_dept d
  		LEFT JOIN tbl_employee e
  		ON d.id=e.d_id
  		WHERE d.id=#{id}
  	</select>
  	
  ```

  * Colletion-分步查询与延迟加载

    ```xml
    <!-- collection：分段查询 -->
    	<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDeptStep">
    		<id column="id" property="id"/>
    		<id column="dept_name" property="departmentName"/>
    		<collection property="emps" 
    			select="com.atguigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
    			column="{deptId=id}" fetchType="lazy"></collection>
    	</resultMap>
    	<!-- public Department getDeptByIdStep(Integer id); -->
    	<select id="getDeptByIdStep" resultMap="MyDeptStep">
    		select id,dept_name from tbl_dept where id=#{id}
    	</select>
    ```

* 扩展-多列值封装map传递

  * 分步查询的时候通过column指定，将对应的列的数据 传递过去，我们有时需要传递多列数据。

  * 使用{key1=column1,key2=column2...}的形式

    ```xml
    <!-- collection：分段查询 -->
    	<resultMap type="com.atguigu.mybatis.bean.Department" id="MyDeptStep">
    		<id column="id" property="id"/>
    		<id column="dept_name" property="departmentName"/>
    		<collection property="emps" 
    			select="com.atguigu.mybatis.dao.EmployeeMapperPlus.getEmpsByDeptId"
    			column="{deptId=id}" fetchType="lazy"></collection>
    	</resultMap>
    ```

  * association或者collection标签的 fetchType=eager/lazy可以覆盖全局的延迟加载策略， 指定立即加载(eager)或者延迟加载(lazy)