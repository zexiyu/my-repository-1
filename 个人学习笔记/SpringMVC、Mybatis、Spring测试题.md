# SpringMVC、Mybatis、Spring测试题

###  1.请写出 spring 中常用的依赖注入方式。

常见的就是 setter 注入 和 构造方法 注入。
另外还有静态工厂的方法注入、实例工厂的方法注入。

### 2.简述Spring中IOC容器常用的接口和具体的实现类。

1. BeanFactory  SpringIOC容器的基本设置，是最底层的实现， 面向框架本身的.  

2.  ApplicationContext  BeanFactory的子接口, 提供了更多高级的特定. 面向开发者的.

3. ConfigurableApplicationContext, ApplicationContext的子接口，扩展出了 close 和 refresh等 关闭 刷新容器的方法

4. ClassPathXmlApplicationContext：从classpath的XML配置文件中读取上下文，并生成上下文定义。应用程序上下文从程序环境变量中取得。

5. FileSystemXmlApplicationContext ：由文件系统中的XML配置文件读取上下文。

6. XmlWebApplicationContext：由Web应用的XML文件读取上下文。

   

   

### 3. 简述Spring中如何基于注解配置Bean和装配Bean

<context:component-scan base-package=” ”></ context:component-scan>

在具体的类上加上具体的注解.
Spring 中通常使用@Autowired 或者是@Resource 等注解进行bean的装配.



### 4.说出Spring 或者 Springmvc中常用的5个注解 ，并解释含义

	   [1]. @Component  基本注解，标识一个受Spring管理的组件
	   [2]. @Controller  标识为一个表示层的组件
	   [3]. @Service       标识为一个业务层的组件
	   [4]. @Repository    标识为一个持久层的组件
	   [5]. @Autowired     自动装配
	   [6]. @Qualifier(“”)   具体指定要装配的组件的id值
	   [7]. @RequestMapping()  完成请求映射
	   [8]. @PathVariable    映射请求URL中占位符到请求处理方法的形参
	
	 只要说出5个注解并解释含义即可，如上答案只做参考
### 5.请解释Spring Bean的生命周期？

 1.默认情况下，IOC容器中bean的生命周期分为五个阶段:
① 调用构造器 或者是通过工厂的方式创建Bean对象
②给bean对象的属性注入值
③ 调用初始化方法，进行初始化， 初始化方法是通过init-method来指定的.
④使用
⑤IOC容器关闭时， 销毁Bean对象.

2.当加入了Bean的后置处理器后，IOC容器中bean的生命周期分为七个阶段:
    ① 调用构造器 或者是通过工厂的方式创建Bean对象
    ② 给bean对象的属性注入值 
	③ 执行Bean后置处理器中postProcessBeforeInitialization
	④ 调用初始化方法，进行初始化， 初始化方法是通过init-method来指定的.
	⑤ 执行Bean的后置处理器中 postProcessAfterInitialization   
	⑥ 使用
	⑦ IOC容器关闭时， 销毁Bean对象 
只需要回答出第一点即可。 第二点也回答可适当加分. 



### 6、简述SpringMvc里面拦截器是如何定义，如何配置，拦截器中三个重要的方法

- 定义: 有两种方式
          	1. 实现HandlerInterceptor接口
                   	2. 继承HandlerInterceptorAdapter

