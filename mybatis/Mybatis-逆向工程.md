# Mybatis-逆向工程

#### Mybatis Generator

​		简称MBG，是一个专门为Mybatis框架使用者定制的代码生成器，可以快速的根据表生成对应的映射文件，接口，以及bean类。支持基本的增删改查，以及QBC风格的条件查询。但是表连接、存储过程等这些复杂sql的定义需要手动编写

官方文档地址：http://mybatis.org/generator/

官方工程地址：https://github.com/mybatis/generator/releases

#### MBG使用

* 编写MBG的配置文件
  * jdbcCOnnection配置数据库连接信息
  * javaModelGenerator配置javaBean的生成策略
  * sqlMapGenerator配置sql映射文件生成策略
  * javaClientGenerator配置Mapper接口的生成策略
  * table配置要逆向解析的数据库表
    * tableName：表名
    * domainObjectName：对应的javaBean名
* 运行代码生成器生成代码
* 注意点
  * Context标签
  * targetRuntime="MyBatis3"可以生成带条件的增删改查
  * targetRuntime="MyBatis3Simple"可以生成基本的增删改查，如果再次生成，建议将之前生成的数据删除，避免xml向后追加内容出现问题。

#### MBG配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>

	<!-- 
		targetRuntime="MyBatis3Simple":生成简单版的CRUD
		MyBatis3:豪华版
	
	 -->
  <context id="DB2Tables" targetRuntime="MyBatis3">
  	<!-- jdbcConnection：指定如何连接到目标数据库 -->
    <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
        connectionURL="jdbc:mysql://localhost:3306/asiainfo?characterEncoding=utf8"
        userId="root"
        password="1992Sunwj@">
    </jdbcConnection>

	<!--  -->
    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

	<!-- javaModelGenerator：指定javaBean的生成策略 
	targetPackage="test.model"：目标包名
	targetProject="\MBGTestProject\src"：目标工程
	-->
    <javaModelGenerator targetPackage="com.atguigu.mybatis.bean" 
    		targetProject=".\src">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

	<!-- sqlMapGenerator：sql映射生成策略： -->
    <sqlMapGenerator targetPackage="com.atguigu.mybatis.dao"  
    	targetProject=".\conf">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>

	<!-- javaClientGenerator:指定mapper接口所在的位置 -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="com.atguigu.mybatis.dao"  
    	targetProject=".\src">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>

	<!-- 指定要逆向分析哪些表：根据表要创建javaBean -->
    <table tableName="tbl_dept" domainObjectName="Department"></table>
    <table tableName="tbl_employee" domainObjectName="Employee"></table>
  </context>
</generatorConfiguration>

```

**代码**

```java
public void testMbg() throws Exception {
		List<String> warnings = new ArrayList<String>();
		boolean overwrite = true;
		File configFile = new File("mbg.xml");
		ConfigurationParser cp = new ConfigurationParser(warnings);
		Configuration config = cp.parseConfiguration(configFile);
		DefaultShellCallback callback = new DefaultShellCallback(overwrite);
		MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
				callback, warnings);
		myBatisGenerator.generate(null);
	}
```

**测试**

```java
public void testMyBatis3() throws IOException{
		SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
		SqlSession openSession = sqlSessionFactory.openSession();
		try{
			EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
			//xxxExample就是封装查询条件的
			//1、查询所有
			//List<Employee> emps = mapper.selectByExample(null);
			//2、查询员工名字中有e字母的，和员工性别是1的
			//封装员工查询条件的example
			EmployeeExample example = new EmployeeExample();
			//创建一个Criteria，这个Criteria就是拼装查询条件
			//select id, last_name, email, gender, d_id from tbl_employee 
			//WHERE ( last_name like ? and gender = ? ) or email like "%e%"
			Criteria criteria = example.createCriteria();
			criteria.andLastNameLike("%e%");
			criteria.andGenderEqualTo("1");
			
			Criteria criteria2 = example.createCriteria();
			criteria2.andEmailLike("%e%");
			example.or(criteria2);
			
			List<Employee> list = mapper.selectByExample(example);
			for (Employee employee : list) {
				System.out.println(employee.getId());
			}
			
		}finally{
			openSession.close();
		}
	}
	
```

