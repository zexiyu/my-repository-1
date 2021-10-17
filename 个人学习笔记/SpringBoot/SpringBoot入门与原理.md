



 



# 简介

之前我们使用JavaWeb开发的应用非常麻烦，要实现一个功能的去做了很多步骤，后来使用SSM框架对JavaWeb开发进行了大量的简化，只是这种简化还远远不够，后来推出了SpringBoot框架

SpringBoot框架：就是一个基于JavaWeb的开发框架，类似于SpringMVC,的一个简化开发，约定大于配置的框架，能迅速开发JavaWeb应用,它的出现是为了我们更容易的使用Spring，而不是摒弃Spring！

SpringBoot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，springboot是整合了spring生态圈中的所有的框架 。

> ​		人们根据实际生产应用情况，选取其中实用功能和设计精华，重构出一些轻量级的框架；之后为了提高开发 效率，嫌弃原先的各类配置过于麻烦，于是开始提倡“约定大于配置”，进而衍生出一些一站式的解决方 案。 这就是Java企业级应用:J2EE->spring->springboot的过程。
>
> ​		随着 Spring 不断的发展，涉及的领域越来越多，项目整合开发需要配合各种各样的文件，慢慢变得不那么易用简单，违背了最初的理念，甚至人称配置地狱。Spring Boot 正是在这样的一个背景下被抽象出来的开发框架，目的为了让大家更容易的使用 Spring 、更容易的集成各种常用的中间件、开源软件； 

简而言之

SpringBoot是整合Spring技术栈的一站式框架

SpringBoot是简化Spring技术栈的快速开发脚手架



# Spring Boot的主要优缺点：

## SpringBoot优点

- 创建独立Spring应用
- **开箱即用**，提供各种默认配置来简化项目配置 
- 内嵌web服务器
- 自动starter依赖，简化构建配置
- 提供生产级别的监控、健康检查及外部化配置
- 没有冗余代码生成和XML配置的要求 

## SpringBoot缺点

- 人称版本帝，迭代快，需要时刻关注变化
- 封装太深，内部原理复杂，不容易精通

# 时代背景



## 微服务的介绍

- 微服务是一种架构风格,它是将一个应用拆分为一组小型服务
- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治.服务可以使用不同的语言、不同的存储技术

## 分布式的困难

- 远程调用
- 服务发现
- 负载均衡
- 服务容错
- 配置管理
- 服务监控
- 链路追踪
- 日志管理
- 任务调度
- ......

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210119202900089.png" alt="image-20210119202900089" style="zoom: 50%;" />

微服务：将一个大的应用模块拆分成多个微小的模块，每个模块间可以进行远程调用

分布式：有微服务必然产生分布式，即：将各个微小模块分配到不同的机器上



### 分布式的解决

- SpringBoot + SpringCloud



# 第一个SpringBoot应用

## 创建SpringBoot的方式

### 方式一：通过Spring官网提供的工具快速构建应用 

不推荐，傻子才这么做，除非特殊情况

Spring Initializr： <https://start.spring.io/>

### 方式二：使用 IDEA 直接创建项目

推荐

1. 创建一个新项目
2. 选择spring initializr 
3. 填写project信息完成创建

## 项目结构分析
1. 主启动类
2. application.properties配置文件，自动创建好的
3. 测试类
4. pom.xml

## pom文件结构分析
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
<!--    项目的父依赖-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.zlq</groupId>
    <artifactId>springboot-helloworld</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot-helloworld</name>
    <description>Larry's first SpringBoot programme</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
<!--        SpringBoot启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

<!--        SpringBoot单元测试-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- spring-boot-starter-web ，web场景启动器，帮我们导入了web模块正常运行所依赖的组件-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.4.RELEASE</version>
        </dependency>

    </dependencies>

    <build>
<!--        打包插件-->
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```

这几个依赖包的作用就是内嵌Tomcat 容器，集成各Spring组件。
比如 如果没有web 包 ，Spring 的两大核心功能 IOC 和 AOP 将不会生效。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器



### controller类的编写
```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    @ResponseBody
    public String helloController(){
        return "这是你的第一个SpringBoot程序";
    }
}

```

注意：一定要在SpringBootXxxApplication类所在包及其子包中写你的业务逻辑代码，在包外写是无法访问到的

### 测试

直接运行main方法，访问localhost:8080/hello  即可！



**简化部署**

引入plugin插件

```xml
		<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

