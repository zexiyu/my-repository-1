答题技巧：

总：当前问题回答的是那些具体的点

分：以1，2，3，4，5的方式分细节取描述相关的知识点，如果有哪些点不清楚，直接忽略过去

​		突出一些技术名词（核心概念，接口，类，关键方法）

​		避重就轻：没有重点

一个问题能占用面试官多少时间？问的越多可能露馅越多

当面试官问到一个你熟悉的点的时候，一定要尽量拖时间

# 1.谈谈Spring IOC的理解，原理与实现?

**总：**

- 控制反转：理论思想，原来的对象是由使用者来进行控制，有了spring之后，可以把整个对象交给spring来帮我们进行管理
- DI：依赖注入，就是把对应的属性的值注入到具体的对象中，@Autowired，通过populateBean完成属性值的注入
- 容器：存储对象，使用map结构来存储，在spring中一般存在三级缓存，singletonObjects存放完整的bean对象，整个bean的生命周期，从创建到使用到销毁的过程全部都是由容器来管理（bean的生命周期）

**分：**

1、一般聊ioc容器的时候要涉及到容器的创建过程（beanFactory,DefaultListableBeanFactory）,向bean工厂中设置一些参数（BeanPostProcessor,Aware接口的子类）等等属性

2、加载解析bean对象，准备要创建的bean对象的定义对象beanDefinition,(xml或者注解的解析过程)

3、beanFactoryPostProcessor的处理，此处是扩展点，PlaceHolderConfigureSupport,ConfigurationClassPostProcessor

4、BeanPostProcessor的注册功能，方便后续对bean对象完成具体的扩展功能

5、通过反射的方式讲BeanDefinition对象实例化成具体的bean对象，

6、bean对象的初始化过程（填充属性，调用aware子类的方法，调用BeanPostProcessor前置处理方法，调用init-mehtod方法，调用BeanPostProcessor的后置处理方法）

7、生成完整的bean对象，通过getBean方法可以直接获取

8、销毁过程

面试官，这是我对ioc的整体理解，包含了一些详细的处理过程，您看一下有什么问题，可以指点我一下（允许你把整个流程说完）

您由什么想问的？

​			老师，我没看过源码怎么办？



简化版：

​		具体的细节我记不太清了，但是spring中的bean都是通过反射的方式生成的，同时其中包含了很多的扩展点，比如最常用的对BeanFactory的扩展，对bean的扩展（对占位符的处理），我们在公司对这方面的使用是比较多的，除此之外，ioc中最核心的也就是填充具体bean的属性，和生命周期（背一下）。

# 2.谈一下spring IOC的底层实现

底层实现：工作原理，过程，数据结构，流程，设计模式，设计思想

你对他的理解和你了解过的实现过程

反射，工厂（BeanFactory），设计模式（会的说，不会的不说），记住几个关键的方法

createBeanFactory，getBean,doGetBean,createBean,doCreateBean,

createBeanInstance(getDeclaredConstructor,newinstance),populateBean,initializingBean

(doCreateBean和createBean的区别：加do的方法时实际执行的方法，没加do的方法实际上是在前面套了一层)

1、先通过createBeanFactory创建出一个Bean工厂（DefaultListableBeanFactory）

2、开始循环创建对象，因为容器中的bean默认都是单例的，所以优先通过getBean,doGetBean从容器中查找，找不到的话，

3、通过createBean,doCreateBean方法，以反射的方式创建对象，一般情况下使用的是无参的构造方法（getDeclaredConstructor，newInstance）

4、进行对象的属性填充populateBean

5、进行其他的初始化操作（initializingBean）

![image-20210814150508660](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210814150508660.png)



简化版：

Spring IOC容器的初始化简单的可以分为三个过程：

