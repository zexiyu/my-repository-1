# SSM整合bug汇总

- 感觉代码没写错那就可能是环境出了问题，刷新maven工程为先

  否则很容易出404或500错误，之前是

  之前因为这个问题卡了几个小时

![1602493332349](/Users/liqunzhang/Library/Application Support/typora-user-images/1602493332349.png)

![1602493341367](/Users/liqunzhang/Library/Application Support/typora-user-images/1602493341367.png)

![1602493347719](/Users/liqunzhang/Library/Application Support/typora-user-images/1602493347719.png)

总的配置文件已经将分文件导入，web.xml中已经关联总配置文件applicationContext.xml,但运行Tomcat出现500错误，报找不到这个配置文件

解决办法：重新reimport maven工程，或者重启即可！

- 运行tomcat时，点击显示 查询所有书籍页面，网页出现找不到JDBC连接？！明明数据库连接池等已经配置好！

```
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLException: Connections could not be acquired from the underlying database!
### The error may exist in com/zlq/dao/BookMapper.xml
### The error may involve com.zlq.dao.BookMapper.queryAllBooks
### The error occurred while executing a query
### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLException: Connections could not be acquired from the underlying database!
	org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
	org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:149)
	org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:140)
	sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	java.lang.reflect.Method.invoke(Method.java:498)
	org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:426)
	com.sun.proxy.$Proxy73.selectList(Unknown Source)
	org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:223)
	org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:147)
	org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:80)
	org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:57)
	com.sun.proxy.$Proxy74.queryAllBooks(Unknown Source)
	com.zlq.service.BookServiceImpl.queryAllBooks(BookServiceImpl.java:51)
	com.zlq.controller.BookController.List(BookController.java:31)
	sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	java.lang.reflect.Method.invoke(Method.java:498)
	org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:190)
	org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:138)
	org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:104)
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:892)
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:797)
	org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1039)
	org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:942)
	org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1005)
	org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:897)
	javax.servlet.http.HttpServlet.service(HttpServlet.java:622)
	org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:882)
	javax.servlet.http.HttpServlet.service(HttpServlet.java:729)
	org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
	org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200)
	org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:118)
```

而控制台报错内容为：

