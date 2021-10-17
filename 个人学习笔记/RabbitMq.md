# 1.什么是中间件

## 1、什么是中间件

我国企业从20世纪80年代开始就逐渐进行信息化建设，由于方法和体系的不成熟，以及企业业务和市场需求的不断变化，一个企业可能同时运行着多个不同的业务系统，这些系统可能基于不同的操作系统、不同的数据库、异构的网络环境。现在的问题是，如何把这些信息系统结合成一个有机地协同工作的整体，真正**实现企业跨平台、分布式应用**。中间件便是解决之道，它用自己的复杂换取了企业应用的简单。

```
中间件（Middleware）是处于操作系统和应用程序之间的软件，也有人认为它应该属于操作系统中的一部分。人们在使用中间件时，往往是一组中间件集成在一起，构成一个平台（包括开发平台和运行平台），但在这组中间件中必须要有一个通信中间件，即中间件=平台+通信，这个定义也限定了只有用于分布式系统中才能称为中间件，同时还可以把它与支撑软件和实用软件区分开来。
```

举例：
1，RMI（Remote Method Invocations, 远程调用）
2，Load Balancing(负载均衡，将访问负荷分散到各个服务器中)
3，Transparent Fail-over(透明的故障切换)
4，Clustering(集群,用多个小的服务器代替大型机）
5，Back-end-Integration(后端集成，用现有的、新开发的系统如何去集成遗留的系统)
6，Transaction事务（全局/局部）全局事务（分布式事务）局部事务（在同一数据库联接内的事务）
7，Dynamic Redeployment(动态重新部署,在不停止原系统的情况下，部署新的系统）
8，System Management(系统管理)
9，Threading(多线程处理)
10，Message-oriented Middleware面向消息的中间件（异步的调用编程）
11，Component Life Cycle(组件的生命周期管理)
12，Resource pooling（资源池）
13，Security（安全）
14，Caching（缓存）

## 2、为什么需要使用消息中间件

具体地说，中间件屏蔽了底层操作系统的复杂性，使程序开发人员面对一个简单而统一的开发环境，减少程序设计的复杂性，将注意力集中在自己的业务上，不必再为程序在不同系统软件上的移植而重复工作，从而大大减少了技术上的负担。中间件带给应用系统的，不只是开发的简便、开发周期的缩短，也减少了系统的维护、运行和管理的工作量，还减少了计算机总体费用的投入。

## 3、中间件特点

为解决分布异构问题，人们提出了中间件(middleware)的概念。中间件是位于平台(硬件和操作系统)和应用之间的通用服务，如下图所示，这些服务具有标准的程序接口和协议。针对不同的操作系统和硬件平台，它们可以有符合接口和协议规范的多种实现。

![img](https://img-blog.csdnimg.cn/2018111321555179.jpg)

也许很难给中间件一个严格的定义，但中间件应具有如下的一些特点：
（1）满足大量应用的需要
（2）运行于多种硬件和OS平台
（3）支持分布计算，提供跨网络、硬件和OS平台的透明性的应用或服务的交互
（4）支持标准的协议
（5）支持标准的接口

由于标准接口对于可移植性和标准协议对于互操作性的重要性，中间件已成为许多标准化工作的主要部分。对于应用软件开发，中间件远比操作系统和网络服务更为重要，中间件提供的程序接口定义了一个相对稳定的高层应用环境，不管底层的计算机硬件和系统软件怎样更新换代，只要将中间件升级更新，并保持中间件对外的接口定义不变，应用软件几乎不需任何修改，从而保护了企业在应用软件开发和维护中的重大投资。

> 简单说：中间件有个很大的特点，是脱离于具体设计目标，而具备提供普遍独立功能需求的模块。这使得中间件一定是可替换的。如果一个系统设计中，中间件是不可替换的，不是架构、框架设计有问题，那么就是这个中间件，在 别处可能是个中间件，在这个系统内是引擎。哈。

## 4、在项目中什么时候使用中间件技术

在项目的架构和重构中，使用任何技术和架构的改变我们都需要谨慎斟酌和思考，因为任何技术的融入和变化都可能人员，技术，和成本的增加，中间件的技术一般现在一些互联网公司或者项目中使用比较多，如果你仅仅还只是一个初创公司建议还是使用单体架构，最多加个缓存中间件即可，不要盲目追求新或者所谓的高性能，而追求的背后一定是业务的驱动和项目的驱动，因为一旦追求就意味着你的学习成本，公司的人员结构以及服务器成本，维护和运维的成本都会增加，所以需要谨慎选择和考虑。

但是作为一个开放人员，一定要有学习中间件技术的能力和思维，否则很容易当项目发展到一个阶段在去掌握估计或者在面试中提及，就会给自己带来不小的困扰，在当今这个时代这些技术也并不是什么新鲜的东西，如果去掌握和挖掘最关键的还是自己花时间和花精力去探讨和研究。

## 5、课程的规划和安排

- 消息中间件 ActiveMQ
- 消息中间件 RabbitMQ
- 消息中间件 Kafaka
- 消息中间件 RocketMQ
- 消息中间件应用场景说明
- 负载均衡中间件(Nginx/Lvs)
- 缓存中间件(Memcache/Redis)
- 数据库中间件(ShardingJdbc/Mycat)



# 2.中间件技术及架构的概述

知识图谱

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy7a8af1c2-b406-46d9-965e-df81525649cd.png)

## 01、学习中间件的方式和技巧

1：理解中间件在项目架构中的作用，以及各中间件的底层实现。
2：可以使用一些类比的生活概念去理解中间件，
3：使用一些流程图或者脑图的方式去梳理各个中间件在架构中的作用
4：尝试用java技术去实现中间件的远离
5：静下来去思考中间件在项目中设计的和使用的原因
6：如何找到对应的替代总结方案
7：尝试编写博文总结类同中间件技术的对比和使用场景。
8：学会查看中间件的源码以及开开源项目和博文。

## 02、学习目标

- 什么是消息中间件
- 什么是协议
- 什么是持久化
- 消息分发
- 消息的高可用
- 消息的集群
- 消息的容错
- 消息的冗余

## 03、什么是消息中间件

在实际的项目中，大部分的企业项目开发中，在早期都采用的是单体的架构模式，如下图：

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudybcffe923-72f4-413b-af26-dd91b742311b.png)

## 04、单体架构

在企业开发的中，大部分的初期架构都采用的是单体架构的模式进行架构，而这种架构的典型的特点：就是把所有的业务和模块，源代码，静态资源文件等都放在一个一工程中，如果其中的一个模块升级或者迭代发生一个很小变动都会重新编译和重新部署项目。 这种的架构存在的问题就是：
1：耦合度太高
2：运维的成本过高
3：不易维护
4：服务器的成本高
5：以及升级架构的复杂度也会增大
这样就有后续的分布式架构系统。如下

## 05、分布式架构

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudyd40834ed-a15f-4606-bb36-ce475b05a949.png)

