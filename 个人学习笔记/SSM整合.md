# SSM整合
## 一.搭建数据库环境

```my
CREATE DATABASE `ssmbuild`;
USE `ssmbuild`;
DROP TABLE IF EXISTS `books`;
CREATE TABLE `books` ( 
`bookID` INT(10) NOT NULL AUTO_INCREMENT COMMENT '书id',
`bookName` VARCHAR(100) NOT NULL COMMENT '书名',
`bookCounts` INT(11) NOT NULL COMMENT '数量',
`detail` VARCHAR(200) NOT NULL COMMENT '描述', 
KEY `bookID` (`bookID`) ) ENGINE=INNODB DEFAULT CHARSET=utf8 
INSERT INTO `books`(`bookID`,`bookName`,`bookCounts`,`detail`)VALUES 
(1,'Java',1,'从入门到放弃'),
(2,'MySQL',10,'从删库到跑路'),
(3,'Linux',5,'从进门到进牢');
```



## 二.搭建idea基本环境

1. 新建工程
2. 导入依赖
```xml
<dependencies>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <!--数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>

        <!-- 数据库连接池 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>
        <!--Servlet - JSP -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <!--Mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.2</version>
        </dependency>

        <!--Spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
    
    
		<!--引入pageHelper分页插件 -->
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper</artifactId>
			<version>5.0.0</version>
		</dependency>
    
    <!-- MBG逆向工程 -->
		<!-- https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core -->
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.3.5</version>
		</dependency>
    
    <dependency> 
        <groupId>com.fasterxml.jackson.core</groupId> 
        <artifactId>jackson-databind</artifactId> 
        <version>2.9.8</version> 
	 </dependency>
    
    <!--JSR303数据校验支持；tomcat7及以上的服务器， 
		tomcat7以下的服务器：el表达式。额外给服务器的lib包中替换新的标准的el
		-->
		<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-validator -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>5.4.1.Final</version>
		</dependency>
    
    <!--Spring-test -->
    <dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>4.3.7.RELEASE</version>
		</dependency>
    
    <!-- Spring面向切面编程 -->
		<!-- https://mvnrepository.com/artifact/org.springframework/spring-aspects -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aspects</artifactId>
			<version>4.3.7.RELEASE</version>
		</dependency>
    </dependencies>
```
3. maven静态资源过滤
使用Maven构建项目的时候，会默认过滤掉静态资源，所以，需要手动来配置
静态资源 : 包含HTMl,图片,CSS,JS等不需要与数据库交互的一类文件
动态资源 : 需要与数据库交互，可以根据需要显示不同的数据，不需要修改页面
```xml
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```
4. 建立基本包结构
dao ,service, controller, pojo

## 三.编写Mybatis层
1. 配置数据库文件database.properties

  > jdbc.driver=com.mysql.jdbc.Driver
  > jdbc.url=jdbc:mysql://localhost:3306/ssmbuild?useSSL=true&useUnicode=true&characterEncoding=utf8 
  > jdbc.username=root 
  > jdbc.password=zaxscd

2. 编写Mybatis-config.xml核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--    自动驼峰命名转换-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings> 
    
    <typeAliases>
        <package name="com.zlq.pojo"/>
    </typeAliases>

    <!--    分页插件-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <!--分页参数合理化  -->
            <property name="reasonable" value="true"/>
        </plugin>
    </plugins>
    
    <mappers>
        <mapper resource="com/zlq/dao/BookMapper.xml"/>
    </mappers>
</configuration>
```
3. 编写pojo实体类
```java
public class Books {
    private int bookID;
    private String bookName;
    private int bookCounts;
    private String detail;
}

```
4. 编写dao层的XxxMapper接口,并编写增删改查的抽象方法
```java
public interface BookMapper {
//    增加一本书
    int addBook(Books book);
//    根据id删除一本书
    int deleteBookById(int id);
//    更新图书信息
    int updateBook(Books book);
//    查询一本书的信息
    Books queryOneBook(int id);
//    查询多本书的信息
    List<Books> queryAllBooks();
}

```
6. 编写XxxMapper.xml,写入Mysql语句
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.zlq.dao.BookMapper">
    <insert id="addBook" parameterType="Books">
        insert into books(bookName,bookCounts,detail)
        values (#{bookName},#{bookCounts},#{detail})
    </insert>

    <delete id="deleteBookById" parameterType="int">
        delete from books where id = #{id}
    </delete>

    <update id="updateBook" parameterType="Books">
        update books set bookName = #{bookName},bookCounts = #{bookCounts},
        detail = #{detail}
    </update>

<!--    根据id返回一条书籍的记录-->
    <select id="queryOneBook" parameterType="int" resultType="Books">
        select * from books where id = #{id}
    </select>

<!--    返回所有书籍信息-->
    <select id="queryAllBooks"  resultType="Books">
        select * from books
    </select>

</mapper>
```
7.编写Service层接口和实现类
```java
public interface BookService {
    //    增加一本书
    int addBook(Books book);
    //    根据id删除一本书
    int deleteBookById(int id);
    //    更新图书信息
    int updateBook(Books book);
    //    查询一本书的信息
    Books queryOneBook(int id);
    //    查询多本书的信息
    List<Books> queryAllBooks();

}
```

