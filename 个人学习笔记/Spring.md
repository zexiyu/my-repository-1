# Spring的发展历程

在02年，首次推出了Spring框架的雏形：interface21框架 

在04年，以interface21框架为基础进行了设计，发布Spring1.0

Spring是一位悉尼大学音乐学博士Rod Johnson发明的

# Spring的作用

Spring就是为了解决企业级应用开发的复杂性，为了简化开发而创建。



# Spring如何简化开发

为了降低Java开发的复杂性，Spring采用了以下4种关键策略： 

1、基于POJO的轻量级和最小侵入性编程，所有东西都是bean； 

2、通过IOC，依赖注入（DI）和面向接口实现松耦合； 

3、基于切面（AOP）和惯例进行声明式编程； 

4、通过切面和模版减少样式代码，RedisTemplate，xxxTemplate； 

# Spring的优点

- 开源免费
- 是一个容器
- 轻量级框架，非侵入式（安全）
- 控制反转IOC，面向切面AOP
- 支持事物，对其他框架有着良好的融合性，支持性

# Spring的组成（先了解）

Spring框架是一个分层架构，它共有七个模块组成

核心容器

Spring上下文

Spring AOP

Spring DAO

Spring ORM

Spring Web模块

Spring MVC框架





# IOC

1. 编写一个Hello实体类 

```java
public class Hello { 
    private String name;
    public String getName() {
        return name; 
    }
    public void setName(String name) { 
        this.name = name;
    }
    public void show(){
        System.out.println("Hello,"+ name );
    } 
}
```



2. 引入命名空间

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

异常解析：java.lang.IllegalStateException: BeanFactory not initialized or already closed - call 'refresh' before accessing beans via the ApplicationContext
该异常是在测试时        ApplicationContext context = new ClassPathXmlApplicationContext();
这行代码中没有写入xml文件的路径
正确写法：        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");



```xml
<bean id="hello" class="com.zlq.pojo.Hello"></bean>
```
这个配置是用Spring创建Hello类的对象，其中id相当于给当前配置起个别名，默认起别名为类名首字母小写，也推荐这么写，
该配置中的id一定要和此行代码中getBean()中的字符串对应   Hello hello = context.getBean("hello", Hello.class);



 ```xml
    <bean id="MysqlDaoImpl" class="com.zlq.dao.MysqlDaoImpl"/>
    <bean id="OracleDaoImpl" class="com.zlq.dao.OracleDaoImpl"/>
    <bean id="userServiceImpl" class="com.zlq.service1.UserServiceImpl">
        <property name="userDao" ref="OracleDaoImpl"/>
    </bean>
 ```

这个property配置中的name属性是注入对应类中的setter方法中的setXxx方法，而name对应Xxx首字母小写
例如这个property中name是userDao，Spring就会去对应的setter方法中去寻找setUserDao()来进行注入；property配置中的ref属性对应引入OracleDaoImpl的这个bean实例

5. 通过无参构造创建对象
```xml
<bean id="user" class="com.zlq.pojo.com.zlq.pojo.User">
        <property name="name" value="张立群"/>
</bean>
```
6. 通过有参构造创建对象
- 方法1.根据index下标创建 
```xml
<bean id="user" class="com.zlq.pojo.com.zlq.pojo.User">
       <constructor-arg index="0" value="张立群"/>
</bean>
```
- 方法2.根据参数类型创建
```xml
<bean id="user" class="com.zlq.pojo.com.zlq.pojo.User">
       <constructor-arg name="name" value="张立群"/>
</bean>
```
- 方法3.根据参数名称创建 
```xml
<bean id="user" class="com.zlq.pojo.com.zlq.pojo.User">
       <constructor-arg type="java.lang.String" value="张立群"/>
</bean>
```

7. bean中的name属性很智能
```xml
 <bean id="hello" name="h1 h2,h3;h4" class="com.zlq.pojo.Hello">
        <property name="name" value="Spring"/>
 </bean>
```
可以取多个name名，可以用空格 逗号 分号 进行分隔

8. 使用c命名空间、p命名空间注入
 - c命名空间的约束： xmlns:c="http://www.springframework.org/schema/c"
 - p命名空间的约束： xmlns:p="http://www.springframework.org/schema/p"