何谓分布式系统呢：

> 通俗一点：就是一个请求由服务器端的多个服务（服务或者系统）协同处理完成

和单体架构不同的是，单体架构是一个请求发起jvm调度线程（确切的是tomcat线程池）分配线程Thread来处理请求直到释放，而分布式是系统是：一个请求是由多个系统共同来协同完成，jvm和环境都可能是独立。如果生活中的比喻的话，单体架构就想建设一个小房子很快就能够搞定，如果你要建设一个鸟巢或者大型的建筑，你就必须是各个环节的协同和分布，这样目的也是项目发展都后期的时候要去部署和思考的问题。我们也不能看出来：分布式架构系统存在的特点和问题如下：

**存在问题**
1：学习成本高，技术栈过多
2：运维成本和服务器成本增高
3：人员的成本也会增高
4：项目的负载度也会上升
5：面临的错误和容错性也会成倍增加
6：占用的服务器端口和通讯的选择的成本高
7：安全性的考虑和因素逼迫可能选择RMI/MQ相关的服务器端通讯。

**好处**
1：服务系统的独立，占用的服务器资源减少和占用的硬件成本减少，
确切的说是：可以合理的分配服务资源，不造成服务器资源的浪费
2：系统的独立维护和部署，耦合度降低，可插拔性。
3：系统的架构和技术栈的选择可以变的灵活（而不是单纯的选择java）
4：弹性的部署，不会造成平台因部署造成的瘫痪和停服的状态。

## 06、小结



# 3.基于消息中间件的分布式系统的架构

## 01、基于消息中间件的分布式系统的架构

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudyb888e5f0-2c0f-4576-af88-0176abfa7832.png)

从上图中可以看出来，消息中间件的是
1：利用可靠的消息传递机制进行系统和系统直接的通讯
2：通过提供消息传递和消息的排队机制，它可以在分布式系统环境下扩展进程间的通讯。

## 02、消息中间件应用的场景

1:跨系统数据传递
2:高并发的流量削峰
3:数据的分发和异步处理
4:大数据分析与传递
5:分布式事务
比如你有一个数据要进行迁移或者请求并发过多的时候，比如你有10W的并发请求下订单，我们可以在这些订单入库之前，我们可以把订单请求堆积到消息队列中，让它稳健可靠的入库和执行。

