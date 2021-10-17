# 容器功能及注解的介绍

## 1、@Configuration

告诉SpringBoot这是一个配置类 等同于我们以前写的配置文件

配置类里面使用@Bean标注在方法上用来往容器里注册组件



### @Configuration注解中参数proxyBeanMethods的讲解



@Configuration注释中的proxyBeanMethods参数是springboot1.0，升级到springboot2.0之后新增的比较重要的内容，
该参数是用来代理bean的

**1.理论**

**首先引出两个概念：Full 全模式，Lite 轻量级模式**

- Full(proxyBeanMethods = true) :proxyBeanMethods参数设置为true时即为：Full 全模式。 该模式下注入容器中的同一个组件无论被取出多少次都是**同一个bean实例**，即单实例对象，在该模式下SpringBoot每次启动都会判断检查容器中是否存在该组件
- Lite(proxyBeanMethods = false) :proxyBeanMethods参数设置为false时即为：Lite 轻量级模式。该模式下注入容器中的同一个组件无论被取出多少次都是**不同的bean实例**，即多实例对象，在该模式下SpringBoot每次启动会跳过检查容器中是否存在该组件
  **什么时候用Full全模式，什么时候用Lite轻量级模式？**
- 当在你的同一个Configuration配置类中，注入到容器中的bean实例之间有依赖关系时，建议使用Full全模式
- 当在你的同一个Configuration配置类中，注入到容器中的bean实例之间没有依赖关系时，建议使用Lite轻量级模式，以提高springboot的启动速度和性能

**2.实操验证**

新建一个Boy类和Girl类

```java
 public class Boy {
    private String name;
    private Integer age;
}
public class Girl {
    private String name;
    private Integer age;
    private Boy boyfriend;
}
```

新建一个Config类

```java
@Configuration(proxyBeanMethods = true)
public class MyConfiguration {

    @Bean
    public Boy getBoy(){
        Boy boy = new Boy("Jerry",18);
        return Boy;
    }
}
```

这里就直接把测试代码写到主方法中了，方便启动SpringBoot直接能看到结果

```java
@SpringBootApplication
public class Springboot2StudyApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(Springboot2StudyApplication.class, args);
        String[] beanNames = run.getBeanDefinitionNames(); //获取所有注入到容器的组件名字
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
        MyConfiguration bean = run.getBean(MyConfiguration.class);
        Boy boy1 = bean.getBoy();
        Boy boy2 = bean.getBoy();
        System.out.println( "两次调用的对象是否相等？" +(boy1 == boy2));
    }
}
```

当我设置proxyBeanMethods = true时，运行结果为true,显然两次获取的对象是相同的
<img src="https://img-blog.csdnimg.cn/20210119152805774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:33%;" />
当我设置proxyBeanMethods = false时，运行结果为false,两次获取的对象并不同
<img src="https://img-blog.csdnimg.cn/20210119153056826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTE0OTYxNA==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:33%;" />

Boy类和Girl类是关联的，Girl类中有Boy类型的boyfriend，我们验证下proxyBeanMethods参数对关联组件之间的影响
在config配置类中添加getGirl组件并注入容器中

```java
@Configuration(proxyBeanMethods = true)
public class MyConfiguration {


    @Bean
    public Boy getBoy(){
        Boy boy = new Boy("Jerry",18);
        return boy;
    }
    @Bean
    public Girl getGirl(){
        Girl girl = new Girl();
        girl.setName("Susan");
        girl.setAge(18);
        girl.setBoyfriend(getBoy());
        return girl;
    }
}
```

依然在主启动类中测试

```java
@SpringBootApplication
public class Springboot2StudyApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(Springboot2StudyApplication.class, args);
        MyConfiguration bean = run.getBean(MyConfiguration.class);

        Boy boy = bean.getBoy();
        Girl girl = bean.getGirl();
        System.out.println("该女生有只有一个男朋友吗？" + (girl.getBoyfriend() == boy));
    }
}
```

运行结果：
当我设置proxyBeanMethods = true时，运行结果为true,说明获取到的girl实例中的boyfriend参数和获取到的boy实例是相同的
<img src="https://img-blog.csdnimg.cn/20210119155522758.png" alt="在这里插入图片描述" style="zoom:50%;" />

当我设置proxyBeanMethods = false时，运行结果为false,说明获取到的girl实例中的boyfriend参数和获取到的boy实例不同，该女生居然有两个男朋友，渣女一枚鉴定完毕！~
<img src="https://img-blog.csdnimg.cn/20210119155632666.png" alt="在这里插入图片描述" style="zoom:50%;" />