把项目打成jar包

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210119204341347.png" alt="image-20210119204341347" style="zoom:50%;" />

在target包中有我们打包好的jar包

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210119204501278.png" alt="image-20210119204501278" style="zoom:50%;" />

直接在目标服务器java -jar命令执行即可。



#  配置文件

## SpringBoot的配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的；

•application.properties    ---properties配置文件

•application.yml   ---yml配置文件

配置文件的作用：修改SpringBoot自动配置中的默认值；有很多的配置值SpringBoot在底层都给我们配置好了



## Yaml语法

YAML（YAML Ain't Markup Language）

​	全称可以是：YAML  A Markup Language：是一个标记语言

​	也可以是    ：YAML   isn't Markup Language：不是一个标记语言；

反正不知道它是个啥，但是我们要知道它可是代替properties配置文件的好东西！毕竟可以减少配置代码的冗余！



### 前言：对比yaml和xml和properties的配置区别
以前的配置文件，大多数都是使用xml来配置；比如一个简单的端口配置，我们来对比下yaml和xml

传统xml配置：
```xml
<server> 
    <port>8081<port> 
</server>
```
使用properties

```properties
server.port = 8081
```



使用yaml配置

```yaml
server: 
    port:8081
```



### 语法要求

语法要求非常简单
1. yaml语言以键值对的形式出现，但是 : 的后面必须有空格，否则无法识别
2. 以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的
3. 属性和值的大小写敏感

**注**：yaml语法中，双引号和单引号表示值有区别

在配置文件中注入Person的属性值，在web页面输出personName

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121205204193.png" alt="image-20210121205204193" style="zoom:50%;" />

" ''：双引号： 双引号不会转义特殊字符，"张立群\tLiQun"中的空格标识符

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121205414393.png" alt="image-20210121205414393" style="zoom:50%;" />

' '：单引号；会转义特殊字符，它把\t通过转义并打印出来了

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121205347949.png" alt="image-20210121205347949" style="zoom:50%;" />



#### 对象：

​	k: v：在下一行来写对象的属性和值的关系；注意缩进

​		对象还是k: v的方式

```yaml
friends:
		lastName: zhangsan
		age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```



#### 数组（List、Set）：

用- 值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```



#### 定义map
- 方式一（行内写法）
```yaml
map: {key1: value1,key2: value2 ,key3: value3}   
```
注：使用yaml定义map要用{}包围,定义String的键上不允许使用中文，否则无法读取该键值对！


- 方式二
```yaml
map:
  key1: value1
  key2: value2 
  key3: value3
```
#### 
### 测试使用yaml配置注入属性
实体类代码如下
```java
@Component
@ConfigurationProperties(prefix = "student")
public class Student {
    private String name;
    private Integer age;
    private boolean adult;
    private Date birthday;
    private Map<String,Object> lessons;
    private List<Object> hobbies;
    private Pet pet;
}
```
yaml配置如下：
```yaml
student:
  name: 张立群
  age: 18
  adult: false
  birthday: 2002/11/11
  lessons:
    lessonOne: 数学
    lessonTwo: 英语
    lessonThree: 物理  
  hobbies: [code,travel,music,swim,cycle]
  pet:
    petStyle: 小狗哆哆
    age: 5
```
运行结果如下：
Student{name='张立群', age=18, adult=false, birthday=Mon Nov 11 00:00:00 GMT+08:00 2002, lessons={lessonOne=数学, lessonTwo=英语, lessonThree=物理}, hobbies=[code, travel, music, swim, cycle], pet=Pet{petStyle='小狗哆哆', age=5}}

@configurationProperties：注入属性的方法,告诉SpringBoot将本类中的属性和配置文件中的属性绑定。在 Spring Boot 项目中，我们将大量的参数配置在 application.properties 或 application.yml 文件中，通过 @ConfigurationProperties注解，我们可以方便的获取这些参数值

扩展 ：@EnableConfigurationProperties注解的作用：让使用 @ConfigurationProperties 注解的类生效。 

perfix：指明配置文件下的哪个属性



### 加入提示功能

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```



# 注入属性的方式

## 1.通过Spring容器的IOC注入
## 2.在实体类的属性上使用@Value属性注入
```java
@Component
public class User {
    @Value("123456")
    private Integer userId;
    @Value("张立群")
    private String userName;
    @Value("18")
    private Integer age;
}
```
## 3.通过yaml配置文件注入