- 配置:

  ```xml
  <mvc:interceptors>
  <!--默认是对所有请求都拦截 -->
  <bean id="myFirstInterceptor" class="com.atguigu.interceptor.MyFirstInterceptor">
  </bean>	
  <!-- 只针对部分请求拦截或者不拦截 -->
  <mvc:interceptor>
  <mvc:mapping path=" " />  <!--指定拦截-->
  <mvc:exclude-mapping path=””/> <!--指定不拦截-->
  <bean class=" com.atguigu.interceptor.MySecondInterceptor " /> </mvc:interceptor>
  </mvc:interceptors>
  ```

  - 拦截器中三个重要的方法
     	1. preHandle
        	2. postHandle
                	3. afterCompletion

  

 ### 7.简单的谈一下SpringMVC的工作流程？

  1、用户发送请求至前端控制器DispatcherServlet 
  2、DispatcherServlet收到请求调用HandlerMapping处理器映射器。 
  3、处理器映射器找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。 
  4、DispatcherServlet调用HandlerAdapter处理器适配器 
  5、HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。 
  6、Controller执行完成返回ModelAndView 
  7、HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet 
  8、DispatcherServlet将ModelAndView传给ViewReslover视图解析器 
  9、ViewReslover解析后返回具体View 
  10、DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。 
  11、DispatcherServlet响应用户

  

  ### 8、Springmvc中如何解决POST请求中文乱码问题 

    Springmvc中通过CharacterEncodingFilter解决中文乱码问题. 
  在web.xml中加入：
  	<filter>
  	    <filter-name>CharacterEncodingFilter</filter-name>
  	    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
  	    <param-name>encoding</param-name>
         <param-value>utf-8</param-value>
      </init-param>
  	</filter>
  	<filter-mapping>
  	    <filter-name>CharacterEncodingFilter</filter-name>
  	    <url-pattern>/*</url-pattern>
  	</filter-mapping>

###  9、MyBatis中 #{}和${}的区别是什么？

  #{}是预编译处理，${}是字符串替换。
  Mybatis在处理#{}时，会将sql中的#{}替换为?号，调用PreparedStatement的set方法来赋值；
  Mybatis在处理${}时，就是把${}替换成变量的值。
  使用#{}可以有效的防止SQL注入，提高系统安全性。



###  10、Mybatis 结果集 的 映射方式有几种，并分别解释每种映射方式如何使用。 

   -   自动映射 ，通过resultType来指定要映射的类型即可。 
   -   自定义映射 通过resultMap来完成具体的映射规则，指定将结果集中的哪个列映射到对象的哪个属性。



### 11、简述MyBatis的单个参数、多个参数如何传递及如何取值。 

MyBatis传递单个参数，如果是普通类型(String+8个基本)的，取值时在#{}中可以任意指定，如果是对象类型的，则在#{}中使用对象的属性名来取值

 MyBatis传递多个参数，默认情况下，MyBatis会对多个参数进行封装Map. 取值时
      在#{}可以使用0 1 2 .. 或者是param1 param2.. 

MyBatis传递多个参数，建议使用命名参数，在Mapper接口的方法的形参前面使用
  @Param() 来指定封装Map时用的key. 取值时在#{}中使用@Param指定的key.



### 12、MyBatis如何获取自动生成的(主)键值?

      在<insert>标签中使用 useGeneratedKeys   和  keyProperty 两个属性来获取自动生成的主键值。
示例: 
    <insert id="insertname" usegeneratedkeys="true" keyproperty="id"> 
     insert into names (name) values (#{name}) 
    </insert>



### 13、简述Mybatis的动态SQL，列出常用的6个标签及作用

	动态SQL是MyBatis的强大特性之一 基于功能强大的OGNL表达式。 
	动态SQL主要是来解决查询条件不确定的情况，在程序运行期间，根据提交的条件动态的完成查询
	常用的标签:
	<if> : 进行条件的判断
	<where>：在<if>判断后的SQL语句前面添加WHERE关键字，并处理SQL语句开始位置的AND 或者OR的问题
	<trim>：可以在SQL语句前后进行添加指定字符 或者去掉指定字符.
	<set>:  主要用于修改操作时出现的逗号问题
	<choose> <when> <otherwise>：类似于java中的switch语句.在所有的条件中选择其一
   <foreach>：迭代操作	



### 14、Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

不同的Xml映射文件，如果配置了namespace，那么id可以重复；如果没有配置namespace，那么id不能重复；



###  15、Mybatis 如何完成MySQL的批量操作,举例说明

MyBatis完成MySQL的批量操作主要是通过<foreach>标签来拼装相应的SQL语句. 
      例如: 
      <insert id="insertBatch" >
		 insert into tbl_employee(last_name,email,gender,d_id) values 
		<foreach collection="emps" item="curr_emp" separator=",">
	(#{curr_emp.lastName},#{curr_emp.email},#{curr_emp.gender},#{curr_emp.dept.id})
		</foreach>
	</insert>



### 16、简述Spring中如何给bean对象注入集合类型的属性。 

	Spring使用 <list>  <set>  <map> 等标签给对应类型的集合注入值



### 17、简述Spring 中bean的作用域

1、singleton 表示在spring容器中的单例，通过spring容器获得该bean时总是返回唯一的实例
2、prototype表示每次获得bean都会生成一个新的对象
3、request表示在一次http请求内有效（只适用于web应用）
4、session表示在一个用户会话内有效（只适用于web应用）
5、global session表示在全局会话内有效（只适用于web应用）



### 18、简述Spring中自动装配常用的两种装配模式

- byName:  根据bean对象的属性名 进行装配

- byType： 根据bean对象的属性的类型进行装配,需要注意匹配到多个兼容类型的bean对象时，会抛出异常.

  

### 19、简述动态代理的原理， 常用的动态代理的实现方式

  动态代理的原理: 使用一个代理将对象包装起来，然后用该代理对象取代原始对象。任何对原始对象的调用都要通过代理。

代理对象决定是否以及何时将方法调  用转到原始对象上
      动态代理的方式
       基于接口实现动态代理： JDK动态代理
       基于继承实现动态代理： Cglib、Javassist动态代理



### 20、请解释@Autowired注解的工作机制及required属性的作用

- 该属性为true意味着容器中必须有这个组件，否则会报错
- 该属性为false意味着容器中有没有这个属性无所谓，没有会自动跳过不报错。