**3.总结**

- Full 全模式，proxyBeanMethods默认是true：使用代理，也就是说该配置类会被代理，直接从IOC容器之中取得bean对象，不会创建新的对象。SpringBoot总会检查这个组件是否在容器中是否存在，保持组件的单实例
- Lite 轻量级模式，proxyBeanMethods设置为false：每次调用@Bean标注的方法获取到的对象是一个新的bean对象,和之前从IOC容器中获取的不一样，SpringBoot会跳过检查这个组件是否在容器中是否存在，保持组件的多实例



在单实例Bean中，容器创建的同时bean就已经被创建好了

但是在多实例Bean中,Bean的创建要随着对象的获取而创建



## 2、@Bean、@Component、@Controller、@Service、@Repository



@Bean: 写在方法上，向容器中注入组件，默认组件名称为方法名，如果想更改组件名称，则可以使用@Bean("组件名")



这几个注解之前就经常用了，都是用来往容器中添加组件的，**其中**

@Component、@Controller、@Service、@Repository这四大注解作用基本相同，只是用在不同的应用层上，分别是普通层，controller层，service层，dao层



## 3、@ComponentScan、@Import、工厂Bean

这两个注解都是向容器中导入组件

### @ComponentScan包扫描

@ComponentScan 的作用就是根据定义的路径扫描包内的类，把符合扫描规则的类装配到spring容器中。

在SpringBoot中，@ComponentScan如果默认不写就是扫描主配置类所在的包及其子包

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<? extends BeanNameGenerator> nameGenerator() default BeanNameGenerator.class;

    Class<? extends ScopeMetadataResolver> scopeResolver() default AnnotationScopeMetadataResolver.class;

    ScopedProxyMode scopedProxy() default ScopedProxyMode.DEFAULT;

    String resourcePattern() default "**/*.class";

    boolean useDefaultFilters() default true;

    ComponentScan.Filter[] includeFilters() default {};

    ComponentScan.Filter[] excludeFilters() default {};

    boolean lazyInit() default false;

    @Retention(RetentionPolicy.RUNTIME)
    @Target({})
    public @interface Filter {
        FilterType type() default FilterType.ANNOTATION;

        @AliasFor("classes")
        Class<?>[] value() default {};

        @AliasFor("value")
        Class<?>[] classes() default {};

        String[] pattern() default {};
    }
}
```

参数详解

- basePackages与value: 用于指定包的路径，进行扫描

- basePackageClasses: 用于指定某个类的包的路径进行扫描

- nameGenerator: bean的名称的生成器

- useDefaultFilters: 是否使用默认规则，是否开启对@Component，@Repository，@Service，@Controller的类进行检测。true开启，false关闭

- includeFilters: 指定扫描时包含的过滤条件(设置指定的过滤器)

  举例：

  ```java
  @ComponentScan(includeFilters = {useDefaultFilters =false,
                                   @ComponentScan.Filter(type = FilterType.ANNOTATION,classe{Controller.class})})
  ```

  首先关闭默认规则，设置过滤条件：只扫描Controller注解

  

- excludeFilters: 指定扫描时排除的过滤条件，用法和includeFilters一样（排除指定的过滤器）

  - FilterType.ANNOTATION：按照指定的注解类进行过滤

  - FilterType.ASSIGNABLE_TYPE：按照指定的类进行过滤

  - FilterType.ASPECTJ：使用ASPECTJ表达式

  - FilterType.REGEX：正则

  - FilterType.CUSTOM：自定义规则

    

### @Import

@Import:给容器中自动导入指定类型的组件、默认组件的名字就是全类名

- 注册方式1：普通导入，此方法最常用，很简单，就是在@Import注解中写入相应的类.class就会导入哪个类到组件中

```java
@SpringBootApplication()
@Import(PatternLayoutEncoder.class) //向容器中指定导入这个类
public class Springboot2StudyApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(Springboot2StudyApplication.class, args);
        System.out.println(run.getBean(PatternLayoutEncoder.class));
    }