## 4.通过properties配置文件注入
编写user.properties配置文件
```properties
user.userId=123456
user.userName=LarryZhang
user.age=18
```
User类如下
```java
@Component
@PropertySource(value = "classpath:user.properties")
public class User {
    @Value("${user.userId}")
    private Integer userId;

    @Value("${user.userName}")
    private String userName;

    @Value("#{23*23}")  //#{SPEL} Spring表达式
    private Integer age;
}
```

综上，建议使用yaml配置文件注入属性，因为@Value使用起来比较麻烦，需要每个属性都要赋值



## Value获取值和@ConfigurationProperties获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |



## 配置文件注入值数据校验

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    /**
     * <bean class="Person">
     *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

   //lastName必须是邮箱格式
    @Email
    //@Value("${person.last-name}")
    private String lastName;
    //@Value("#{11*2}")  
    private Integer age;
    //@Value("true")
    private Boolean boss;

    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

## 加载指定的配置文件

@**PropertySource**：加载指定的配置文件；

比如我们去resource目录下创建 person.properties,

```xml
 name=Larry
```



```java
@PropertySource(value = "classpath:person.properties") 

@Component //注册bean 

public class Person { 

@Value("${name}") 

private String name; 

...... 

}
```



# 多环境切换



## 多Profile文件

我们在主配置文件编写的时候，文件名可以是 application-{profile}.properties/yml , 
用来指定多个环境版本；例如：application-test.properties 代表测试环境配置 ，
application-dev.properties 代表开发环境配置



默认配置文件为application.properties/yaml；只要我们在默认配置文件中指定环境那么任何时候都会加载指定环境中的配置文件 application-{profile).yaml 

那么如何激活指定的Profile配置文件？

​	1、在application.properties配置文件中指定  spring.profiles.active=dev

​	2、命令行：

​		java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；

​		可以直接在测试的时候，配置传入命令行参数

​	3、虚拟机参数；

​		-Dspring.profiles.active=dev

​	4、通过maven命令参数 (即在使用maven打包时通过-P参数，-P后跟上profile的唯一id) mvn clean package -Pdev|prod|test

**如果默认配置文件和profile配置文件中有相同配置，那么profile中的配置优先**



## 配置文件加载位置

优先级1：项目路径下的config文件夹下的配置文件 

优先级2：项目路径下配置文件 

优先级3：resource资源路径(classpath)下的config文件夹配置文件 

优先级4：resource资源路径(classpath)下配置文件 

优先级由高到底，高优先级的配置会覆盖低优先级的配置； 

**SpringBoot**会从这四个位置全部加载主配置文件；互补配置；



我们还可以通过spring.config.location来改变默认的配置文件位置。项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；这种情况，一般是后期运维做的多，相同配置，外部指定的配置文件优先级最高 

如：java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties

```yaml
server: 
  port:  8081 
#选择要激活那个环境块 
spring: 
  profiles: 
    active: prod 
--- 
server: 
  port: 8083 
spring: 
  profiles: dev #配置开发环境的名称 
--- 
server: 
  port: 8084 
spring: 
  profiles: text #配置测试环境的名称
---
server: 
  port: 8085
spring: 
  profiles: prod #配置生产环境的名称 
```

@**Profile**注解：指定测试环境 @Profile("prod") @Profile("test")

该注解可以标注在类上也可以标注在方法上，表示该类/方法在指定的环境下才能生效









# 运行原理和自动配置原理探究

## 一、运行原理探究

### 自动配置

- SpringBoot帮我们自动配好Tomcat

  点进我们的主pom文件中的spring-boot-starter-web，发现里面有tomcat场景启动器

- <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121213206364.png" alt="image-20210121213206364" style="zoom:30%;" />

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121213141663.png" alt="image-20210121213141663" style="zoom:30%;" />

- SpringBoot帮我们自动配好SpringMVC

- 引入SpringMVC全套组件

  自动配好SpringMVC常用组件（功能）

- 自动配好Web常见功能，如：字符编码问题

- SpringBoot帮我们配置好了所有web开发的常见场景

- 默认的包结构

- 主程序所在包及其下面的所有子包里面的组件都会被默认扫描进来

  无需以前的包扫描配置

  想要改变扫描路径，@SpringBootApplication(scanBasePackages=**"com.zlq"**)

- 或者@ComponentScan 指定扫描路径

> @SpringBootApplication
> 等同于
> @SpringBootConfiguration
> @EnableAutoConfiguration
> @ComponentScan("com.zlq.boot")   
>
> 三个注解之和



- 各种配置拥有默认值

- - 默认配置最终都是映射到某个类上，如：MultipartProperties
  - 配置文件的值最终会绑定每个类上，这个类会在容器中创建对象

- 按需加载所有自动配置项

- - 非常多的starter

  - 引入了哪些场景这个场景的自动配置才会开启

  - SpringBoot所有的自动配置功能都在 spring-boot-autoconfigure 包里面

    



### pom.xml的结构

在我们的pom.xml中有个父依赖

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121214041248.png" alt="image-20210121214041248" style="zoom:30%;" />

点进spring-boot-starter-parent发现里面有很多plugin配置,它主要管理项目资源过滤及插件



```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
  </parent>
```

点进spring-boot-dependencies发现里面有很多<dependency>配置，原来它是用来管理SpringBoot所有依赖的地方，它正是
SpringBoot的核心配置文件，依赖版本控制中心

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121214243776.png" alt="image-20210121214243776" style="zoom:30%;" />



### SpringBoot的主启动程序类

搭建SpringBoot环境时自动生成

@SpringBootApplication：用来标注一个主程序类，说明这是一个Spring Boot应用，被标注后就代表这是应用程序的主启动类

看似只调用了run方法，但这个是启动了整个SpringBoot服务

```java 
@SpringBootApplication
public class SpringbootHelloworldApplication {

    public static void main(String[] args) {
        
        SpringApplication.run(SpringbootHelloworldApplication.class, args);
    }

}
```



- 点进SpringBootApplication：

```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited //这个和上面3个都是元注解
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

```

@**SpringBootConfiguration**:表示SpringBoot的主配置类，在这个类中可以在main方法里面调用run( )启动这个SpringBoot应用

@**ComponentScan**：自动扫描符合加载条件的组件，注入到IOC容器

@**EnableAutoConfiguration**：开启自动配置，以前我们需要进行大量的手动配置，现在SpringBoot帮我们进行自动配置



- 点进EnableAutoConfiguration：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}

```

​	@**AutoConfigurationPackage**：自动配置包，开启自动配置功能，指定了默认的包规则

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

​		@import:Spring的底层注解：给容器中导入组件

@**Import**(Registrar.class)： 导入组件Registrar.class,这个Registrar.class的作用是将指定的包里面的所有组件批量注册到Spring容器中来，而SpringBoot默认扫描注册的是主启动类下的包及其子包



回到上一层：

@**Import**(AutoConfigurationImportSelector.class)：用来给容器中导入组件，

AutoConfigurationImportSelector自动配置导入选择器，它会自动配置导入哪些选择器？

他会给容器中导入大量的自动配置类：XxxAutoConfiguration，导入这个场景所需的所有组件，并配置好这些组件，免去了我们手动配置



点进AutoConfigurationImportSelector类中找到selectImports方法

```JAVA
public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
  //利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

```java
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
          //获取到所有需要导入到容器中的候选配置类
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.getConfigurationClassFilter().filter(configurations);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
```

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121221722577.png" alt="image-20210121221722577" style="zoom:30%;" />



getCandidateConfigurations方法中

```java
//用来获得候选配置
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
//getSpringFactoriesLoaderFactoryClass():用来返回自动配置类EnableAutoConfiguration
//getBeanClassLoader():获取bean的类加载器
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(), 
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

```

```java
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;   //返回自动配置类EnableAutoConfiguration
}
```

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    //加载自动配置类的名字，返回一个list集合   
    String factoryTypeName = factoryType.getName();
        return (List)loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
    }
```

```java
 private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
        MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
//这里的get(classLoader)就是获取自动配置类EnableAutoConfiguration的类加载器
//先从缓存中获取自动配置类（可以避免重复加载配置类从而过多消耗资源），如果获取到自动配置类直接返回自动配置类
        if (result != null) {
            return result;
        } else {
 //如果缓存中没获取到，我们就去META-INF/spring.factories去获取类加载器
            try {
                Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
                LinkedMultiValueMap result = new LinkedMultiValueMap();

                while(urls.hasMoreElements()) {
                    URL url = (URL)urls.nextElement();
                    UrlResource resource = new UrlResource(url);
                    Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                    Iterator var6 = properties.entrySet().iterator();

                    while(var6.hasNext()) {
                        Entry<?, ?> entry = (Entry)var6.next();
                        String factoryTypeName = ((String)entry.getKey()).trim();
                        String[] var9 = StringUtils.commaDelimitedListToStringArray((String)entry.getValue());
                        int var10 = var9.length;

                        for(int var11 = 0; var11 < var10; ++var11) {
                            String factoryImplementationName = var9[var11];
                            result.add(factoryTypeName, factoryImplementationName.trim());
                        }
                    }
                }

                cache.put(classLoader, result);
                return result;
            } catch (IOException var13) {
                throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var13);
            }
        }
    }
```

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121223743038.png" alt="image-20210121223743038" style="zoom:50%;" />

依据上面，找到了是从META-INF/spring.factories，我们全局搜索spring.factories点进

这里就是SpringBoot的自动配置根源呢~

里面加载了大量的自动配置类！

```properties
文件里面写死了spring-boot一启动就要给容器中加载的所有配置类
spring-boot-autoconfigure-2.4.2.RELEASE.jar/META-INF/spring.factories
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

我们点进这些自动配置类，可以看到里面都是JavaConfig配置类，还注入了一些Bean

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {

	public static final String DEFAULT_PREFIX = "";

	public static final String DEFAULT_SUFFIX = "";

	private static final String[] SERVLET_LOCATIONS = { "/" };

	@Bean
	@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
	@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
	public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
		return new OrderedHiddenHttpMethodFilter();
	}
```

自动配置的实现是从全局classpath搜索META-INF/spring.factories，并获取EnableAutoConfifiguration指定的值，然后找到org.springframework.boot.autoconfigure包下的所有自动配置类，那里面全是自动配置类，通过@Confuguration注入到IOC容器中，然后自动配置类开始发生作用帮我们进行配置，免去了我们的手动配置。

可以看出J2EE (企业级分布式应用程序开发规范)中所有自动配置和解决方案都在spring-boot-autoconfifigure-1.5.9.RELEASE.jar；这个jar包下



## 二、自动配置原理探究

```java
@Configuration(proxyBeanMethods = false)   //表示这是个自动配置类，可以给容器中添加组件，和以前编写的配置类一样。而且是多实例的

@EnableConfigurationProperties(ServerProperties.class) //该注解的作用是：使在ServerProperties类中使用的@ConfigurationProperties注解生效。
//进入这个ServerProperties查看，将配置文件中对应的值和ServerProperties绑定起来； //并把HttpProperties加入到ioc容器中
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)  //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果 满足指定的条件，整个配置类里面的配置就会生效； 判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class)//判断当前项目有没有这个类 CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)//判断配置文件中是否存在值为enabled的server.servlet.encoding的属性。即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {

	private final Encoding properties;  //他已经和SpringBoot的配置文件映射了

    //只有一个有参构造器的情况下，参数的值就会从容器中拿
	public HttpEncodingAutoConfiguration(ServerProperties properties) {
		this.properties = properties.getServlet().getEncoding();
	}

    //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@Bean
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
		return filter;
	}

	@Bean
	public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
		return new LocaleCharsetMappingsCustomizer(this.properties);
	}

	static class LocaleCharsetMappingsCustomizer
			implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {

		private final Encoding properties;

		LocaleCharsetMappingsCustomizer(Encoding properties) {
			this.properties = properties;
		}

		@Override
		public void customize(ConfigurableServletWebServerFactory factory) {
			if (this.properties.getMapping() != null) {
				factory.setLocaleCharsetMappings(this.properties.getMapping());
			}
		}

		@Override
		public int getOrder() {
			return 0;
		}

	}

}
```

**总结：根据当前不同的条件判断，决定这个配置类是否生效！** 

- 一但这个配置类生效；这个配置类就会给容器中添加各种组件； 

- 这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的； 

- 所有在配置文件中能配置的属性都是在xxxxProperties类中封装着； 

- 配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
//从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
}
                             
```



**精髓**

1、SpringBoot启动先加载所有的自动配置类  xxxxxAutoConfiguration

2、我们看我们需要的功能有没有在SpringBoot默认写好的自动配置类当中； 

3、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件存在在其中，我们就不需 

要再手动配置了） 

4、给容器中自动配置类添加组件的时候，会从Xxxproperties类中获取某些属性。我们只需要在配置文件中 

指定这些属性的值即可； 

**xxxxAutoConfifigurartion**：自动配置类；给容器中添加组件 

**xxxxProperties:**封装配置文件中相关属性；



总结：

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。这些值都从xxxxProperties里面获取。而xxxProperties又和我们的配置文件application.properties/application.yaml进行了绑定
- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了
- 定制化配置

- - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改。

**xxxxxAutoConfiguration ---> 给我们导入很多组件  --->** **从xxxxProperties里面拿值  ----> application.properties/application.yaml中获取**

# 启动原理探究

```java
 public static void main(String[] args) {
        SpringApplication.run(SpringbootdataMysqlApplication.class, args);
    }
```



1. 由主启动类代码可知是先创建SpringApplication对象

```java
public class SpringApplication{
    
   //SpringBoot启动应用。调用的这个run方法
    public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
    }

	public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
        return (new SpringApplication(primarySources)).run(args);
    }
    
    public ConfigurableApplicationContext run(String... args) {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
        ConfigurableApplicationContext context = null;
        this.configureHeadlessProperty();
         //获取SpringApplicationRunListeners，从类路径下的找到META-INF/spring.factories配置的所有SpringApplicationRunListener
        SpringApplicationRunListeners listeners = this.getRunListeners(args);
         //回调所有SpringApplicationRunListener的starting方法
        listeners.starting(bootstrapContext, this.mainApplicationClass);

        try {
            //封装命令行参数
            ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
             /*
             * 准备环境：
             *  - 创建环境完成后回调所有SpringApplicationRunListener的environmentPrepared方法；表示环境				准备完成
             */
            ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
            this.configureIgnoreBeanInfo(environment);
              //打印Banner
            Banner printedBanner = this.printBanner(environment);
            //创建ApplicationContext。决定创建Web的IOC容器还是普通的IOC容器
            context = this.createApplicationContext();
            context.setApplicationStartup(this.applicationStartup);
            //准备上下文环境，将environment保存到IOC容器中，applyInitializers(context);回到之前保存的所有的ApplicationContextInitializer的initialize()方法,回调listeners.contextPrepared(context);
            this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
               //prepareContext完成后，回调listeners.contextLoaded(context);
             
            //刷新容器。IOC容器初始化
            this.refreshContext(context);
             //从IOC容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
            this.afterRefresh(context, applicationArguments);
             //所有的SpringApplicationRunListener回调finished方法
            stopWatch.stop();
            if (this.logStartupInfo) {
                (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
            }

            listeners.started(context);
            this.callRunners(context, applicationArguments);
        } catch (Throwable var10) {
            this.handleRunFailure(context, var10, listeners);
            throw new IllegalStateException(var10);
        }

        try {
            listeners.running(context);
            //整个SpringBoot启动完成以后，返回启动的IOC容器
            return context;
        } catch (Throwable var9) {
            this.handleRunFailure(context, var9, (SpringApplicationRunListeners)null);
            throw new IllegalStateException(var9);
        }
    }
}

