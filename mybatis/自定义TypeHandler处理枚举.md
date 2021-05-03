# 自定义TypeHandler处理枚举

​		我们可以通过自定义TypeHandler的形式来在设置参数或者取出结果集的时候自定义参数封装策略。

#### 步骤

1. 实现TypeHandler接口或者继承BaseTypeHandler

2. 使用@MappedTypes定义处理的java类型

   使用@MappedJdbcTypes定义jdbcType类型

3. 在自定义结果集标签或者参数处理的时候声明使用自定义TypeHandler进行处理或者在全局配置TypeHandler要处理javaType

#### 测试

1. 枚举类

   ```java
   /**
    * 希望数据库保存的是100,200这些状态码，而不是默认0,1或者枚举的名
    * @author lfy
    *
    */
   public enum EmpStatus {
   	LOGIN(100,"用户登录"),LOGOUT(200,"用户登出"),REMOVE(300,"用户不存在");
   	
   	
   	private Integer code;
   	private String msg;
   	private EmpStatus(Integer code,String msg){
   		this.code = code;
   		this.msg = msg;
   	}
   	public Integer getCode() {
   		return code;
   	}
   	
   	public void setCode(Integer code) {
   		this.code = code;
   	}
   	public String getMsg() {
   		return msg;
   	}
   	public void setMsg(String msg) {
   		this.msg = msg;
   	}
   	
   	//按照状态码返回枚举对象
   	public static EmpStatus getEmpStatusByCode(Integer code){
   		switch (code) {
   			case 100:
   				return LOGIN;
   			case 200:
   				return LOGOUT;	
   			case 300:
   				return REMOVE;
   			default:
   				return LOGOUT;
   		}
   	}
   	
   	
   }
   ```

   2. 全局配置枚举

      ```xml
      <typeHandlers>
      		<!--1、配置我们自定义的TypeHandler  -->
      		<typeHandler handler="com.atguigu.mybatis.typehandler.MyEnumEmpStatusTypeHandler" javaType="com.atguigu.mybatis.bean.EmpStatus"/>
      		<!--2、也可以在处理某个字段的时候告诉MyBatis用什么类型处理器
      				保存：#{empStatus,typeHandler=xxxx}
      				查询：
      					<resultMap type="com.atguigu.mybatis.bean.Employee" id="MyEmp">
      				 		<id column="id" property="id"/>
      				 		<result column="empStatus" property="empStatus" typeHandler=""/>
      				 	</resultMap>
      				注意：如果在参数位置修改TypeHandler，应该保证保存数据和查询数据用的TypeHandler是一样的。
      		  -->
      	</typeHandlers>
      ```

   3. 自定义TypeHandler

      ```java
      /**
       * 1、实现TypeHandler接口。或者继承BaseTypeHandler
       * @author lfy
       *
       */
      public class MyEnumEmpStatusTypeHandler implements TypeHandler<EmpStatus> {
      
      	/**
      	 * 定义当前数据如何保存到数据库中
      	 */
      	@Override
      	public void setParameter(PreparedStatement ps, int i, EmpStatus parameter,
      			JdbcType jdbcType) throws SQLException {
      		// TODO Auto-generated method stub
      		System.out.println("要保存的状态码："+parameter.getCode());
      		ps.setString(i, parameter.getCode().toString());
      	}
      
      	@Override
      	public EmpStatus getResult(ResultSet rs, String columnName)
      			throws SQLException {
      		// TODO Auto-generated method stub
      		//需要根据从数据库中拿到的枚举的状态码返回一个枚举对象
      		int code = rs.getInt(columnName);
      		System.out.println("从数据库中获取的状态码："+code);
      		EmpStatus status = EmpStatus.getEmpStatusByCode(code);
      		return status;
      	}
      
      	@Override
      	public EmpStatus getResult(ResultSet rs, int columnIndex)
      			throws SQLException {
      		// TODO Auto-generated method stub
      		int code = rs.getInt(columnIndex);
      		System.out.println("从数据库中获取的状态码："+code);
      		EmpStatus status = EmpStatus.getEmpStatusByCode(code);
      		return status;
      	}
      
      	@Override
      	public EmpStatus getResult(CallableStatement cs, int columnIndex)
      			throws SQLException {
      		// TODO Auto-generated method stub
      		int code = cs.getInt(columnIndex);
      		System.out.println("从数据库中获取的状态码："+code);
      		EmpStatus status = EmpStatus.getEmpStatusByCode(code);
      		return status;
      	}
      
      }
      ```

      