## 03、常见的消息中间件

ActiveMQ、RabbitMQ、Kafka、RocketMQ等。

## 04、消息中间件的本质及设计

它是一种接受数据，接受请求、存储数据、发送数据等功能的技术服务。

> MQ消息队列：负责数据的传接受，存储和传递，所以性能要过于普通服务和技术。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudybcd23133-d755-4146-95da-7f21a85f2e2f.png)

谁来生产消息，存储消息和消费消息呢？

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy7c171d88-687a-4c6c-8a97-2b257467172e.png)

## 05、消息中间件的核心组成部分

1：消息的协议
2：消息的持久化机制
3：消息的分发策略
4：消息的高可用，高可靠
5：消息的容错机制

## 06、小结

其实不论选择单体架构还是分布式架构都是项目开发的一个阶段，在什么阶段选择适合的架构方式，而不能盲目追求，最后造成的后果和问题都需要自己买单。但是作为一个开发人员学习和探讨新的技术是我们每个程序开发者都应该去保持和思考的问题。当我们没办法去改变社会和世界的时候，我们为了生活和生存那就必须要迎合企业和市场的需求，发挥你的价值和所学的才能，创造价值和实现自我。



# 4.消息队列协议



## 01、什么是协议

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy9e91d500-e775-45f3-92fa-78a6278efc51.png)

我们知道消息中间件负责数据的传递，存储，和分发消费三个部分，数据的存储和分发的过程中肯定要遵循某种约定成俗的规范，你是采用底层的TCP/IP，UDP协议还是其他的自己取构建等，而这些约定成俗的规范就称之为：协议。

> 所谓协议是指：
> 1：计算机底层操作系统和应用程序通讯时共同遵守的一组约定，只有遵循共同的约定和规范，系统和底层操作系统之间才能相互交流。
> 2：和一般的网络应用程序的不同它主要负责数据的接受和传递，所以性能比较的高。
> 3：协议对数据格式和计算机之间交换数据都必须严格遵守规范。

## 02、网络协议的三要素

1.语法。语法是用户数据与控制信息的结构与格式,以及数据出现的顺序。
2.语义。语义是解释控制信息每个部分的意义。它规定了需要发出何种控制信息,以及完成的动作与做出什么样的响应。
3.时序。时序是对事件发生顺序的详细说明。

比如我MQ发送一个信息，是以什么数据格式发送到队列中，然后每个部分的含义是什么，发送完毕以后的执行的动作，以及消费者消费消息的动作，消费完毕的响应结果和反馈是什么，然后按照对应的执行顺序进行处理。如果你还是不理解：大家每天都在接触的http请求协议:

> 1：语法：http规定了请求报文和响应报文的格式。
> 2：语义：客户端主动发起请求称之为请求。（这是一种定义，同时你发起的是post/get请求）
> 3：时序：一个请求对应一个响应。（一定先有请求在有响应，这个是时序）

而消息中间件采用的并不是http协议，而常见的消息中间件协议有：OpenWire、AMQP、MQTT、Kafka，OpenMessage协议。

> **面试题：为什么消息中间件不直接使用http协议呢？**
> 1: 因为http请求报文头和响应报文头是比较复杂的，包含了cookie，数据的加密解密，状态码，响应码等附加的功能，但是对于一个消息而言，我们并不需要这么复杂，也没有这个必要性，它其实就是负责数据传递，存储，分发就行，一定要追求的是高性能。尽量简洁，快速。
> 2:大部分情况下http大部分都是短链接，在实际的交互过程中，一个请求到响应很有可能会中断，中断以后就不会就行持久化，就会造成请求的丢失。这样就不利于消息中间件的业务场景，因为消息中间件可能是一个长期的获取消息的过程，出现问题和故障要对数据或消息就行持久化等，目的是为了保证消息和数据的高可靠和稳健的运行。

## 03：AMQP协议

AMQP：(全称：Advanced Message Queuing Protocol) 是高级消息队列协议。由摩根大通集团联合其他公司共同设计。是一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。Erlang中的实现有RabbitMQ等。
特性：
1：分布式事务支持。
2：消息的持久化支持。
3：高性能和高可靠的消息处理优势。

AMQP协议的支持者：
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy1705264a-5917-4bf7-99a8-2ed993b463fa.png)

## 04：MQTT协议

