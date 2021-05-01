# Myatis入门

#### 简介

* MyBatis是支持定制化SQL、存储过程以及高级映射的优秀的持久层框架。
* Mybatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。
* Mybatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO映射成数据库中的记录。

#### Mybatis的优势

* Mybatis是一个半自动化的持久层框架。
* JDBC
  * SQL夹在Java代码块里，耦合度高导致硬编码内伤。
  * 维护不易且实际开发需求sql是有变化，频繁修改的情况多见。
* Hibernate和JPA
  * 长难复杂SQL，对于Hibernate而言处理也不容易。
  * 内部自动生产的SQL，不容易做特殊化。
  * 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难。导致数据库性能下降。
* 对于Mybatis，开发人员的核心sql还是需要自己优化，sql和java编码分开，功能边界清晰，一个专注业务，一个专注数据。

#### Mybatis的入门Demo

* ##### 创建一个员工表

  ```mysql
  CREATE TABLE `tbl_employee` (
    `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '员工编号',
    `last_name` varchar(20),
    `email` varchar(20) ,
    `gender` varchar(20),
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=1000 DEFAULT CHARSET=utf8;
  ```

* 创建对应的JavaBean

  ```java
  import lombok.Data;
  
  @Data
  public class Employee {
  
      private Integer id;
      private String lastName;
      private String email;
      private String gender;
  
  
  
      @Override
      public String toString() {
          return "Employee [id=" + id + ", lastName=" + lastName + ", email="
                  + email + ", gender=" + gender + "]";
      }
  
  
  }
  ```

* 创建Mapper接口

  ```java
  package com.sunwj.mybatis.mapper;
  
  
  import com.sunwj.mybatis.bean.Employee;
  
  public interface EmployeeMapper {
  	
  	public Employee getEmpById(Integer id);
  
  }
  
  ```

* 创建mybatis配置文件，sql映射文件

  Mybatis-config.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
   PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
   "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
  	<environments default="development">
  		<environment id="development">
  			<transactionManager type="JDBC" />
  			<dataSource type="POOLED">
  				<property name="driver" value="com.mysql.cj.jdbc.Driver" />
  				<property name="url" value="jdbc:mysql://localhost:3306/asiainfo?characterEncoding=utf8" />
  				<property name="username" value="root" />
  				<property name="password" value="1992Sunwj@" />
  			</dataSource>
  		</environment>
  	</environments>
  	<!-- 将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 -->
  	<mappers>
  		<mapper resource="mapper/EmployeeMapper.xml" />
  	</mappers>
  </configuration>
  ```

  recource/EmployeeMapper.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
   PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
   "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <mapper namespace="com.sunwj.mybatis.mapper.EmployeeMapper">
  <!-- 
  namespace:名称空间;指定为接口的全类名
  id：唯一标识
  resultType：返回值类型
  #{id}：从传递过来的参数中取出id值
  
  public Employee getEmpById(Integer id);
   -->
  	<select id="getEmpById" resultType="com.sunwj.mybatis.bean.Employee">
  		select id,last_name lastName,email,gender from tbl_employee where id = #{id}
  	</select>
  </mapper>
  ```

* 测试

  ```java
  package com.sunwj.mybatis;
  
  import com.sunwj.mybatis.bean.Employee;
  import com.sunwj.mybatis.mapper.EmployeeMapper;
  import org.apache.ibatis.io.Resources;
  import org.apache.ibatis.session.SqlSession;
  import org.apache.ibatis.session.SqlSessionFactory;
  import org.apache.ibatis.session.SqlSessionFactoryBuilder;
  import org.junit.Test;
  
  import java.io.IOException;
  import java.io.InputStream;
  
  /**
   * @author sunwjcoder
   * @version 1.0
   * @description
   * 1、接口式编程
   *  原生 ： Dao  -- > DaoImpl
   *  mybatis: Mapper --> xxMapper.xml
   *
   * 2、SqlSession代表和数据库的一次会话；用完必须关闭；
   * 3、SqlSession和connection一样它都是非线程安全的。每次使用都应该去获取新的对象。
   * 4、mapper没有实现类，但是mybatis会为这个接口生成一个代理对象（将接口和xml进行绑定）
   *  如：
   *      EmpoyeeMapper empMapper = sqlSession.getMapper(EmployeeMapper.class)；
   * 5、两个重要的配置文件：
   *      mybatis的全局配置文件：包含数据库连接池信息，事务管理信息等...系统运行环境信息
   *      sql映射文件：保存了每一个sql语句的映射信息：将sql抽取出来。
   *
   * @date 2021/5/1 1:56 下午
   */
  public class MyatisTest {
  
      public SqlSessionFactory getSqlSessionFactory() throws IOException{
  
          String resource = "mybatis-config.xml";
          // 加载配置文件
          InputStream inputStream = Resources.getResourceAsStream(resource);
          // 构建SqlSeesionFactory实例
          return new SqlSessionFactoryBuilder().build(inputStream);
      }
  
      /**
       * 1、根据xml配置（全局配置文件）创建一个SqlSessionFactory对象 和数据源一些运行环境信息
       * 2、sql映射文件，配置了每一个sql，以及sql的封装规则等
       * 3、将sql映射文件注册到全局配置文件中
       * 4、代码：
       *      1、根据全局配置文件得到SqlSessionFactory
       *      2、根据sqlSession工厂，获取到sqlSession对象使用它来执行增删改查
       *          一个sqlSession就是代表和数据库的一次会话，用完关闭
       *      3、使用sql的唯一标志来告诉Mybatis执行那个sql，sql都是保存在sql映射文件中的。
       */
      @Test
      public void test() throws IOException{
  
          // 2、获取sqlSession实例，能直接执行已经映射的sql语句
          SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
  
          SqlSession sqlSession = sqlSessionFactory.openSession();
  
          try {
              Employee employee = sqlSession.selectOne("com.sunwj.mybatis.mapper.EmployeeMapper.getEmpById",1000);
              System.out.println(employee);
          } finally {
              sqlSession.close();
          }
      }
  
      @Test
      public void test01() throws IOException{
  
          // 获取sqlSessionFactory
          SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
          // 获取sqlSession对象
          SqlSession openSession = sqlSessionFactory.openSession();
  
          try {
              // 获取接口的实现类对象
              // 会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
              EmployeeMapper employeeMapper = openSession.getMapper(EmployeeMapper.class);
              Employee employee = employeeMapper.getEmpById(1000);
              System.out.println(employeeMapper.getClass());
              System.out.println(employee);
          } finally {
              openSession.close();
          }
      }
  }
  
  ```

  