IOC总结：我们可以看出依赖注入的方式主要有构造器注入和setter注入

# 浅析bean的生命周期

## 简述：

bean的生命周期指的是由创的对象由执行到销毁的过程， 在传统的Java应用中，也就是通过new创建对象的方式bean的生命周期很简单，使用Java关键字 new 进行Bean 的实例化，然后该Bean 就能够使用了。一旦bean不再被使用，则由Java自动进行垃圾回收。 



## bean的生命周期

- 不使用后置处理器，由Spring创建对象bean的生命周期如下五步：

  1. 通过无参构造创建bean（有参不行）
  2. 通过set方法设置bean的属性值
  3. 调用 bean 的初始化的方法（需要在bean.xml中配置init-method属性）
  4. 获取到了bean对象，bean 已使用
  5. 调用 bean 的销毁的方法（需需要在bean.xml中配置destroy-method属性，手动关闭容器才会调用） 

  

- 使用bean的后置处理器， 生命周期如下七步
  1. 通过无参构造创建bean（有参不行）
  2. 通过set方法设置bean的属性值
  3. **把 bean 实例传递 bean 后置处理器的方法 postProcessBeforeInitialization **
  4. 调用 bean 的初始化的方法（需要在bean.xml中配置init-method属性）
  5. **把bean实例传递 bean后置处理器的方法 postProcessAfterInitialization **
  6. 获取到了bean对象，bean 已使用
  7. 调用 bean 的销毁的方法（需需要在bean.xml中配置destroy-method属性，手动关闭容器才会调用） 

## 测试一下：



### 一.不使用bean的后置处理器

1. 编写实体类

```java
public class Order {
    private String orderName;
    public Order() {
        System.out.println("1:通过无参构造创建Bean实例");
    }

    public void setOrderName(String orderName) {
        this.orderName = orderName;
        System.out.println("第二步 调用 set 方法设置属性值");
    }
    public void initMethod() {
        System.out.println("第三步 执行初始化的方法");
    }
    //创建执行的销毁的方法
    public void destroyMethod() {
        System.out.println("第五步 执行销毁的方法");
    }
}
```

2. bean中的xml配置：

```xml
 <bean id="order" class="com.zlq.Order" init-method="initMethod" destroy-method="destroyMethod">
        <property name="orderName" value="bilibili"/>
 </bean>
```



3. 编写测试方法：

注意： 

### 二.使用bean的后置处理器


创建一个类，实现BeanPostProcessor接口，重写postProcessBeforeInitialization和 postProcessAfterInitialization方法

```java
public class MyBeanPost implements BeanPostProcessor {
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}
```

2.在配置文件中注入

```xml
<bean id="myBeanPost" class="com.zlq.MyBeanPost"/>
```

3.直接使用原来的代码测试即可

4.测试结果如下

![1602853465878](/Users/liqunzhang/Library/Application Support/typora-user-images/1602853465878.png)

# 注解

首先要使用Spring注解，必须要在Spring配置中开启注解扫描！

```xml
<context:component-scan base-package="com.zlq.pojo"/>
```

1. @Autowired 是把对象值注入类的属性中，是注入属性的注解
2. @Qualifier(value = "cat"),其中的value值对应xml中bean的id值，如
```xml
<bean id="cat" class="com.zlq.pojo.Cat"/>
```
@Qualifier 不能单独使用， 要和@Autowired同时使用！而且@Qualifier要具体指定装配组件的id值

3. @Component("user")   //相当于 <bean id="user" class= "类路径"/>

4. @Value  可以不再pojo类中加setter方法，直接在属性中注入值；
```java
@Value("Larry")
private String name 
```
也可以写setter方法，在setter方法上注入值   
```java
 @Value("Larry")
    public void setName(String name) {
        this.name = name;
    }
```

5. @Component 和其衍生注解
    @Repository  Dao层
    @Service     service层
    @Controller  web层