MQTT协议：（Message Queueing Telemetry Transport）消息队列是IBM开放的一个即时通讯协议，物联网系统架构中的重要组成部分。
特点：
1：轻量
2：结构简单
3：传输快，不支持事务
4：没有持久化设计。
应用场景：
1：适用于计算能力有限
2：低带宽
3：网络不稳定的场景。
支持者：

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy1705264a-5917-4bf7-99a8-2ed993b463fa.png)

## 05、OpenMessage协议

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy579c94ed-0947-439e-95e7-2ca6b425dc79.png)

是近几年由阿里、雅虎和滴滴出行、Stremalio等公司共同参与创立的分布式消息中间件、流处理等领域的应用开发标准。
特点：
1：结构简单
2：解析速度快
3：支持事务和持久化设计。

## 06、Kafka协议

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy707bddcb-ef4a-44e0-96f1-3c2b16c02028.png)

Kafka协议是基于TCP/IP的二进制协议。消息内部是通过长度来分割，由一些基本数据类型组成。
特点是：
1：结构简单
2：解析速度快
3：无事务支持
4：有持久化设计

## 07、小结

协议：是在tcp/ip协议基础之上构建的一种约定成俗的规范和机制、它的主要目的可以让客户端（应用程序 java，go）进行沟通和通讯。并且这种协议下规范必须具有持久性，高可用，高可靠的性能。



# 5.消息队列持久化

## 01、持久化

简单来说就是将数据存入磁盘，而不是存在内存中随服务器重启断开而消失，使数据能够永久保存。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudyf908e193-4ca3-44b7-87d0-cbb17b55a107.png)

## 02、常见的持久化方式

|          | ActiveMQ | RabbitMQ | Kafka | RocketMQ |
| -------- | -------- | -------- | ----- | -------- |
| 文件存储 | 支持     | 支持     | 支持  | 支持     |
| 数据库   | 支持     | /        | /     | /        |



# 6.消息的分发策略



## 01、消息的分发策略

MQ消息队列有如下几个角色
1：生产者
2：存储消息
3：消费者
那么生产者生成消息以后，MQ进行存储，消费者是如何获取消息的呢？一般获取数据的方式无外乎推（push）或者拉（pull）两种方式，典型的git就有推拉机制，我们发送的http请求就是一种典型的拉取数据库数据返回的过程。而消息队列MQ是一种推送的过程，而这些推机制会适用到很多的业务场景也有很多对应推机制策略。

## 02、场景分析一

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy4acfaeda-fc73-4832-9207-7a9aa45b15a4.png)

> 比如我在APP上下了一个订单，我们的系统和服务很多，我们如何得知这个消息被那个系统或者那些服务或者系统进行消费，那这个时候就需要一个分发的策略。这就需要消费策略。或者称之为消费的方法论。

## 03、场景分析二

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy1a150320-9535-423a-99ec-76c5e3e654f7.png)

> 在发送消息的过程中可能会出现异常，或者网络的抖动，故障等等因为造成消息的无法消费，比如用户在下订单，消费MQ接受，订单系统出现故障，导致用户支付失败，那么这个时候就需要消息中间件就必须支持消息重试机制策略。也就是支持：出现问题和故障的情况下，消息不丢失还可以进行重发。

## 04、消息分发策略的机制和对比

|          | ActiveMQ | RabbitMQ | Kafka | RocketMQ |
| -------- | -------- | -------- | ----- | -------- |
| 发布订阅 | 支持     | 支持     | 支持  | 支持     |
| 轮询分发 | 支持     | 支持     | 支持  | /        |
| 公平分发 | /        | 支持     | 支持  | /        |
| 重发     | 支持     | 支持     | /     | 支持     |
| 消息拉取 | /        | 支持     | 支持  | 支持     |



# 7.消息队列高可用和高可靠



### 01、什么是高可用机制

所谓高可用：是指产品在规定的条件和规定的时刻或时间内处于可执行规定功能状态的能力。
当业务量增加时，请求也过大，一台消息中间件服务器的会触及硬件（CPU,内存，磁盘）的极限，一台消息服务器你已经无法满足业务的需求，所以消息中间件必须支持集群部署。来达到高可用的目的。

### 02、集群模式1 - Master-slave主从共享数据的部署方式

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/28/kuangstudy71c973d1-2a1a-4a82-9d71-85182c38c9f2.png)

解说：生产者讲消费发送到Master节点，所有的都连接这个消息队列共享这块数据区域，Master节点负责写入，一旦Master挂掉，slave节点继续服务。从而形成高可用，

### 03、集群模式2 - Master- slave主从同步部署方式

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudy6504af54-f8f7-4d58-aac4-bb547176ae88.png)