```

  总结事件回调机制

- 需要配置在META-INF/spring.factories的事件是：
  - ApplicationContextInitializer。
  - SpringApplicationRunListener
- 不需要配置在META-INF/spring.factories，只需要加入到IOC容器的事件是：
  - ApplicationRunner。
  - CommandLineRunner

## 事件监听机制

配置在META-INF/spring.factories 

**ApplicationContextInitializer** 

```java
public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> { 
    @Override 
    public void initialize(ConfigurableApplicationContext applicationContext) { System.out.println("ApplicationContextInitializer...initialize..."+applicationContext); 
	 } 
}
```

**SpringApplicationRunListener** 

```java
public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {
    //必须有的构造器 
    public HelloSpringApplicationRunListener(SpringApplication application, String[] args){ 
    
    }
    @Override 
    public void starting() { 
        System.out.println("SpringApplicationRunListener...starting..."); 
    }
    @Override 
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemProperties().get("os.name");
        System.out.println("SpringApplicationRunListener...environmentPrepared.."+o); 
    }
    @Override 
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextPrepared..."); 
    }
    @Override 
    public void contextLoaded(ConfigurableApplicationContext context) { System.out.println("SpringApplicationRunListener...contextLoaded..."); 
    }
    @Override 
    public void finished(ConfigurableApplicationContext context, Throwable exception) { System.out.println("SpringApplicationRunListener...finished..."); 
    } 
}
```

配置（META-INF/spring.factories） 

```properties
org.springframework.context.ApplicationContextInitializer=\ com.atguigu.springboot.listener.HelloApplicationContextInitializer org.springframework.boot.SpringApplicationRunListener=\ com.atguigu.springboot.listener.HelloSpringApplicationRunListener
```

只需要放在ioc容器中 

**ApplicationRunner**

```java
@Component 
public class HelloApplicationRunner implements ApplicationRunner { 
    @Override 
    public void run(ApplicationArguments args) throws Exception { System.out.println("ApplicationRunner...run...."); 
    } 
}
```

**CommandLineRunner** 

```java
@Component 
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override 
    public void run(String... args) throws Exception { System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
    } 
}
```

# 提高：自定义starter

启动器模块是一个 空 jar 文件，仅提供辅助性依赖管理，这些依赖可能用于自动装配或者其他类库； 

**命名方式：** 

- 官方命名： 

spring-boot-starter-xxx ，比如：spring-boot-starter-web.... 

- 自定义命名： 

xxx-spring-boot-starter ，比如：mybatis-spring-boot-starter

## 1、在idea中新建个空工程spring-boot-starter-diy

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114170817545.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70#pic_center) 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114170846741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70#pic_center) 

建完以后里面什么都没有



## 2、新建一个普通Maven模块：zlq-spring-boot-starter

##  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114170549686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70#pic_center) 

## 3、新建一个Springboot模块：zlq-spring-boot-starter-autoconfigure 

基本结构如下：

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114170609847.png#pic_center) 

## 4、在我们的启动器模块starter中 导入 autoconfigure 的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>zlq-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
    <dependencies> <!-- 引入自动配置模块 -->
        <dependency>
            <groupId>com.zlq</groupId>
            <artifactId>zlq-spring-boot-starter-autoconfigure</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>bo
    </dependencies>

```