```java
public class BookServiceImpl implements BookService{
    private BookMapper bookMapper;

    public void setBookMapper(BookMapper bookMapper) {
        this.bookMapper = bookMapper;
    }

    public int addBook(Books book) {
        return bookMapper.addBook(book);
    }

    public int deleteBookById(int id) {
        return bookMapper.deleteBookById(id);
    }

    public int updateBook(Books book) {
        return bookMapper.updateBook(book);
    }

    public Books queryOneBook(int id) {
        return bookMapper.queryOneBook(id);
    }

    public List<Books> queryAllBooks() {
        return bookMapper.queryAllBooks();
    }
}
```
## 四.编写Spring层
配合Spring整合Mybatis，这里使用C3P0数据库连接池

1. 编写Spring整合Mybatis的配置文件：Spring-dao.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
    <!--    配置整合Mybatis-->
    <!--    1.关联数据库文件-->
    <context:property-placeholder location="classpath:database.properties"/>
    <!--    2.配置数据库连接池-->
    <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" id="dataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>

        <!--连接池的最大连接数最小连接数-->
        <property name="maxPoolSize" value="30"/>
        <property name="minPoolSize" value="10"/>
        <!-- 关闭连接后不自动commit -->
        <property name="autoCommitOnClose" value="false"/>
        <!-- 获取连接超时时间 -->
        <property name="checkoutTimeout" value="10000"/>
        <!-- 当获取连接失败重试次数 -->
        <property name="acquireRetryAttempts" value="2"/>
    </bean>

    <!--3.配置SqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 注入数据库连接池 -->
        <property name="dataSource" ref="dataSource"/>
        <!--关联绑定Mybatis里的配置文件Mybatis-Config.xml-->
        <property name="configLocation" value="classpath:Mybatis-config.xml"/>
        <!--配置mapper映射文件资源路径 -->
        <property name="mapperLocations" value="classpath:com/zlq/dao/*.xml"/>
    </bean>

    <!-- 4.配置扫描Dao接口包，动态实现Dao接口注入到spring容器中 -->
    <!--解释 ： https://www.cnblogs.com/jpfss/p/7799806.html-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 给出需要扫描Dao接口的包 -->
        <property name="basePackage" value="com.zlq.dao"/>
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
    
    
    <!-- 配置一个可以执行批量的sqlSession -->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"></constructor-arg>
        <constructor-arg name="executorType" value="BATCH"></constructor-arg>
    </bean>
</beans>
```

2.Spring整合Service层
```xml
<?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop
           https://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd">

        <!--    扫描service的相关bean-->
        <context:component-scan base-package="com.zlq.service"/>

        <!--    配置事务管理器注入数据库连接池-->
        <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="dataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"/>
        </bean>

        <!--开启基于注解的事务，使用xml配置形式的事务（必要主要的都是使用配置式）  -->
        <aop:config>
            <!-- 切入点表达式 -->
            <aop:pointcut expression="execution(* com.zlq.service..*(..))" id="txPoint"/>
            <!-- 配置事务增强 -->
            <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
        </aop:config>

        <!--配置事务增强，事务如何切入  -->
        <tx:advice id="txAdvice" transaction-manager="dataSourceTransactionManager">
            <tx:attributes>
                <!-- 所有方法都是事务方法 -->
                <tx:method name="*"/>
                <!--以get开始的所有方法  -->
                <tx:method name="get*" read-only="true"/>
            </tx:attributes>
        </tx:advice>

    </beans>
```
Spring层也配置完毕，再次理解Spring就是个大杂烩，就是个容器！

## 五.编写SpringMVC层
1. 配置web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

<!--    1.注册DispatcherServlet-->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

<!--    2.配置encodingFilter-->
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceRequestEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

<!--    3.session过期时间-->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>
</web-app>
```

2.编写SpringMVC.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       https://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!-- 1.开启SpringMVC注解驱动 -->
    <mvc:annotation-driven/>
    <!-- 2.过滤静态资源，默认servlet配置，将SpringMVC不能处理的请求交给tomcat-->
    <mvc:default-servlet-handler/>
    <!--3.设置自动扫描包，让包下的注解生效，由ioc容器统一管理-->
    <context:component-scan base-package="com.zlq.controller"/>
    <!--4.配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

3. 整合Spring配置文件applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="Spring-service.xml"/>
    <import resource="Spring-dao.xml"/>
    <import resource="SpringMVC.xml"/>
</beans>
```

配置结束！





处理Json的乱码问题

```xml
<mvc:annotation-driven>    
    <mvc:message-converters register-defaults="true">        
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">            <constructor-arg value="UTF-8"/>        </bean>        
        <bean class="org.springframework.http.converter.smile.MappingJackson2SmileHttpMessageConverter">            <property name="objectMapper">                <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">                    <property name="failOnEmptyBeans" value="false"/>                </bean>            </property>        </bean>   
    </mvc:message-converters></mvc:annotation-driven>
```