解释：这种模式写入消息同样在Master主节点上，但是主节点会同步数据到slave节点形成副本，和zookeeper或者redis主从机制很类同。这样可以达到负载均衡的效果，如果消费者有多个这样就可以去不同的节点就行消费，以为消息的拷贝和同步会暂用很大的带宽和网络资源。在后续的rabbtmq中会有使用。

### 04、集群模式3 - 多主集群同步部署模式

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudy402ac1b2-2d51-493c-9bc0-37329912dd2b.png)

解释：和上面的区别不是特别的大，但是它的写入可以往任意节点去写入。

### 05、集群模式4 - 多主集群转发部署模式

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudycc632a4a-a382-4303-85f9-8f63e93cea7a.png)

解释：如果你插入的数据是broker-1中，元数据信息会存储数据的相关描述和记录存放的位置（队列）。
它会对描述信息也就是元数据信息就行同步，如果消费者在broker-2中进行消费，发现自己几点没有对应的消息，可以从对应的元数据信息中去查询，然后返回对应的消息信息，场景：比如买火车票或者黄牛买演唱会门票，比如第一个黄牛有顾客说要买的演唱会门票，但是没有但是他会去联系其他的黄牛询问，如果有就返回。

### 06、集群模式5 Master-slave与Breoker-cluster组合的方案

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudya02ff996-446e-4d39-b230-dadb183269e5.png)

解释：实现多主多从的热备机制来完成消息的高可用以及数据的热备机制，在生产规模达到一定的阶段的时候，这种使用的频率比较高。

这么集群模式，具体在后续的课程中会进行一个分析和讲解。他们的最终目的都是为保证：消息服务器不会挂掉，出现了故障依然可以抱着消息服务继续使用。

反正终归三句话：
1：要么消息共享，
2：要么消息同步
3：要么元数据共享

## 07、什么是高可靠机制

所谓高可用是指：是指系统可以无故障低持续运行，比如一个系统突然崩溃，报错，异常等等并不影响线上业务的正常运行，出错的几率极低，就称之为：高可靠。
在高并发的业务场景中，如果不能保证系统的高可靠，那造成的隐患和损失是非常严重的。
如何保证中间件消息的可靠性呢？可以从两个方面考虑：
1：消息的传输：通过协议来保证系统间数据解析的正确性。
2：消息的存储可靠：通过持久化来保证消息的可靠性。



# 8.RabbitMQ入门及安装



## 01、概述

官网：https://www.rabbitmq.com/
什么是RabbitMQ,官方给出来这样的解释：

> RabbitMQ is the most widely deployed open source message broker.
> With tens of thousands of users, RabbitMQ is one of the most popular open source message brokers. From T-Mobile to Runtastic, RabbitMQ is used worldwide at small startups and large enterprises.
> RabbitMQ is lightweight and easy to deploy on premises and in the cloud. It supports multiple messaging protocols. RabbitMQ can be deployed in distributed and federated configurations to meet high-scale, high-availability requirements.
> RabbitMQ runs on many operating systems and cloud environments, and provides a wide range of developer tools for most popular languages.
> 翻译以后：
> RabbitMQ是部署最广泛的开源消息代理。
> RabbitMQ拥有成千上万的用户，是最受欢迎的开源消息代理之一。从T-Mobile 到Runtastic，RabbitMQ在全球范围内的小型初创企业和大型企业中都得到使用。
> RabbitMQ轻巧，易于在内部和云中部署。它支持多种消息传递协议。RabbitMQ可以部署在分布式和联合配置中，以满足大规模，高可用性的要求。
> RabbitMQ可在许多操作系统和云环境上运行，并为大多数流行语言提供了广泛的开发人员工具。

简单概述：
RabbitMQ是一个开源的遵循AMQP协议实现的基于Erlang语言编写，支持多种客户端（语言）。用于在分布式系统中存储消息，转发消息，具有高可用，高可扩性，易用性等特征。

## 02、安装RabbitMQ

1：下载地址：https://www.rabbitmq.com/download.html
2：环境准备：CentOS7.x+ / Erlang
RabbitMQ是采用Erlang语言开发的，所以系统环境必须提供Erlang环境，第一步就是安装Erlang。

> erlang和RabbitMQ版本的按照比较: https://www.rabbitmq.com/which-erlang.html
> ![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudy8f5da415-ab34-4a72-a375-c9f3123cab00.png)

## 03、 Erlang安装

查看系统版本号

