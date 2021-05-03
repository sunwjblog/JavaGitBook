# PageHelper插件进行分页

PageHelper是MyBatis中非常方便的第三方分页插件

官方文档：https://github.com/pagehelper/Mybatis-PageHelper/blob/master/README_zh.md

可以按照官方文档的说明，快速的使用插件

#### 使用步骤

1. 导入相关包pagehelper-xxx.jar和jsqlparser-0.9.5.jar

2. 在MyBatis全局配置文件中配置分页插件。

   ```xml
   <plugins>
   		<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
   	</plugins>
   ```

3. 使用PageHelper提供的方法进行分页
4. 可以使用更强大的PageInfo封装返回结果