-  第一个过程是Resource资源定位。这个Resouce指的是BeanDefinition的资源定位。这个过程就是容器找数据的过程，就像水桶装水需要先找到水一样。
-  第二个过程是BeanDefinition的载入过程。这个载入过程是把用户定义好的Bean表示成Ioc容器内部的数据结构，而这个容器内部的数据结构就是BeanDefition。
-  第三个过程是向IOC容器注册这些BeanDefinition的过程，这个过程就是将前面的BeanDefition保存到HashMap中的过程。





# 3.描述一下bean的生命周期 ？

背图：记住图中的流程

在表述的时候不要只说图中有的关键点，要学会扩展描述

1、实例化bean：反射的方式生成对象

2、填充bean的属性：populateBean(),循环依赖的问题（三级缓存）

3、调用aware接口相关的方法：invokeAwareMethod(完成BeanName,BeanFactory,BeanClassLoader对象的属性设置)

4、调用BeanPostProcessor中的前置处理方法：使用比较多的有（ApplicationContextPostProcessor,设置ApplicationContext,Environment,ResourceLoader,EmbeddValueResolver等对象）

5、调用initmethod方法：invokeInitmethod(),判断是否实现了initializingBean接口，如果有，调用afterPropertiesSet方法，没有就不调用

6、调用BeanPostProcessor的后置处理方法：spring的aop就是在此处实现的，AbstractAutoProxyCreator

​		注册Destuction相关的回调接口：钩子函数

7、获取到完整的对象，可以通过getBean的方式来进行对象的获取

8、销毁流程，1；判断是否实现了DispoableBean接口，2，调用destroyMethod方法

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210813221405462.png" alt="image-20210813221405462" style="zoom:50%;" />

# 4.Spring 是如何解决循环依赖的问题的？

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210813225915591.png" alt="image-20210813225915591" style="zoom:50%;" />

三级缓存，提前暴露对象，aop

总：什么是循环依赖问题，A依赖B,B依赖A

分：先说明bean的创建过程：实例化，初始化（填充属性）

​		1、先创建A对象，实例化A对象，此时A对象中的b属性为空，填充属性b

​		2、从容器中查找B对象，如果找到了，直接赋值不存在循环依赖问题（不通），找不到直接创建B对象

​		3、实例化B对象，此时B对象中的a属性为空，填充属性a

​		4、从容器中查找A对象，找不到，直接创建

​		形成闭环的原因

​		此时，如果仔细琢磨的话，会发现A对象是存在的，只不过此时的A对象不是一个完整的状态，只完成了实例化但是未完成初始化，如果在程序调用过程中，拥有了某个对象的引用，能否在后期给他完成赋值操作，可以优先把非完整状态的对象优先赋值，等待后续操作来完成赋值，**相当于提前暴露了某个不完整对象的引用**，所以解决问题的核心在于**实例化和初始化分开操作，这也是解决循环依赖问题的关键，**

​		当所有的对象都完成实例化和初始化操作之后，还要把完整对象放到容器中，此时在容器中存在对象的几个状态，完成实例化=但未完成初始化，完整状态，因为都在容器中，所以要使用不同的map结构来进行存储，此时就有了一级缓存和二级缓存，如果一级缓存中有了，那么二级缓存中就不会存在同名的对象，**因为他们的查找顺序是1级，2级，3级 这样的方式来查找的**。



**结论：一级缓存中放的是完整对象（成品），二级缓存中放的是非完整对象（半成品）**



​		**为什么需要三级缓存？**三级缓存的value类型是ObjectFactory,是一个函数式接口，存在的意义是保证在整个容器的运行过程中同名的bean对象只能有一个。

​		如果一个对象需要被代理，或者说需要生成代理对象，那么要不要优先生成一个普通对象（被代理对象）？要

​		普通对象和代理对象是不能同时出现在容器中的，因此当一个对象需要被代理的时候，就要使用代理对象覆盖掉之前的普通对象，在实际的调用过程中，是没有办法确定什么时候对象被使用，所以就要求当某个对象被调用的时候，优先判断此对象是否需要被代理，类似于一种回调机制的实现，因此传入lambda表达式的时候，可以通过lambda表达式来执行对象的覆盖过程，getEarlyBeanReference()