```

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210119213300281.png" alt="image-20210119213300281" style="zoom:50%;" />

- 注册方法2：批量导入组件

  1. 我们事先写好一些类

  

  <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120110159710.png" alt="image-20210120110159710" style="zoom:50%;" />

  1. 自定义ImportSelector

     ```java
     public class MyImportSelector implements ImportSelector {
         @Override
         public String[] selectImports(AnnotationMetadata annotationMetadata) {
             return new String[]{"com.zlq.test.Test2","com.zlq.test.Test3"};
         }
     }
     ```

  2. 测试

     ```java
     @Import({Test1.class, MyImportSelector.class})
     ```

     

     成功导入组件<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120110448235.png" alt="image-20210120110448235" style="zoom:50%;" />

     注：组件中不会导入MyImportSelector类，即实现过ImportSelector的类

     

- 注册方法3：使用ImportBeanDefinitionRegistrar手动导入

  1. 我们事先写好一些类

     <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120112845813.png" alt="image-20210120112845813" style="zoom:50%;" />

  2. 自定义MyImportBeanDefinitionRegistrar

     ```java
     public class MyRegisterBeanDefinition implements ImportBeanDefinitionRegistrar {
         @Override
         public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
             //判断容器中是否有Test2组件
             boolean definition1 = registry.containsBeanDefinition("com.zlq.test.Test2");
             //判断容器中是否有Test3组件
             boolean definition2 = registry.containsBeanDefinition("com.zlq.test.Test3");
     
             if (definition1 && definition2){
                 //指定Bean的注册信息
                 RootBeanDefinition beanDefinition = new RootBeanDefinition(Test4.class);
                 //指定完并注册到容器中
                 registry.registerBeanDefinition("test4",beanDefinition);
             }
         }
     }
     ```

  3. 在主启动类中测试

     ```java
     @Import({Test1.class, MyImportSelector.class, MyRegisterBeanDefinition.class})
     ```

  

  成功导入组件<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120113228168.png" alt="image-20210120113228168" style="zoom:50%;" />

注：组件中也不会导入MyRegisterBeanDefinition类，即实现过ImportBeanDefinitionRegistrar的类



### 工厂Bean

如果我们想通过工厂Bean的方式创建实例做法如下

1. 编写一个类实现FactoryBean<>

   ```java
   public class MyFactoryBean implements FactoryBean<Dog> {
       @Override
       public Dog getObject() throws Exception {
           System.out.println("getObject new Dog------------");
           return new Dog();
       }
   
       @Override
       public Class<?> getObjectType() {
           return Dog.class;
       }
   
       @Override
       public boolean isSingleton() {  //是否为单实例
           return true;
       }
   }
   ```

   2. 将我们编写的继承FactoryBean的类注入到容器

      ```java
       @Bean
          public MyFactoryBean myFactoryBean(){
              return  new MyFactoryBean();
          }
      ```

测试：

```java
        Object bean1 =  run.getBean("myFactoryBean");
        Object bean2 =  run.getBean("myFactoryBean");
        System.out.println(bean1 == bean2);
        System.out.println("MyFactoryBean的类型：" + bean1.getClass());
```

得到bean1的类型是Dog，使用了单实例，两次获取的对象都相等

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120151714662.png" alt="image-20210120151714662" style="zoom:50%;" />

如果我们想获取的bean非的是myFactoryBean，我们可以在getBean( );参数内加&

 run.getBean("&myFactoryBean");即可

```java
Object bean1 =  run.getBean("&myFactoryBean");
Object bean2 =  run.getBean("&myFactoryBean");
System.out.println(bean1 == bean2);
System.out.println("MyFactoryBean的类型：" + bean1.getClass());
```

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120152056257.png" alt="image-20210120152056257" style="zoom:50%;" />





## 4、@Conditional

条件装配：满足Conditional指定的条件，则进行组件注入

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnMissingBean(name = "myConfiguration")
public class MyConfiguration {

    @Bean
    public Boy getBoy(){
        Boy boy = new Boy("Jerry",18);
        return boy;
    }
```



- @ConditionalOnBean:满足条件时候进行组件的注入

  例如在配置类中标入@ConditionalOnBean(User.class)意为：组件中有User类,我们配置类中的下面的注入才会生效，组件中没有User类，我们配置类中的下面的注入不会生效



@ConditionalOnMissingBean 不满足条件时候进行组件的注入

例如在配置类中标入@ConditionalOnMissingBean(User.class)意为：组件中有User类，我们配置类中的下面的注入不会生效，当组件中没有User类时,我们配置类中的下面的注入才会生效



总结：@ConditionalOnBean和@ConditionalOnMissingBean条件是反的



## 5.原生配置文件引入@ImportResource