6. @Resource注解：

   该注解可以根据类型注入，也可以根据名称注入

   

   @Resource如有指定的name属性，先按该属性进行byName方式查找装配； 

   其次再进行默认的byName方式进行装配，如果再不行就报异常

   @Resource如果没有指定的name属性，会先按该属性默认进行byName方式查找装配

   如果以上都不成功，则会回退到按byType的方式自动装配。 再不行，就报异常。 



@Autowired与@Resource异同： 

1. @Autowired与@Resource都可以用来装配bean。都可以写在字段上，或写在setter方法上。 
2.  @Autowired默认按类型装配（属于spring规范），默认情况下必须要求依赖对象必须存在，如果 要允许null 值，可以设置它的required属性为false：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualififier注解进行使用  
3. @Resource（属于J2EE复返），默认按照名称进行装配，名称可以通过name属性进行指定。如果 没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，如果注解写在 setter方法上默认取属性名进行装配。 当找不到与名称匹配的bean时才按照类型进行装配。但是 需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。 它们的作用相同都是用注解方式注入对象，但执行顺序不同。@Autowired先byType，@Resource先 byName。



7. 完全注解

  - 创建配置类代替xml
 ```java
@Configuration //作为配置类，替代 xml 配置文件
@ComponentScan(basePackages = {"com.zlq"})
public class SpringConfig {	
}
 ```
   - 测试类中不能使用ClassPathXmlApplicationContext，
      需要使用AnnotationConfigApplicationContext

# 代理

1. 静态代理
- 使用静态代理的好处：公共业务都由代理类来进行完成，实现业务分工
                   业务扩展时也会变得更加方便和集中！不需要去修改业务源代码了
               
- 静态代理存在的缺点：将代码写死，每个代理类都需要有一个被代理类，使得代码量变得庞大

- 静态代理中的代理类中只需要实现被代理类实现的那个接口，再创建那个接口类的成员变量
  将成员变量放入构造器中，然后写出增强的方法即可
  
  

2. 动态代理
- 动态代理的好处：动态代理可以自动生成，使代码更具有生命力,一个动态代理类可以
  代理多个类，代理的是接口

- 写动态代理类主要有三点
  1. 实现InvocationHandler接口，重写invoke方法
  2. 在invoke方法中写增强的方法
  3. 使用Proxy类中的newProxyInstance()生成代理类
  
  ```java
  public class DynamicProxy implements InvocationHandler {
      SayHello sayHello;
      public DynamicProxy(SayHello sayHello) {
          this.sayHello = sayHello;
      }
  
      public Object getProxy(){
          return Proxy.newProxyInstance(this.getClass().getClassLoader(),sayHello.getClass().getInterfaces(),this);
      }
  
  
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          sayHello();
          Object result = method.invoke(sayHello, args);
          return result;
      }
  
      public void sayHello(){
          System.out.println("Hello！");
      }
  }
  
  ```
  
  

AOP的基本思想：在不更改原代码的基础上实现功能的加强，而AOP的底层原理就是动态代理





# 用Spring实现AOP的三种方式



## 方式一：通过API实现AOP操作

1. 通过实现接口（如：AfterReturningAdvice接口，MethodBeforeAdvice接口）创建增强类，
写入对应的增强方法
2. 通过xml文件注册
```xml
<bean id="userService" class="com.zlq.service1.UserServiceImpl"/>
    <bean id="beforeLog" class="com.zlq.service1.BeforeLog"/>
    <bean id="afterLog" class="com.zlq.service1.AfterLog"/>

<!--    aop的配置-->
    <aop:config>
        <aop:pointcut id="pointCut" expression="execution(* com.zlq.service1.UserServiceImpl.*(..))"/>
        <aop:advisor advice-ref="beforeLog" pointcut-ref="pointCut"/>
        <aop:advisor advice-ref="afterLog" pointcut-ref="pointCut"/>
    </aop:config>
```

expression="execution(* com.zlq.service1.UserServiceImpl.*(..))"    * 代表被增强的类，最后一个*表示所有方法都要被增强

## 方式二：通过自定义增强类来实现AOP操作

 1.第一步：