## 5、将 autoconfigure 项目下多余的文件都删掉（测试包也删掉），Pom中只留下一个 spring-boot-starter，这是所有的启动器基本配置

<img src="https://img-blog.csdnimg.cn/20201114170708358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70#pic_center" alt="在这里插入图片描述" style="zoom:50%;" />



## 6、在autoconfigure 模块中编写自己的配置

```java
ConfigurationProperties(prefix = "zlq.hello")
public class HelloProperties {
  
}
```



```java
public class HelloService {
    HelloProperties helloProperties;

    public HelloProperties getHelloProperties() {
        return helloProperties;
    }

    public void setHelloProperties(HelloProperties helloProperties) {
        this.helloProperties = helloProperties;
    }

    public String sayHello(String name){
        return helloProperties.getPrefix() + name + helloProperties.getSuffix();
    }
}
```

## 7、将配置类注入bean

```java
@Configuration
@ConditionalOnWebApplication   //使web应用生效
@ConditionalOnMissingBean(HelloService.class)
@EnableConfigurationProperties(HelloProperties.class) //使使用 @ConfigurationProperties 注解的类生效
public class HelloServiceConfiguration {
    @Autowired
    HelloProperties helloProperties;

    @Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        helloService.setHelloProperties(helloProperties);
        return helloService;
    }
}
```

## 8、在resources编写一个自己的 META-INF\spring.factories 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114170724324.png#pic_center) 

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.zlq.configuration.HelloServiceConfiguration
```

## 9、将两个模块打包

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114170747940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70#pic_center) 

## 10、创建新的SpringBoot项目spring-boot-starter-test，引入web环境

导入我们自己的依赖

```xml
<dependency>
    <groupId>com.zlq</groupId>
    <artifactId>zlq-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

## 11、编写controller类

```java
@Controller
public class HelloController {
    @Autowired
    HelloService helloService;
    @RequestMapping("/hello")
    @ResponseBody
    public String sayHello(){
        String hello = helloService.sayHello("张立群");
        return hello;
    }
}

```

## 12、在application.properties中编写配置

```properties
zlq.hello.prefix=Hello
zlq.hello.suffix=,you must be HAPPY everyday!
```

因为我们导入的启动器中的自动配置类HelloProperties中，绑定的前缀是zlq.hello，里面有setPerfix和setSuffix方法



## 13、启动项目测试

![1605344372019](/Users/liqunzhang/Library/Application Support/typora-user-images/1605344372019.png)