我们在使用spring框架时用web.xml的方式向容器中注入组件的，如果在Springboot中我们也想用web.xml方式向容器中注入组件，就要用@ImportResource注解来将写过的web.xml导入进来，当然，傻叉才会这么做，毕竟springboot中用注解的方式来注入组件它不香吗，除非老项目导入第三方组件才会使用到它

首先我们在resource中写一个xml文件bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="user1" class="com.zlq.bean.User">
        <property name="name" value="Larry"></property>
        <property name="age" value="18"></property>
    </bean>

    <bean id="user2" class="com.zlq.bean.User">
        <property name="name" value="Susan"></property>
        <property name="age" value="18"></property>
    </bean>
</beans>
```

在主启动中测试

```java
@ImportResource("classpath:beans.xml")


======================测试=================
        boolean user1 = run.containsBean("user1");
        boolean user2 = run.containsBean("user2");
        System.out.println("user1：" + user1);//true
        System.out.println("user2：" + user2);//true
```



## 6、配置绑定

### @ConfigurationProperties（perfix = " ")

perfix:前缀,与配置文件中的哪个属性进行绑定

在学习yaml中会提及到



## 7、@EnableConfigurationProperties

@EnableConfigurationProperties注解的作用：让使用 @ConfigurationProperties 注解的类生效，并把使用 @ConfigurationProperties 注解的类注入到容器中



## 8、 使用@Scope注解设置组件的作用域

Spring容器中的组件默认是单例的，在Spring启动时就会实例化并初始化这些对象，并将其放到Spring容器中，之后，每次获取对象时，直接从Spring容器中获取，而不再创建对象。如果每次从Spring容器中获取对象时都要创建一个新的实例对象，那么该如何处理呢？此时就需要使用@Scope注解来设置组件的作用域了。

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120201126535.png" alt="image-20210120201126535" style="zoom:50%;" />

# Bean的生命周期



## Bean的生命周期的介绍

bean的生命周期指的是bean从创建到初始化，经过一系列中间流程，到最终销毁的过程。在Spring中，bean的生命周期是由Spring容器来管理的。

在Spring中，我们可以自己来自定义bean的初始化和销毁的方法。我们自定义指定bean的初始化和销毁方法之后，当容器在bean进行到当前生命周期的阶段时，会自动调用我们自定义的初始化和销毁方法



## 如何自定义初始化和销毁方法

###  方法1、使用配置文件的方式

通过配置文件的方式是最原始的方法，在初学Spring的时候已经接触过,只要在配置文件中指定我们写过的初始化方法和销毁方法即可

```xml
<bean id="User" class="com.zlq.bean.User" init-method="init" destroy-method="destroy">
        <property name="name" value="LiQun"></property>
        <property name="age" value="18"></property>
    </bean>
```

```java
@Data
@AllArgsConstructor
@Getter
@Setter
@ToString
public class User {
    private String name;
    private int age;

    public User(){
        System.out.println("使用了User的无参构造器~~~~~~");
    }
    public void init(){
        System.out.println("user正在初始化，init user...........");
    }

    public void destroy() {
        System.out.println("user正在销毁，destroy user...");
    }
}

```

在配置类中导入beans.xml

@ImportResource("classpath:beans.xml")

测试：

```java
  @Test
    void contextLoads() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfiguration.class);
        System.out.println("容器创建完成！~~~");
        context.close();
    }