```java
public class DiyPointCut {
    public void before() {
        System.out.println("---------方法执行前---------");
    }

    public void after() {
        System.out.println("---------方法执行后---------");
    }
}
```
 2. 第二步：
 ```xml
<bean id="userService" class="com.zlq.service1.UserServiceImpl"/>
<bean id="diyPointCut" class="com.zlq.service1.DiyPointCut"/>

<aop:config>
        <aop:aspect ref="diyPointCut">
            <aop:pointcut id="divPointCut" expression="execution(*
            com.zlq.service1.UserServiceImpl.*(..))"/>
            <aop:before method="before" pointcut-ref="divPointCut"/>
            <aop:before method="after" pointcut-ref="divPointCut"/>
        </aop:aspect>
    </aop:config>
 ```

## 方式三：通过自定义增强类使用注解来实现AOP操作 （推荐使用）

1. 去定义增强类，在增强类中加入注解 在类前面加@Aspect
before方法前加 @Before("execution(* com.zlq.service1.UserServiceImpl.*(..))")
after方法前加  @After("execution(* com.zlq.service1.UserServiceImpl.*(..))")
```java
@Aspect
public class AnnotationPointCut {
    @Before("execution(* com.zlq.service1.UserServiceImpl.*(..))")
    public void before() {
        System.out.println("---------方法执行前(annotation)---------");
    }

    @After("execution(* com.zlq.service1.UserServiceImpl.*(..))")
    public void after() {
        System.out.println("---------方法执行后(annotation)---------");
    }
}
```
    2. 配置xml文件
```xml
<!--注册被增强类-->
 <bean id="userService" class="com.zlq.service1.UserServiceImpl"/>   
<!--注册增强类-->
 <bean id="annotationPointCut" class="com.zlq.service1.AnnotationPointCut"/>
 <aop:aspectj-autoproxy/>
```

# Spring 整合Mybatis
## 方式一
1. 编写数据源（），代替Mybatis中的数据库连接配置，这里使用spring来管理数据源为例
还可以使用其他数据源如c3p0，druid
```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis? useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
        <property name="username" value="root"/>
        <property name="password" value="zaxscd"/>
</bean>
```

2. 配置sqlSessionFactory （核心）
```xml
 <!--配置SqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--关联Mybatis,绑定Mybatis里的配置文件Mybatis-Config.xml-->
        <property name="configLocation" value="classpath:Mybatis-Config.xml"/>
        <property name="mapperLocations" value="classpath:com/zlq/dao/*.xml"/>
    </bean>
```

3. 注册sqlSessionTemplate , 关联sqlSessionFactory（核心）
```xml
 <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate"> 
        <!--只能用构造器注入sqlSession，因为源码中SqlSessionTemplate类里面没有getSession方法-->
        <constructor-arg index="0" ref="sqlSessionFactory"/>
</bean>
```

4. 创建实现类
我们不需要创建sqlSession了，由spring来管理
以前的操作都是用SqlSession来执行，现在都是用SqlSessionTemplate来管理
```java
public class UserMapperImpl implements UserMapper {
    //我们不需要创建sqlSession了，由spring来管理
    //以前的操作都是用SqlSession来执行，现在都是用SqlSessionTemplate来管理，因为SqlSessionTemplate可以完全替代SqlSession
    private SqlSessionTemplate sqlSession;

    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    public List<User> selectUser() {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        return mapper.selectUser();
    }
}
```

5.注册userMapperImpl
```xml
 <bean id="userMapper" class="com.zlq.dao.UserMapperImpl">
        <property name="sqlSession" ref="sqlSession"/>
</bean>
```


## 方式二

1. 创建实现类
同方式一的区别在于实现类继承了SqlSessionDaoSupport类，该类里面有getSqlSession方法
```java
public class UserMapperImpl2 extends SqlSessionDaoSupport implements UserMapper {

    public List<User> selectUser() {
//        SqlSession sqlSession = getSqlSession();
//        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
//        return mapper.selectUser();
        return getSqlSession().getMapper(UserMapper.class).selectUser();
    }
}
```
我们不需要注册sqlSessionTemplate，相当于省略方式一的第三步
但是要在userMapper中注入sqlSessionFactory
```xml
 <bean id="userMapper2" class="com.zlq.dao.UserMapperImpl2">
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
 </bean>
```