```
 java.sql.SQLException: Unsupported character encoding 'utf8 '.
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:965)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:898)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:887)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:861)
	at com.mysql.jdbc.ConnectionPropertiesImpl.postInitialization(ConnectionPropertiesImpl.java:2575)
	at com.mysql.jdbc.ConnectionPropertiesImpl.initializeProperties(ConnectionPropertiesImpl.java:2545)
	at com.mysql.jdbc.ConnectionImpl.initializeDriverProperties(ConnectionImpl.java:3143)
	at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:762)
	at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:47)
	at sun.reflect.GeneratedConstructorAccessor20.newInstance(Unknown Source)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
	at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:386)
	at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:330)
	at com.mchange.v2.c3p0.DriverManagerDataSource.getConnection(DriverManagerDataSource.java:175)
	at com.mchange.v2.c3p0.WrapperConnectionPoolDataSource.getPooledConnection(WrapperConnectionPoolDataSource.java:220)
	at com.mchange.v2.c3p0.WrapperConnectionPoolDataSource.getPooledConnection(WrapperConnectionPoolDataSource.java:206)
	at com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool$1PooledConnectionResourcePoolManager.acquireResource(C3P0PooledConnectionPool.java:203)
	at com.mchange.v2.resourcepool.BasicResourcePool.doAcquire(BasicResourcePool.java:1138)
	at com.mchange.v2.resourcepool.BasicResourcePool.doAcquireAndDecrementPendingAcquiresWithinLockOnSuccess(BasicResourcePool.java:1125)
	at com.mchange.v2.resourcepool.BasicResourcePool.access$700(BasicResourcePool.java:44)
	at com.mchange.v2.resourcepool.BasicResourcePool$ScatteredAcquireTask.run(BasicResourcePool.java:1870)
	at com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:696)

12-Oct-2020 16:56:15.717 警告 [C3P0PooledConnectionPoolManager[identityToken->1hge715ad7rlxt895ud0t|324229a6]-HelperThread-#1] com.mchange.v2.resourcepool.BasicResourcePool. com.mchange.v2.resourcepool.BasicResourcePool$ScatteredAcquireTask@373cbb8b -- Acquisition Attempt Failed!!! Clearing pending acquires. While trying to acquire a needed new resource, we failed to succeed more than the maximum number of allowed acquisition attempts (2). Last acquisition attempt exception: 
 java.sql.SQLException: Unsupported character encoding 'utf8 '.
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:965)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:898)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:887)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:861)
	at com.mysql.jdbc.ConnectionPropertiesImpl.postInitialization(ConnectionPropertiesImpl.java:2575)
	at com.mysql.jdbc.ConnectionPropertiesImpl.initializeProperties(ConnectionPropertiesImpl.java:2545)
	at com.mysql.jdbc.ConnectionImpl.initializeDriverProperties(ConnectionImpl.java:3143)
	at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:762)
	at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:47)
	at sun.reflect.GeneratedConstructorAccessor20.newInstance(Unknown Source)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:425)
	at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:386)
	at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:330)
	at com.mchange.v2.c3p0.DriverManagerDataSource.getConnection(DriverManagerDataSource.java:175)
	at com.mchange.v2.c3p0.WrapperConnectionPoolDataSource.getPooledConnection(WrapperConnectionPoolDataSource.java:220)
	at com.mchange.v2.c3p0.WrapperConnectionPoolDataSource.getPooledConnection(WrapperConnectionPoolDataSource.java:206)
	at com.mchange.v2.c3p0.impl.C3P0PooledConnectionPool$1PooledConnectionResourcePoolManager.acquireResource(C3P0PooledConnectionPool.java:203)
	at com.mchange.v2.resourcepool.BasicResourcePool.doAcquire(BasicResourcePool.java:1138)
	at com.mchange.v2.resourcepool.BasicResourcePool.doAcquireAndDecrementPendingAcquiresWithinLockOnSuccess(BasicResourcePool.java:1125)
	at com.mchange.v2.resourcepool.BasicResourcePool.access$700(BasicResourcePool.java:44)
	at com.mchange.v2.resourcepool.BasicResourcePool$ScatteredAcquireTask.run(BasicResourcePool.java:1870)
	at com.mchange.v2.async.ThreadPoolAsynchronousRunner$PoolThread.run(ThreadPoolAsynchronousRunner.java:696)
```

 java.sql.SQLException: Unsupported character encoding 'utf8 '.

utf8无法识别？发现问题！

```
jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8
```

这句话最后多了个空格！太草了！

要牢记配置utf8的各种配置里这个字符都不能带空格的！



- 前端页面的表格中用户名和密码中没有添加name属性，导致提交表单后，后端收不到前端发送的表单数据

  控制台居然输出为null！

  ```xml
  <form action="${pageContext.request.contextPath}/user/log" method="post">
      <p> 用户名:<input type="text" id="username" （没有name属性）  onblur="a1()"/> <span id="userInfo"></span></p>
      <p> 密码:<input type="text" id="password" （没有name属性） onblur="a2()"/> <span id="passwordInfo"></span></p>
      <input type="submit" value="提交">
  </form>
  ```

  ```java
    @RequestMapping("/log")
      public String log(HttpSession httpSession,String username,String password){
          httpSession.setAttribute("username",username);   
          System.out.println("接收前端的username = " + username);
          return "redirect:/";
      }
  ```

  

  ![1602666468889](/Users/liqunzhang/Library/Application Support/typora-user-images/1602666468889.png)



解决后：

```xml
<form action="${pageContext.request.contextPath}/user/log" method="post">
    <p> 用户名:<input type="text" id="username"   onblur="a1()"/> <span id="userInfo"></span></p>
    <p> 密码:<input type="text" id="password"  onblur="a2()"/> <span id="passwordInfo"></span></p>
    <input type="submit" value="提交">
</form>
```

控制台也受到了前端提交的数据！

![1602666647660](/Users/liqunzhang/Library/Application Support/typora-user-images/1602666647660.png)

- 在XxxController类中的某方法上未加@ResponseBody方法导致前端页面无法接受到数据

  

![1603354696230](/Users/liqunzhang/Library/Application Support/typora-user-images/1603354696230.png)



发现到没有走视图解析器，原因就在方法上没加@ResponseBody注解，果然如此！

![1603354715653](/Users/liqunzhang/Library/Application Support/typora-user-images/1603354715653.png)



加上以后完美解决！