```



![image-20210120171255430](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120171255430.png)



###  方法2、使用注解的方式

我们可以使用注解的方式指定初始化和销毁的方法

在配置类中的@Bean注解中声明初始化和销毁的方法即可

```java
@Bean(initMethod = "init",destroyMethod = "destroy")
public User getUser(){
    return new User();
}
```

![image-20210120171258294](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120171258294.png)



**指定初始化和销毁方法的使用场景**

一个典型的使用场景就是对于数据源的管理。例如，在配置数据源时，在初始化的时候，会对很多的数据源的属性进行赋值操作；在销毁的时候，我们需要对数据源的连接等信息进行关闭和清理。这个时候，我们就可以在自定义的初始化和销毁方法中来做这些事情了！



**初始化和销毁方法的调用时机**

- bean对象的初始化方法在对象创建完成时调用，如果对象中存在一些属性，并且这些属性也都赋好值之后，那么就会调用bean的初始化方法。对于单实例bean来说，在Spring容器创建完成后，Spring容器会自动调用bean的初始化方法；对于多实例bean来说，在每次获取bean对象的时候，调用bean的初始化方法。

- bean对象的销毁方法调用的时机：

  	- 单实例Bean：容器关闭即销毁
   
    	![image-20210120201716961](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120201716961.png)
    	
   - 多实例bean：需要手动调用bean的销毁方法。对于多实例bean来说，Spring不会去管的

   ![image-20210120201612178](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120201612178.png)

  

**对于单实例bean来说，会在Spring容器启动的时候创建对象；对于多实例bean来说，会在每次获取bean的时候创建对象。**



### 方法3、使用InitializingBean和DisposableBean来管理bean的生命周期

1. 创建Person类实现InitializingBean和DisposableBean

   ```java
   @Component
   public class Person implements InitializingBean, DisposableBean {
       public Person(){
           System.out.println("调用Person的无参构造器");
       }
     
      //会在bean创建完成，并且属性都赋好值以后进行调用
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("person afterPropertiesSet------------");
       }
     
       //会在容器关闭时调动
       @Override
       public void destroy() throws Exception {
           System.out.println("Person destroy-----------------");
       }
   }
   ```

2. 配置类中

   ```java
   @Configuration()
   @ComponentScan("com.zlq.bean")     
   public class MyConfiguration {
   
   }
   ```

   <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120210007731.png" alt="image-20210120210007731" style="zoom:30%;" />



如果改为多实例

在Person类上加@scope("prototype")

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120210132988.png" alt="image-20210120210132988" style="zoom:30%;" />



从输出的结果信息中可以看出，在多实例bean情况下，Spring不会自动调用bean的销毁方法destroy( )。





**原理**

- **1、InitializingBean接口**

  Spring中提供了一个InitializingBean接口，该接口为bean提供了属性初始化后的处理方法，它只包括afterPropertiesSet方法，凡是继承该接口的类，在bean的属性初始化后都会执行该方法。InitializingBean接口的源码如下所示。

  <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120210608889.png" alt="image-20210120210608889" style="zoom:39%;" />

  不难看出，afterPropertiesSet方法是在属性赋值好以后调用

  <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120211849289.png" alt="image-20210120211849289" style="zoom:39%;" />

  分析上述代码后，我们可以初步得出如下信息：

  1. Spring为bean提供了两种初始化的方式，实现InitializingBean接口（也就是要实现该接口中的afterPropertiesSet方法），或者在配置文件或@Bean注解中通过init-method来指定，两种方式可以同时使用。

  2. 实现InitializingBean接口是直接调用afterPropertiesSet()方法，与通过反射调用init-method指定的方法相比，效率相对来说要高点。但是init-method方式消除了对Spring的依赖。

  3. 如果调用afterPropertiesSet方法时出错，那么就不会调用init-method指定的方法了。

     

  也就是说Spring为bean提供了两种初始化的方式，第一种方式是实现InitializingBean接口（也就是要实现该接口中的afterPropertiesSet方法），第二种方式是在配置文件或@Bean注解中通过init-method来指定，这两种方式可以同时使用，同时使用先调用afterPropertiesSet方法，后执行init-method指定的方法

- **2、DisposableBean接口**

  实现DisposableBean接口的bean在销毁前，Spring将会调用DisposableBean接口的destroy()方法。也就是说我们可以实现DisposableBean这个接口来定义销毁逻辑。<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210120212326935.png" alt="image-20210120212326935" style="zoom:50%;" />

在bean生命周期结束前调用destroy()方法做一些收尾工作，亦可以使用destroy-method。前者与Spring耦合高，使用**类型强转.方法名()**，效率高；后者耦合低，使用反射，效率相对来说较低。



DisposableBean接口分析：

多实例bean的生命周期不归Spring容器来管理，这里的DisposableBean接口中的方法是由Spring容器来调用的，所以如果一个多实例bean实现了DisposableBean接口是没有啥意义的，因为相应的方法根本不会被调用，当然了，在XML配置文件中指定了destroy方法，也是没有任何意义的。所以，在多实例bean情况下，Spring是不会自动调用bean的销毁方法的。



### 方法4、使用PostConstruct注解和@PreDestroy注解



**PostConstruct注解**

@PostConstruct注解不是Spring提供的，而是Java自己的注解：JSR-250规范。源码如下所示。

```java
package javax.annotation;