​		因此，所有的bean对象在创建的时候都要优先放到三级缓存中，在后续的使用过程中，如果需要被代理则返回代理对象，如果不需要被代理，则直接返回普通对象

![image-20210814170434269](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210814170434269.png)

# 4.1缓存的放置时间和删除时间

​		三级缓存：createBeanInstance之后：addSingletonFactory

​		二级缓存：第一次从三级缓存确定对象是代理对象还是普通对象的时候，同时删除三级缓存 getSingleton

​		一级缓存：生成完整对象之后放到一级缓存，删除二三级缓存:addSingleton

# 5.BeanFactory与FactoryBean有什么区别？

相同点：都是用来创建bean对象的

不同点：使用BeanFactory创建对象的时候，必须要遵循严格的生命周期流程，太复杂了，，如果想要简单的自定义某个对象的创建，同时创建完成的对象想交给spring来管理，那么就需要实现FactroyBean接口了。一个是复杂的创建流程，一个是简化版的创建流程

​			isSingleton:是否是单例对象

​			getObjectType:获取返回对象的类型

​			getObject:自定义创建对象的过程(new，反射，动态代理)





\- **BeanFactory**是个Factory，也就是IOC容器或对象工厂，在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的，提供了实例化对象和拿对象的功能。

使用场景：

\- 从Ioc容器中获取Bean(byName or byType)

\- 检索Ioc容器中是否包含指定的Bean

\- 判断Bean是否为单例

\- **FactoryBean**是个Bean，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。

使用场景

\- ProxyFactoryBean: 如果想要简单的自定义某个对象的创建，同时创建完成的对象想交给spring来管理



# 6.Spring中用到的设计模式? 