```
[root@iZm5eauu5f1ulwtdgwqnsbZ ~]# lsb_release -aLSB Version:    :core-4.1-amd64:core-4.1-noarchDistributor ID: CentOSDescription:    CentOS Linux release 8.3.2011Release:        8.3.2011Codename:       n/a
```

#### 3-1:安装下载

参考地址：https://www.erlang-solutions.com/downloads/

```
wget https://packages.erlang-solutions.com/erlang-solutions-2.0-1.noarch.rpmrpm -Uvh erlang-solutions-2.0-1.noarch.rpm
```

#### 3-2：安装成功

```
yum install -y erlang
```

#### 3-3：安装成功

```
erl -v
```

## 04、安装socat

```
yum install -y socat
```

### 05、安装rabbitmq

下载地址：https://www.rabbitmq.com/download.html
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudy6323e2bf-a056-4aec-a124-13dd814dc9c2.png)

#### 5-1:下载rabbitmq

```
> wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.13/rabbitmq-server-3.8.13-1.el8.noarch.rpm> rpm -Uvh rabbitmq-server-3.8.13-1.el8.noarch.rpm
```

#### 5-2:启动rabbitmq服务

```
# 启动服务> systemctl start rabbitmq-server# 查看服务状态> systemctl status rabbitmq-server# 停止服务> systemctl stop rabbitmq-server# 开机启动服务> systemctl enable rabbitmq-server
```

## 06、RabbitMQ的配置

RabbitMQ默认情况下有一个配置文件，定义了RabbitMQ的相关配置信息，默认情况下能够满足日常的开发需求。如果需要修改需要，需要自己创建一个配置文件进行覆盖。
参考官网：
1:https://www.rabbitmq.com/documentation.html
2:https://www.rabbitmq.com/configure.html
3:https://www.rabbitmq.com/configure.html#config-items
4：https://github.com/rabbitmq/rabbitmq-server/blob/add-debug-messages-to-quorum_queue_SUITE/docs/rabbitmq.conf.example

### 06-1、相关端口

> 5672:RabbitMQ的通讯端口
> 25672:RabbitMQ的节点间的CLI通讯端口是
> 15672:RabbitMQ HTTP_API的端口，管理员用户才能访问，用于管理RabbitMQ,需要启动Management插件。
> 1883，8883：MQTT插件启动时的端口。
> 61613、61614：STOMP客户端插件启用的时候的端口。
> 15674、15675：基于webscoket的STOMP端口和MOTT端口

一定要注意：RabbitMQ 在安装完毕以后，会绑定一些端口，如果你购买的是阿里云或者腾讯云相关的服务器一定要在安全组中把对应的端口添加到防火墙。



# 9.RabbitMQWeb管理界面及授权操作



## 01、RabbitMQ管理界面

### 01-1：默认情况下，rabbitmq是没有安装web端的客户端插件，需要安装才可以生效

```
rabbitmq-plugins enable rabbitmq_management
```

> 说明：rabbitmq有一个默认账号和密码是：`guest` 默认情况只能在localhost本机下访问，所以需要添加一个远程登录的用户。

### 01-2：安装完毕以后，重启服务即可

```
systemctl restart rabbitmq-server
```

> 一定要记住，在对应服务器(阿里云，腾讯云等)的安全组中开放`15672`的端口。

### 01-3：在浏览器访问

http://ip:15672/ 如下：
![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/01/kuangstudydc5e08c8-0333-4cdf-8e70-c9523a133443.png)

## 02、授权账号和密码

### 2-1：新增用户

```
rabbitmqctl add_user admin admin
```

### 2-2:设置用户分配操作权限

```
rabbitmqctl set_user_tags admin administrator
```

用户级别：

- 1、administrator 可以登录控制台、查看所有信息、可以对rabbitmq进行管理
- 2、monitoring 监控者 登录控制台，查看所有信息
- 3、policymaker 策略制定者 登录控制台,指定策略
- 4、managment 普通管理员 登录控制台

### 2-3：为用户添加资源权限

```
rabbitmqctl.bat set_permissions -p / admin ".*" ".*" ".*"
```

## 03、小结：

```
rabbitmqctl add_user 账号 密码
rabbitmqctl set_user_tags 账号 administrator
rabbitmqctl change_password Username Newpassword 修改密码
rabbitmqctl delete_user Username 删除用户
rabbitmqctl list_users 查看用户清单
rabbitmqctl set_permissions -p / 用户名 ".*" ".*" ".*" 为用户设置administrator角色
rabbitmqctl set_permissions -p / root ".*" ".*" ".*"
```



# 10.RabbitMQ之Docker安装



## 01、Docker安装RabbitMQ