@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}
```

从源码可以看出，**@PostConstruct注解是Java中的注解，并不是Spring提供的注解。**

@PostConstruct注解被用来修饰一个非静态的void()方法。被@PostConstruct注解修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。被@PostConstruct注解修饰的方法通常在构造函数之后，init()方法之前执行。

通常我们是会在Spring框架中使用到@PostConstruct注解的，该注解的方法在整个bean初始化中的执行顺序如下：

> Constructor（构造方法）→@Autowired（依赖注入）→@PostConstruct（注释的方法）



**@PreDestroy注解**
@PreDestroy注解同样是Java提供的，它也是JSR-250规范里面定义的一个注解。看下它的源码，如下所示。

```java
package javax.annotation;

@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}
```

被@PreDestroy注解修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreDestroy注解修饰的方法会在destroy()方法之后，Servlet被彻底卸载之前执行。执行顺序如下所示：

> 调用destroy()方法→@PreDestroy→destroy()方法→bean销毁





最终执行顺序

![image-20210121102558376](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121102558376.png)



# 关于BeanPostProcessor后置处理器



## 介绍



**源码如下**：<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121092649236.png" alt="image-20210121092649236" style="zoom:50%;" />



BeanPostProcessor是一个接口，其中有两个方法，即postProcessBeforeInitialization和postProcessAfterInitialization这两个方法，这两个方法分别是在Spring容器中的bean初始化前后执行。当容器有多个不同的BeanPostProcessor的实现类时，会按照它们在容器中注册的顺序执行。

当然，对于自定义的BeanPostProcessor实现类，我们也可以让其实现Ordered接口自定义排序。

因此我们可以在每个bean对象初始化前后，加上自己的逻辑。

## 实操



```java
@Component
public class MyBeanPostProcessor  implements BeanPostProcessor , Ordered {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..." + beanName + "=>" + bean);
        return bean;
    }

    @Override
    public int getOrder() {
        return 1;
    }
}
```

![image-20210121094505108](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121094505108.png)

**BeanPostProcessor后置处理器作用**

后置处理器可用于bean对象初始化前后进行逻辑增强。我们可以自定义BeanPostProcessor接口的实现类，在其中写入我们自定义的逻辑。Spring还提供了BeanPostProcessor接口的很多实现类，例如AutowiredAnnotationBeanPostProcessor用于@Autowired注解的实现，AnnotationAwareAspectJAutoProxyCreator用于Spring AOP的动态代理等等。

下面我会以AnnotationAwareAspectJAutoProxyCreator为例，简单说明一下后置处理器是怎样工作的。

我们都知道spring AOP的实现原理是动态代理，最终放入容器的是代理类的对象，而不是bean本身的对象，那么Spring是什么时候做到这一步的呢？就是在AnnotationAwareAspectJAutoProxyCreator后置处理器的postProcessAfterInitialization方法中，即bean对象初始化完成之后，后置处理器会判断该bean是否注册了切面，若是，则生成代理对象注入到容器中。这一部分的关键代码是在哪儿呢？我们定位到AbstractAutoProxyCreator抽象类中的postProcessAfterInitialization方法处便能看到了，如下所示。

![image-20210121094918851](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210121094918851.png)

## BeanPostProcessor的底层原理。



### bean的初始化和销毁方式

我们知道BeanPostProcessor的postProcessBeforeInitialization()方法是在bean的初始化之前被调用；而postProcessAfterInitialization()方法是在bean初始化的之后被调用。



对Bean的生命周期的控制可以有如下四种方式

- （一）通过@Bean指定init-method和destroy-method
- （二）通过让bean实现InitializingBean和DisposableBean这俩接口
- （三）使用JSR-250规范里面定义的@PostConstruct和@PreDestroy这俩注解
- （四）通过让bean实现BeanPostProcessor接口



通过以上这四种方式，我们就可以对bean的整个生命周期进行控制：

- bean的实例化：调用bean的构造方法，我们可以在bean的无参构造方法中执行相应的逻辑。
- bean的初始化：在初始化时，可以通过BeanPostProcessor的postProcessBeforeInitialization()方法和postProcessAfterInitialization()方法进行拦截，执行自定义的逻辑；通过@PostConstruct注解、InitializingBean和init-method来指定bean初始化前后执行的方法，在该方法中咱们可以执行自定义的逻辑。
- bean的销毁：可以通过@PreDestroy注解、DisposableBean和destroy-method来指定bean在销毁前执行的方法，在该方法中咱们可以执行自定义的逻辑。

所以，通过上述四种方式，我们可以控制Spring中bean的整个生命周期。