- 代理模式——在AOP中被用的比较多(动态代理）。

-  单例模式——在spring配置文件中定义的bean默认为单例模式。 

- 模板方法——用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。

- 工厂模式——BeanFactory用来创建对象的实例。

- 适配器——spring aop、Adapter

- 装饰器——BeanWrapper

- 观察者——listener，event，multicast

- 回调——Spring Aware回调接口

- 责任链模式——使用aop的时候会先生成一个拦截器链

  

# 7.Spring的AOP的底层实现原理? 

动态代理

aop是ioc的一个扩展功能，先有的ioc，再有的aop，只是在ioc的整个流程中新增的一个扩展点而已：BeanPostProcessor

总：aop概念，应用场景，动态代理

分：

​		bean的创建过程中有一个步骤可以对bean进行扩展实现，aop本身就是一个扩展功能，所以在BeanPostProcessor的后置处理方法中来进行实现

​		1、代理对象的创建过程（advice，切面，切点）

​		2、通过jdk或者cglib的方式来生成代理对象

​		3、在执行方法调用的时候，会调用到生成的字节码文件中，直接回找到DynamicAdvisoredInterceptor类中的intercept方法，从此方法开始执行

​		4、根据之前定义好的通知来生成拦截器链

​		5、从拦截器链中依次获取每一个通知开始进行执行，在执行过程中，为了方便找到下一个通知是哪个，会有一个CglibMethodInvocation的对象，找的时候是从-1的位置一次开始查找并且执行的。

# 8.Spring的事务是如何回滚的?

​		spring的事务管理是如何实现的？

​		总：spring的事务是由aop来实现的，首先要生成具体的代理对象，然后按照aop的整套流程来执行具体的操作逻辑，正常情况下要通过通知来完成核心功能，但是事务不是通过通知来实现的，而是通过一个TransactionInterceptor来实现的，然后调用invoke来实现具体的逻辑

​		分：1、先做准备工作，解析各个方法上事务相关的属性，根据具体的属性来判断是否开始新事务

​				2、当需要开启的时候，获取数据库连接，关闭自动提交功能，开启事务

​				3、执行具体的sql逻辑操作

​				4、在操作过程中，如果执行失败了，那么会通过completeTransactionAfterThrowing来完成事务的回滚操作，回滚的具体逻辑是通过doRollBack方法来实现的，实现的时候也是要先获取连接对象，通过连接对象来回滚

​				5、如果执行过程中，没有任何意外情况的发生，那么通过commitTransactionAfterReturning来完成事务的提交操作，提交的具体逻辑是通过doCommit方法来实现的，实现的时候也是要获取连接，通过连接对象来提交

​				6、当事务执行完毕之后需要清除相关的事务信息cleanupTransactionInfo

如果想要聊的更加细致的话，需要知道TransactionInfo,TransactionStatus,

# 9.谈一下spring事务传播？

​			传播特性有几种？7种

​			Required,Requires_new,nested,Support,Not_Support,Never,Mandatory

​			某一个事务嵌套另一个事务的时候怎么办？

​			A方法调用B方法，AB方法都有事务，并且传播特性不同，那么A如果有异常，B怎么办，B如果有异常，A怎么办？





- PROPAGATION_REQUIRED -- 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
- PROPAGATION_REQUIRES_NEW -- 新建事务，如果当前存在事务，把当前事务挂起。
- PROPAGATION_SUPPORTS -- 支持当前事务，如果当前没有事务，就以非事务方式执行。
- PROPAGATION_MANDATORY -- 支持当前事务，如果当前没有事务，就抛出异常。
- PROPAGATION_NOT_SUPPORTED -- 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER -- 以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED -- 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。


--------

​			总：事务的传播特性指的是不同方法的嵌套调用过程中，事务应该如何进行处理，是用同一个事务还是不同的事务，当出现异常的时候会回滚还是提交，两个方法之间的相关影响，在日常工作中，使用比较多的是required，Requires_new,nested

​			分：1、先说事务的不同分类，可以分为三类：支持当前事务，不支持当前事务，嵌套事务

​					2、如果外层方法是required，内层方法是，required,requires_new,nested

​					3、如果外层方法是requires_new，内层方法是，required,requires_new,nested

​					4、如果外层方法是nested，内层方法是，required,requires_new,nested

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210814102047039.png" alt="image-20210814102047039" style="zoom:50%;" />	<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210814102104323.png" alt="image-20210814102104323" style="zoom:50%;" />

------

**找工作：**

**1、面试之前一定要调整好心态，不管你会多少东西，干就完了，出去面试就一个心态，老子天下第一，让自己超常发挥**

**2、得失心不要太重，全中国企业很多，好公司也有很多，没必要在一棵树上吊死，你可以有心仪的公司，留到最后，等你准备充分再去**

**3、找工作永远不可能准备好，很多同学怂，心态不好，不敢出去面试，我要准备，先按照你的技术储备取尝试一些公司（我就是来试水的）面试回来之后做总结，做好准备，不断总结，复盘，这样才能成长**

**4、希望大家保持好信息互通，乐于分享**



# 10.SpringBoot自动装配原理是什么

表达的具体思路是：总－分－总

- springboot自动装配是什么，解决了什么问题

  springboot开箱即用，大量的配置在我们启动springboot后自动导入。以前spring框架需要我们大量配置参数，现在springboot帮我们自动配置，无需人工配置。大大简化了开发

- 自动装配实现的原理：

  1. 当启动springboot应用程序的时候，会先创建SpringApplication的对象，在对象的构造方法中会进行某些参数的初始化工作，最主要的是判断当前应用程序的类型以及初始化器和监听器，在这个过程中会加载整个应用程序中的spring.factories文件，将文件的内容放到缓存对象中，方便后续获取。

  2. SpringApplication对象创建完成之后，开始执行run方法，来完成整个启动，启动过程中最主要的有两个方法，第一个叫做prepareContext，第二个叫做refreshContext，在这两个关键步骤中完整了自动装配的核心功能，前面的处理逻辑包含了上下文对象的创建，banner的打印，异常报告期的准备等各个准备工作，方便后续来进行调用。

  3. 在prepareContext方法中主要完成的是对上下文对象的初始化操作，包括了属性值的设置，比如环境对象，在整个过程中有一个非常重要的方法，叫做load，load主要完成一件事，将当前启动类做为一个beanDefinition注册到registry中，I方便后续在进行BeanFactoryPostProcessor调用执行的时候，找到对应的主 类，来完成＠SpringBootApplicaiton，

     ＠EnableAutoConfiguration等注解的解析工作

  4. 在refreshContext方法中会进行整个容器刷新过程，会调用中spring中的refresh方法，refresh中有13个非常关键的方法，来完成整个spring应用程序的启动，在自动装配过程中，会调用invokeBeanFactoryPostProcessor方法，在此方法中主要是对ConfigurationClassPostProcessor类的处理，这次是BFPP的子类也是BDRPP的子类，在调用的时候会先调用BDRPP中的postProcessBeanDefinitionRegistry方法，然后调用postProcessBeanFactory方法，在执postProcessBeanDefinitionRegistry的时候回解析处理各种 注解，包含＠PropertySource，＠ComponentScan，＠ComponentScans，＠Bean，＠lmport等解，最主要的是 ＠lmport注解的解析

  5. 在解析＠lmport注解的时候，会有一个getlmports的方法，从主类开始递归解析注解，把所有包含＠lmport的注解都解析到，然后在processlmport方法中对Import的类进行分类，此处主要识别的时候AutoConfigurationlmportSelect归属于ImportSelect的子类，在后续过程中会调用deferredlmportSelectorHandler中的process方法，来完整EnableAutoConfiguration的加载。

  6. 上面是我对springboot自动装配的简单理解，面试官您看一下，我回答有没有问题，帮我指点一下！

  



# 其他面试题

## 什么是Spring框架，Spring框架主要包含哪些模块

 Spring是一个开源框架，Spring是一个轻量级的Java 开发框架。它是为了解决企业应用开发的复杂性而创建的。框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为 J2EE 应用程序开发提供集成的框架。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是控制反转（IoC）和面向切面（AOP）。简单来说，Spring是一个分层的full-stack(一站式) 轻量级开源框架。

主要包含的模块：

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211001110908237.png" alt="image-20211001110908237" style="zoom:50%;" />

## Spring框架的优势



**此处要背**

 1、Spring通过DI、AOP和消除样板式代码来简化企业级Java开发（简化开发）

 2、Spring框架之外还存在一个构建在核心框架之上的庞大生态圈，它将Spring扩展到不同的领域，如Web服务、微服务、REST、移动开发以及NoSQL（庞大生态）

 3、低侵入式设计，代码的污染极低 （安全）

 4、独立于各种应用服务器，基于Spring框架的应用，可以真正实现Write Once,Run Anywhere的承诺（强适配性）

 5、Spring的IoC容器降低了业务对象替换的复杂性，提高了组件之间的解耦 （解耦）

 6、Spring的AOP允许将一些通用任务如安全、事务、日志等进行集中式处理，从而提供了更好的复用 (面向切面)

 7、Spring的ORM和DAO提供了与第三方持久层框架的的良好整合，并简化了底层的数据库访问 （强整合性）

 8、Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可自由选用Spring框架的部分或全部 （自由选择，高度开放）



## IOC和DI是什么？

**此处要背**

 本来对象的创建和维护权在我们手里，对象需要我们手动new出来，控制翻转将对象的创建和维护权完全交给外部容器

 依赖注入是指:在程序运行期间,由外部容器动态地将依赖对象注入到组件中如：一般，通过构造函数注入或者setter注入。



## 描述下Spring IOC容器的初始化过程



 Spring IOC容器的初始化简单的可以分为三个过程：

 第一个过程是Resource资源定位。这个Resouce指的是BeanDefinition的资源定位。这个过程就是容器找数据的过程，就像水桶装水需要先找到水一样。

 第二个过程是BeanDefinition的载入过程。这个载入过程是把用户定义好的Bean表示成Ioc容器内部的数据结构，而这个容器内部的数据结构就是BeanDefition。

 第三个过程是向IOC容器注册这些BeanDefinition的过程，这个过程就是将前面的BeanDefition保存到HashMap中的过程。



## BeanFactory 和 FactoryBean的区别？

\- **BeanFactory**是个Factory，也就是IOC容器或对象工厂，在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的，提供了实例化对象和拿对象的功能。

使用场景：

\- 从Ioc容器中获取Bean(byName or byType)

-r 检索Ioc容器中是否包含指定的Bean

\- 判断Bean是否为单例

\- **FactoryBean**是个Bean，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。

使用场景

\- ProxyFactoryBean

## BeanFactory和ApplicationContext的异同

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211001111034640.png" alt="image-20211001111034640" style="zoom:50%;" />

相同：

\- Spring提供了两种不同的IOC 容器，一个是BeanFactory，另外一个是ApplicationContext，它们都是Java interface，ApplicationContext继承于BeanFactory(ApplicationContext继承ListableBeanFactory。

\- 它们都可以用来配置XML属性，也支持属性的自动注入。

\- 而ListableBeanFactory继承BeanFactory)，BeanFactory 和 ApplicationContext 都提供了一种方式，使用getBean("bean name")获取bean。

不同：

\- 当你调用getBean()方法时，BeanFactory仅实例化bean，而ApplicationContext 在启动容器的时候实例化单例bean，不会等待调用getBean()方法时再实例化。

\- BeanFactory不支持国际化，即i18n，但ApplicationContext提供了对它的支持。

\- BeanFactory与ApplicationContext之间的另一个区别是能够将事件发布到注册为监听器的bean。

\- BeanFactory 的一个核心实现是XMLBeanFactory 而ApplicationContext 的一个核心实现是ClassPathXmlApplicationContext，Web容器的环境我们使用WebApplicationContext并且增加了getServletContext 方法。

\- 如果使用自动注入并使用BeanFactory，则需要使用API注册AutoWiredBeanPostProcessor，如果使用ApplicationContext，则可以使用XML进行配置。

\- 简而言之，BeanFactory提供基本的IOC和DI功能，而ApplicationContext提供高级功能，BeanFactory可用于测试和非生产使用，但ApplicationContext是功能更丰富的容器实现，应该优于BeanFactory

 



## Spring 是如何管理事务的？

 Spring事务管理主要包括3个接口，Spring的事务主要是由它们(**PlatformTransactionManager，TransactionDefinition，TransactionStatus**)三个共同完成的。

**1. PlatformTransactionManager**：事务管理器--主要用于平台相关事务的管理

主要有三个方法：

\- commit 事务提交；

\- rollback 事务回滚；

\- getTransaction 获取事务状态。

**2. TransactionDefinition**：事务定义信息--用来定义事务相关的属性，给事务管理器PlatformTransactionManager使用

这个接口有下面四个主要方法：

\- getIsolationLevel：获取隔离级别；

\- getPropagationBehavior：获取传播行为；

\- getTimeout：获取超时时间；

\- isReadOnly：是否只读（保存、更新、删除时属性变为false--可读写，查询时为true--只读）

事务管理器能够根据这个返回值进行优化，这些事务的配置信息，都可以通过配置文件进行配置。

**3. TransactionStatus**：事务具体运行状态--事务管理过程中，每个时间点事务的状态信息。

例如它的几个方法：

\- hasSavepoint()：返回这个事务内部是否包含一个保存点，

\- isCompleted()：返回该事务是否已完成，也就是说，是否已经提交或回滚

\- isNewTransaction()：判断当前事务是否是一个新事务

**声明式事务的优缺点**：

\- **优点**：不需要在业务逻辑代码中编写事务相关代码，只需要在配置文件配置或使用注解（@Transaction），这种方式没有侵入性。

\- **缺点**：声明式事务的最细粒度作用于方法上，如果像代码块也有事务需求，只能变通下，将代码块变为方法。



## bean的作用域

（1）singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。

（2）prototype：为每一个bean请求提供一个实例。

（3）request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

（4）session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。

\### 14、Spring框架中有哪些不同类型的事件

（1）上下文更新事件（ContextRefreshedEvent）：在调用ConfigurableApplicationContext 接口中的refresh()方法时被触发。

（2）上下文开始事件（ContextStartedEvent）：当容器调用ConfigurableApplicationContext的Start()方法开始/重新开始容器时触发该事件。

（3）上下文停止事件（ContextStoppedEvent）：当容器调用ConfigurableApplicationContext的Stop()方法停止容器时触发该事件。

（4）上下文关闭事件（ContextClosedEvent）：当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理的所有单例Bean都被销毁。

（5）请求处理事件（RequestHandledEvent）：在Web应用中，当一个http请求（request）结束触发该事件。

\### 15、Spring通知有哪些类型

（1）前置通知（Before advice）：在某连接点（join point）之前执行的通知，但这个通知不能阻止连接点前的执行（除非它抛出一个异常）。

（2）返回后通知（After returning advice）：在某连接点（join point）正常完成后执行的通知：例如，一个方法没有抛出任何异常，正常返回。

（3）抛出异常后通知（After throwing advice）：在方法抛出异常退出时执行的通知。

（4）后通知（After (finally) advice）：当某连接点退出的时候执行的通知（不论是正常返回还是异常退出）。

（5）环绕通知（Around Advice）：包围一个连接点（join point）的通知，如方法调用。这是最强大的一种通知类型。 环绕通知可以在方法调用前后完成自定义的行为。它也会选择是否继续执行连接点或直接返回它们自己的返回值或抛出异常来结束执行。 环绕通知是最常用的一种通知类型。



## Spring的自动装配

在spring中，对象无需自己查找或创建与其关联的其他对象，由容器负责把需要相互协作的对象引用赋予各个对象，使用autowire来配置自动装载模式。

在Spring框架xml配置中共有5种自动装配：

（1）no：默认的方式是不进行自动装配的，通过手工设置ref属性来进行装配bean。

（2）byName：通过bean的名称进行自动装配，如果一个bean的 property 与另一bean 的name 相同，就进行自动装配。

（3）byType：通过参数的数据类型进行自动装配。

（4）constructor：利用构造函数进行装配，并且构造函数的参数通过byType进行装配。

（5）autodetect：自动探测，如果有构造方法，通过 construct的方式自动装配，否则使用 byType的方式自动装配。

基于注解的方式：

使用@Autowired注解来自动装配指定的bean。在使用@Autowired注解之前需要在Spring配置文件进行配置，<context:annotation-config />。在启动spring IoC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowied、@Resource或@Inject时，就会在IoC容器自动查找需要的bean，并装配给该对象的属性。在使用@Autowired时，首先在容器中查询对应类型的bean：

如果查询结果刚好为一个，就将该bean装配给@Autowired指定的数据；

如果查询的结果不止一个，那么@Autowired会根据名称来查找；

如果上述查找的结果为空，那么会抛出异常。解决方法时，使用required=false。

@Autowired可用于：构造函数、成员变量、Setter方法

注：@Autowired和@Resource之间的区别

(1) @Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）。

(2) @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入。





## SpringMVC的执行流程

![image-20211007173501843](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20211007173501843.png)