### 1-1、虚拟化容器技术—Docker的安装

```bash
（1）yum 包更新到最新
> yum update
（2）安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
> yum install -y yum-utils device-mapper-persistent-data lvm2
（3）设置yum源为阿里云
> yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
（4）安装docker
> yum install docker-ce -y
（5）安装后查看docker版本
> docker -v
 (6) 安装加速镜像
 sudo mkdir -p /etc/docker
 sudo tee /etc/docker/daemon.json <<-'EOF'
 {
  "registry-mirrors": ["https://0wrdwnn6.mirror.aliyuncs.com"]
 }
 EOF
 sudo systemctl daemon-reload
 sudo systemctl restart docker
```

### 1-2、docker的相关命令

```bash
# 启动docker：
systemctl start docker
# 停止docker：
systemctl stop docker
# 重启docker：
systemctl restart docker
# 查看docker状态：
systemctl status docker
# 开机启动：  
systemctl enable docker
systemctl unenable docker
# 查看docker概要信息
docker info
# 查看docker帮助文档
docker --help
```

### 1-3、安装rabbitmq

参考网站：
1：https://www.rabbitmq.com/download.html
2：https://registry.hub.docker.com/_/rabbitmq/

### 1-4、获取rabbit镜像：

```
docker pull rabbitmq:management
```

### 1-5、创建并运行容器

```
docker run -di --name=myrabbit -p 15672:15672 rabbitmq:management
```

—hostname：指定容器主机名称
—name:指定容器名称
-p:将mq端口号映射到本地
或者运行时设置用户和密码

```
docker run -di --name myrabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management
```

查看日志

```
docker logs -f myrabbit
```

### 1-6、容器运行正常

使用 `http://你的IP地址:15672` 访问rabbit控制台

## 02、额外Linux相关排查命令

```
> more xxx.log  查看日记信息
> netstat -naop | grep 5672 查看端口是否被占用
> ps -ef | grep 5672  查看进程
> systemctl stop 服务
```



# 11.RabbitMQ的角色分类



## 01、RabbitMQ的角色分类

### 1：none：

- 不能访问management plugin

### 2：management：查看自己相关节点信息

- 列出自己可以通过AMQP登入的虚拟机
- 查看自己的虚拟机节点 virtual hosts的queues,exchanges和bindings信息
- 查看和关闭自己的channels和connections
- 查看有关自己的虚拟机节点virtual hosts的统计信息。包括其他用户在这个节点virtual hosts中的活动信息。

### 3：Policymaker

- 包含management所有权限
- 查看和创建和删除自己的virtual hosts所属的policies和parameters信息。

### 4：Monitoring

- 包含management所有权限
- 罗列出所有的virtual hosts，包括不能登录的virtual hosts。
- 查看其他用户的connections和channels信息
- 查看节点级别的数据如clustering和memory使用情况
- 查看所有的virtual hosts的全局统计信息。

### 5：Administrator

- 最高权限
- 可以创建和删除virtual hosts
- 可以查看，创建和删除users
- 查看创建permisssions
- 关闭所有用户的connections

### 6、具体操作的界面

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy8888eeb0-3146-432b-9201-4f9622ca9c74.png)



# 12.RabbitMQ入门案例 - Simple 简单模式



## 01、实现步骤

1：jdk1.8
2：构建一个maven工程
3：导入rabbitmq的maven依赖
4：启动rabbitmq-server服务
5：定义生产者
6：定义消费者
7：观察消息的在rabbitmq-server服务中的过程

## 02、构建一个maven工程

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy2531c6a9-fd6a-4665-af9e-fdf2d5739df8.png)

## 03、导入rabbitmq的maven依赖

### 03-1：Java原生依赖

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.10.0</version>
</dependency>
```

### 03-2：spring依赖

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-amqp</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
```

#### 03-3、springboot依赖

```xml
<dependency> 
  <groupId>org.springframework.boot</groupId>    
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

上面根据自己的项目环境进行选择即可。

> 番外：rabbitmq和spring同属一个公司开放的产品，所以他们的支持也是非常完善，这也是为什么推荐使用rabbitmq的一个原因。

## 04、启动rabbitmq-server服务

```bash
systemctl start rabbitmq-server或者docker start myrabbit
```

## 05、定义生产者

```java
package com.xuexiangban.rabbitmq.simple;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
/**
 * @author: 学相伴-飞哥
 * @description: Producer 简单队列生产者
 * @Date : 2021/3/2
 */
public class Producer {
    public static void main(String[] args) {
        // 1: 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 设置连接属性
        connectionFactory.setHost("47.104.141.27");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        Connection connection = null;
        Channel channel = null;
        try {
            // 3: 从连接工厂中获取连接
            connection = connectionFactory.newConnection("生产者");
            // 4: 从连接中获取通道channel
            channel = connection.createChannel();
            // 5: 申明队列queue存储消息
            /*
             *  如果队列不存在，则会创建
             *  Rabbitmq不允许创建两个相同的队列名称，否则会报错。
             *
             *  @params1： queue 队列的名称
             *  @params2： durable 队列是否持久化
             *  @params3： exclusive 是否排他，即是否私有的，如果为true,会对当前队列加锁，其他的通道不能访问，并且连接自动关闭
             *  @params4： autoDelete 是否自动删除，当最后一个消费者断开连接之后是否自动删除消息。
             *  @params5： arguments 可以设置队列附加参数，设置队列的有效期，消息的最大长度，队列的消息生命周期等等。
             * */
            channel.queueDeclare("queue1", false, false, false, null);
            // 6： 准备发送消息的内容
            String message = "你好，学相伴！！！";
            // 7: 发送消息给中间件rabbitmq-server
            // @params1: 交换机exchange
            // @params2: 队列名称/routing
            // @params3: 属性配置
            // @params4: 发送消息的内容
            channel.basicPublish("", "queue1", null, message.getBytes());
            System.out.println("消息发送成功!");
        } catch (Exception ex) {
            ex.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 7: 释放连接关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
            if (connection != null) {
                try {
                    connection.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }
    }
}
```

1：执行发送，这个时候可以在web控制台查看到这个队列queue的信息

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy51240e70-79d7-4b3f-9a83-6febb3499a42.png)

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudydf396d39-9059-49fd-aaaa-a41b027c2a4b.png)

2：我们可以进行对队列的消息进行预览和测试如下：

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudyed656121-1e27-4c0e-84e7-70ae48f3b0f1.png)

3:进行预览和获取消息进行测试

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudydc7c7bf4-bffe-4821-92da-c1c8563631d3.png)

## 06、定义消费者

```java
package com.xuexiangban.rabbitmq.simple;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
/**
 * @author: 学相伴-飞哥
 * @description: Producer 简单队列生产者
 * @Date : 2021/3/2
 */
public class Producer {
    public static void main(String[] args) {
        // 1: 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        // 2: 设置连接属性
        connectionFactory.setHost("47.104.141.27");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        Connection connection = null;
        Channel channel = null;
        try {
            // 3: 从连接工厂中获取连接
            connection = connectionFactory.newConnection("生产者");
            // 4: 从连接中获取通道channel
            channel = connection.createChannel();
            // 5: 申明队列queue存储消息
            /*
             *  如果队列不存在，则会创建
             *  Rabbitmq不允许创建两个相同的队列名称，否则会报错。
             *
             *  @params1： queue 队列的名称
             *  @params2： durable 队列是否持久化
             *  @params3： exclusive 是否排他，即是否私有的，如果为true,会对当前队列加锁，其他的通道不能访问，并且连接自动关闭
             *  @params4： autoDelete 是否自动删除，当最后一个消费者断开连接之后是否自动删除消息。
             *  @params5： arguments 可以设置队列附加参数，设置队列的有效期，消息的最大长度，队列的消息生命周期等等。
             * */
            channel.queueDeclare("queue1", false, false, false, null);
            // 6： 准备发送消息的内容
            String message = "你好，学相伴！！！";
            // 7: 发送消息给中间件rabbitmq-server
            // @params1: 交换机exchange
            // @params2: 队列名称/routing
            // @params3: 属性配置
            // @params4: 发送消息的内容
            channel.basicPublish("", "queue1", null, message.getBytes());
            System.out.println("消息发送成功!");
        } catch (Exception ex) {
            ex.printStackTrace();
            System.out.println("发送消息出现异常...");
        } finally {
            // 7: 释放连接关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
            if (connection != null) {
                try {
                    connection.close();
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
            }
        }
    }
}
```

## 07、观察消息的在rabbitmq-server服务中的过程

- 具体详细视频教程



# 13.什么是AMQP



## 什么是AMQP

AMQP全称：Advanced Message Queuing Protocol(高级消息队列协议)。是应用层协议的一个开发标准，为面向消息的中间件设计。

## AMQP生产者流转过程

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy7c8a41b8-e3bf-4821-a1f1-a18860277663.png)

## AMQP消费者流转过程

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/03/02/kuangstudy081077ba-eced-43f9-b148-6f63987f1d2f.png)