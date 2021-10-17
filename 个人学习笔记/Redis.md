# Redis简介

## 关系型数据库与非关系型数据库

### 关系型数据库

采用关系模型来组织数据的数据库，关系模型就是二维表格模型。一张二维表的表名就是关系，二维表中的一行就是一条记录，二维表中的一列就是一个字段。



**优点**

- 容易理解，使用方便
- 通用的sql语言
- 易于维护，丰富的完整性(实体完整性、参照完整性和用户定义的完整性)大大降低了数据冗余和数据不一致的概率

**举例：** 

MySql,Oracle, Server，DB2



### 非关系型数据库

非关系型，分布式，一般不保证遵循ACID原则的数据存储系统。是基于键值对的存储结构，且结构不固定。



**优点**

- 根据需要添加字段，不需要多表联查。仅需id取出对应的value

- 适用于SNS（社会化网络服务软件。比如facebook，微博）

- 严格上讲不是一种数据库，而是一种数据结构化存储方法的集合

**缺点**

- 只适合存储一些较为简单的数据

- 不合适复杂查询的数据

- 不合适持久存储海量数据

**举例**

K-V：Redis，Memcache

文档：MongoDB

搜索：Elasticsearch，Solr



## 概述

Redis 是一个开源的，基于内存的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
它支持多种类型的数据结构，如字符串（strings），散列（hash），列表（lists），集合（sets），有序集合（sortedsets）与范围查询，bitmaps，hyperloglogs和地理空间（geospatial）索引半径查询。
Redis 内置了复制（replication），LUA脚本（Lua scripting），LRU驱动事件（LRU eviction），事务（transactions） 和不同 级别的磁盘持久化（persistence），并通过Redis哨兵（Sentinel）和自动分区（Cluster）提供高可用性（high availability）。



## Redis的特性

- 数据类型、基本操作和配置 
- 持久化和复制，RDB、AOF 
- 事务的控制 



## Redis和memcached的对比

1. 存储方式： 
   memecache 把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小 
   redis可以将数据持久化到硬盘上。 

2. 数据支持类型： 
   redis在数据支持上要比memecache多的多。 **memcached只支持存储复杂结构的文本结构如：json**

   

   **有持久化需求**或者对数据结构和处理有高级需求的应用，选择redis，其他简单的key/value存储，选择memcache。

# Redis单线程和多线程

Redis 是单线程，主要是指 **Redis 的键值对读写业务功能是由一个线程来完成的，工作线程就一个,满足redis的串行原子**。Redis在处理客户端的请求时包括获取 (socket 读)、解析、执行、内容返回 (socket 写) 等都由一个顺序串行的主线程处理，这就是所谓的“单线程”。这也是 Redis 对外提供键值存储服务的主要流程。

但 Redis 的其他功能，比如**异步删除、网络IO、RDB持久化、集群数据同步等，其实是由额外的线程执行的。**

￼

1.  版本3.x ，最早版本，那是的redis是 真 .单线程。 
2.  版本4.x，严格意义来说也不是单线程，而是负责处理客户端请求的线程是单线程，但是开始加了点多线程的东西(异步删除) 
3. 最新版本的6.0.x后， 告别了大家印象中的单线程，用一种全新的多线程来解决问题。

   ￼￼									<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210729213715696.png" alt="image-20210729213715696" style="zoom:50%;" />

对于 Redis 系统来说，随着数据量的变大， 主要的**性能瓶颈是IO,内存或者网络带宽**，**绝不是 CPU！**。



## Redis作为单线程的问题

**删除大键导致卡顿：**

​	如果是单线程，当被删除的 key 是一个非常大的对象，例如删除一个包含成千上万个元素的 hash 集合时，那么 del 指令就会造成卡	顿。 

​	因此为了解决大键删除的卡顿问题，从4.0以后开始使用了异步删除的缓存删除策略

**IO成为其瓶颈**

**持久化和集群同步会导致卡顿**

​	持久化和集群同步都是基于fork子进程的



## Redis的最终瓶颈

最后Redis的瓶颈可以初步定为：网络IO,因此redis6中采用了多路io复用

I/O 的读和写本身是堵塞的，比如当 socket 中有数据时，Redis 会通过调用read( )先将数据从内核态空间拷贝到用户态空间，再交给 Redis 调用，而这个拷贝的过程就是阻塞的，当数据量越大时拷贝所需要的时间就越多，而这些操作都是基于单线程完成的。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210729214738512.png" alt="image-20210729214738512" style="zoom:50%;" />

在 Redis 6.0 中新增了多线程的功能来提高 I/O 的读写性能

他的主要实现思路是**将主线程的 IO 读写任务拆分给一组独立的线程去执行**，多个 socket 的读写并行，采用I/O 多路复用技术，**将最耗时的Socket的读取、请求解析、写入单独外包出去，剩下的命令执行仍然由主线程串行执行并和内存的数据交互。** 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210729220206734.png" alt="image-20210729220206734" style="zoom:50%;" />





Redis 6.0 将网络数据读写、请求协议解析通过多个IO线程的来处理 ，对于真正的命令执行来说，仍然使用主线程操作，一举两得，便宜占尽！！！ o(￣▽￣)ｄ 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210729220751496.png" alt="image-20210729220751496" style="zoom:39%;" />



**总结：**

Redis的键值读写是单线程，但是异步删除、网络IO、持久化、集群数据同步等，由额外的线程执行。



**问：redis为什么选择单线程处理键值读写**

对线程处理键值对的读写会产生很多意想不到的并发问题；CPU并不是Redis的性能瓶颈，而是网络和IO，因此io多路复用很好的解决了网络和IO的问题



# Redis的epoll和io多路复用

**同步异步阻塞非阻塞**

- 同步 ：调用者等待调用结果通知后才能继续往下进行，不出结果就在那死等

- 异步： 被调用者先给调用者一个应答，调用者回去可以干别的事情，等被调用方出了结果再通知调用方

  同步、异步体现的是被调用者(服务提供者)，**对消息的通知方式的不同**

- 阻塞：调用发出去后， 调用方一直在等待，别的事情什么都不做，当前进/线程会被挂起，啥都不干

- 非阻塞：调用发出去后， 调用方不等待，去干别的事情，不会阻塞当前进/线程，而会立即返回

  阻塞和非阻塞体现的是**调用者等待调用结果的行为上的不同**



同步阻塞：服务员说快到你了，先别离开，我后台看一眼马上通知你。客户在海底捞火锅前台干等着，啥都不干。

同步非阻塞：服务员说快到你了，先别离开。客户在海底捞火锅前台边刷抖音边等着叫号

异步阻塞：服务员说还要再等等，一会儿通知你。客户找个位置去一边座位上傻等着啥也不干，一直等着店员通知

异步非阻塞：服务员说还要再等等，一会儿通知你。拿着排号小票+刷着抖音，等着店员通知



## BIO

如下图，两个客户端他们分别具有文件描述符fd8和fd9，因此两个客户端就要对应两个线程，而每个线程都是阻塞的 ，因为socket的数据没有到达内核，read命令就不会返回，后面的也执行不了，进而形成每个soket的阻塞。 每个socket都会一直等待队列中有数据才返回，是一个阻塞模型

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804225547560.png" alt="image-20210804225547560" style="zoom:50%;" />

缺点：只要连接了一个socket，操作系统就分配一个线程来处理， 如果连接的客户端迟迟不发数据 ，程序就会一直堵塞时时刻刻准备去读， 这样一次只能处理一个客户端。

**每个线程只分配一个连接，要想有多个连接必然产生多个线程**



## NIO

在nio模型中接收read是非阻塞的，在用户空间会对每个channel发生轮询，看是否有channel新的加入，是否有channel发送了消息

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804225914492.png" alt="image-20210804225914492" style="zoom:39%;" />

缺点：

- 虽然在用户空间发生轮询，这个模型在客户端少的时候十分好用，但是客户端如果很多，那么每次循环就要遍历大量的socket，如果一万个socket中只有10个socket有数据，也会遍历一万个socket，会做很多无用功。
- 遍历发生在用户态，用户态判断socket是否有数据是调用内核的read()方法实现的，轮询将会不断地调用read方法询问内核，**涉及到用户态和内核态的切换**，这将占用大量的 CPU 时间，系统资源利用率较低。 

为了解决大量无用的遍历，只要发现哪一个连接中能read读了，就不需要进行频繁的遍历，减少用户态内核态的切换，因此需要发展内核

## IO多路复用( I/O multiplexing )

**这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程**

select 、poll、 epoll只是和内核调用后知道了对应的socket是否有事件。

### select方法



为了解决用户空间发生的轮询进而衍生了select方法（1984年）

用户空间在内核中发起select系统调用，每次调用都返回一个文件描述符bit数组信息[1,0,0,1,0,0,1,0,1.....]（相当于bitmap，0代表没数据，1代表有数据），然后将生成的文件描述符bit数组信息&rset（监听端口）拷贝到内核，在内核中发生轮询遍历文件描述符bit数组信息，看哪个文件能去read读，再进行读取。此过程无限循环。

使用&rset标志位的优化，使得遍历发生在内核空间，避免了用户态和内核态的切换

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210805151714600.png" alt="image-20210805151714600" style="zoom:50%;" />

缺点：

- 文件描述符数组信息（监听端口）32位机默认是1024个，64位机默认是2048，这也是select最大的缺点，因为现在的服务器并发量远远不止1024。
- 每次循环都会将文件描述符数组信息&rset标记位置位为0
- 每一次循环都要将文件描述符数组信息&rset从用户态拷贝到内核态，拷来拷去



进一步演进：

### poll方法

调用过程和select类似

和select不同的地方：采用链表的方式替换原有r_set数据结构，poll使用pollfd的指针，pollfd结构包含了要监视的event和发生的       evevt，不再使用select传值的方法。更方便，而使其没有连接数的限制。



### epoll方法

- epoll维护的描述符数目不受到限制，而且性能不会随着文件描述符数目的增加而下降。
- 服务器的特点是经常维护着大量连接，但其中某一时刻读写的操作符数量却不多。epoll先通过epoll_ctl注册一个描述符到内核中，并一直维护着而不像poll每次操作都将所有要监控的描述符传递给内核；在描述符读写就绪时，通过回调函数将自己加入就绪队列中，之后epoll_wait返回该就绪队列。也就是说，epoll基本不做无用的操作，时间复杂度仅与活跃的客户端数有关，而不会随着描述符数目的增加而下降。
- epoll在传递内核与用户空间的消息时**使用了内存共享空间**，而不是一次次的从用户态到内核态的内存拷贝，使得epoll的效率比poll和select更高。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210805170226956.png" alt="image-20210805170226956" style="zoom:50%;" />

**问题引申：Redis单线程如何能处理那么多并发客户端连接，为什么用单线程，为什么那么快？**

服务器经常维护着大量连接，但其中某一时刻读写的操作符数量却不多，**通过epoll函数能监控到哪个连接向Redis发来消息**，Redis 利用 epoll函数来实现 IO 多路复用， 将连接信息和事件放到队列中 ，一次放到文件事件分派器 ，事件分派器将事件分发给事件处理器 。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804221545843.png" alt="image-20210804221545843" style="zoom:25%;" />

**什么是文件描述符**

文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

**内核利用文件描述符（file descriptor）来读写文件。**文件描述符是非负整数。打开现存文件或新建文件时，内核会返回一个文件描述符。

在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210926004115213.png" alt="image-20210926004115213" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210805144931219.png" alt="image-20210805144931219" style="zoom:50%;" />





# Redis五大数据类型



**Redis键 Key**

```bash
# keys * 查看所有的key
# exists key 的名字，判断某个key是否存在
# move key db  移除当前库的指定key
# expire key 秒钟：为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删 除。
# ttl key 查看还有多少秒过期，-1 表示永不过期，-2 表示已过期
```



## 字符串String



```bash
# 最常用
127.0.0.1:6379> set key value #设置 key-value 类型的值
OK
127.0.0.1:6379> get key # 根据 key 获得对应的 value
"value"
127.0.0.1:6379> exists key  # 判断某个 key 是否存在
(integer) 1
127.0.0.1:6379> strlen key # 返回 key 所储存的字符串值的长度。
(integer) 5
127.0.0.1:6379> del key # 删除某个 key 对应的值
(integer) 1
--------------------
# 数值增减
127.0.0.1:6379> set key 1
OK
127.0.0.1:6379> incr key # 递增数字
(integer) 2
127.0.0.1:6379> decr key # 递减数字
(integer) 1
127.0.0.1:6379> incrby key 5 # 递增指定的值
(integer) 6
127.0.0.1:6379> decrby key 6 # 递减指定的值
(integer) 0
```

**批量设置** :

```bash
127.0.0.1:6379> mset key1 value1 key2 value2 # 批量设置 key-value 类型的值
OK
127.0.0.1:6379> mget key1 key2 # 批量获取多个 key 对应的 value
1) "value1"
2) "value2"
```

**正反向索引**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210801145424568.png" alt="image-20210801145424568" style="zoom:50%;" />



**string类型做分布式锁**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210729225030174.png" alt="image-20210729225030174" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210801144130328.png" alt="image-20210801144130328" style="zoom:50%;" />

```bash
127.0.0.1:6379> setnx key1 v1  # 设置key1键的值为v1
(integer) 1
127.0.0.1:6379> setnx key1 v2  # 发现设置key1键的值为v2设不了
(integer) 0
127.0.0.1:6379> get key1 # 再次获取仍然是最开始的v1
"v1"
```

**应用场景**

- 适用于基于xxx量的统计，如网站访问量。
- 适用于存储简单的信息，json字符串，如储存项目中的类的信息



## 列表List

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210802103346837.png" alt="image-20210802103346837" style="zoom:50%;" />

 ```bash
 127.0.0.1:6379> lpush list1 a  # 添加元素
 (integer) 1
 127.0.0.1:6379> lpush list1 b	 # 添加元素
 (integer) 2
 127.0.0.1:6379> lpush list1 f  # 添加元素
 (integer) 3
 127.0.0.1:6379> lrange list1 0 -1  # 获取list的全部元素
 1) "f"
 2) "b"
 3) "a"
 127.0.0.1:6379> llen list1 # 获取list长度
 (integer) 3
 127.0.0.1:6379> lpop list1 # 向左出队列
 "f"
 127.0.0.1:6379> lpop list1
 "b"
 127.0.0.1:6379> lpop list1
 "a"
 127.0.0.1:6379> lpop list1 # 值全移除，键就没了
 (nil)
 ```



一个双端链表的结构， 容量是2的32次方减1个元素，大概40多亿，主要功能有push/pop等，一般用在栈、队列、消息队列等场景。 

- 它是一个字符串链表，left，right 都可以插入添加 
- 如果键不存在，创建新的链表 
- 如果键已存在，新增内容 
- **如果值全移除，对应的键也就消失了** 

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210727093718816.png" alt="image-20210727093718816" style="zoom:50%;" />

基于链表的数据结构，适合插入操作不适合查询。List的另一个应用就是消息队列，可以利用List的PUSH操作，将任务存在List中，然后工作线程再用POP操作将任务取出进行执行。

**应用场景**

想到是基于顺序 的存储，用作评论功能非常合适，因为评论需要时间排序（增序/降序排序）

## 哈希Hash

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210726092102064.png" alt="image-20210726092102064" style="zoom:30%;" />

**应用场景**

hash特别适合用于存储对象，存储。存储部分变更的数据，特别适合存储用户的某种行为信息，订单信息，**如用户信息，购物车信息等**。 



## 集合Set

在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis还为集合提供了求交集、并集、差集等运算功能，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210726093340101.png" alt="image-20210726093340101" style="zoom:50%;" />

**集合运算**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210726095604709.png" alt="image-20210726095604709" style="zoom:50%;" />

**应用场景**

因set含有很多计算功能所以应用场景也是非常多的

- set具有随机弹出元素的功能，因此适合**做抽奖程序**
- set具有去重功能和取总数功能，因此特别适合做点赞功能
- set可以求交集运算，可以**应用在共同爱好、共同关注功能上**





## 有序集合ZSet

在set基础上，加一个score值。之前set是k1 v1 v2 v3，现在zset是 k1 score1 v1 score2 v2和set相比，**sorted set增加了一个权重参数score**，**使得集合中的元素能够按score进行有序排列**，比如一个存储全班同学成绩的sorted set，其集合value可以是同学的学号，而score就可以是其考试得分。**在数据插入集合的时候，就已经进行了天然的排序**。可以用sorted set来做带权重的队列，比如普通消息的score为1，重要消息的score为2，然后工作线程可以选择按score的倒序来获取工作任务。让重要的任务优先执行。**排行榜应用，取TOP N操作 ！** 



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210727094717674.png" alt="image-20210727094717674" style="zoom:30%;" />

**应用场景**

zset内部具有排序功能，因此像是**排行榜，热搜**，这种取topN的功能最合适



**总结：**

<img src="https://static001.geekbang.org/resource/image/c0/6e/c0bb35d0d91a62ef4ca1bd939a9b136e.jpg" alt="img" style="zoom:20%;" />

# Redis五大数据类型的底层实现

**Redis字典数据库中的k-v键值对到底是什么**

redis 是 key-value 存储系统，其中key类型一般为字符串类型， **value 类型则为redis对象(redisObject)**



## String底层结构

String底层的三大结构：

### int 

保存Long型的8个字节（64位）的带符号的整数，0~9999 这一万个数是作为共享变量。 如果是浮点数就会转化成embstr，类似于Java中int类型的缓存（-128~127），

例如set key1 123 执行两次

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804104739710.png" alt="image-20210804104739710" style="zoom:50%;" />

### embstr

嵌入式的String（SDS ---Simple Dynamic String 简单动态字符串)，保存长度小于44字节，EMBSTR即：embedded string 表示嵌入式的String。从内存结构上来讲 即字符串 sds结构体与其对应的 redisObject 对象分配在同一块连续的内存空间，字符串sds嵌入在redisObject对象之中。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804110013530.png" alt="image-20210804110013530" style="zoom:50%;" />

**注：对于embstr，只要对其进行追加修改，无论修改后的长度是否大于44，都会改成raw类型**



### raw

保存长度大于44字节的字符串，长度大于44时，Redis 则会将键值的内部编码方式改为OBJ_ENCODING_RAW格式，这与OBJ_ENCODING_EMBSTR编码方式的不同之处在于，此时动态字符串sds的内存 与其依赖的redisObject的内存不再连续了 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804110204651.png" alt="image-20210804110204651" style="zoom:50%;" />



总结：逻辑转换图

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804110733338.png" alt="image-20210804110733338" style="zoom:39%;" />



### SDS数据结构

Redis没有直接复用C语言的字符串，而是 新建了属于自己的结构 --- SDS，**Redis中所有键以及包含字符串类型的值都是由SDS实现的**

Redis中字符串的实现,SDS有多种结构（sds.h）：

- sdshdr (2^5=32byte) 

- sdshdr8 (2 ^ 8=256byte) 

- sdshdr16 (2 ^ 16=65536byte=64KB) 

- sdshdr32  (2 ^ 32byte=4GB) 

- sdshdr64（2的64次方byte＝17179869184G）

- 用于存储不同的长度的字符串。

  **可见SDS的空间大小还可以动态调整**

  **注：在Redis中的value最大不能超过512M**   A String value can be at max 512 Megabytes in length.
  
  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804102708962.png" alt="image-20210804102708962" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804102759662.png" alt="image-20210804102759662" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804102819864.png" alt="image-20210804102819864" style="zoom:50%;" />



**Redis为什么重新设置了一个SDS数据结构**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804103009853.png" alt="image-20210804103009853" style="zoom:50%;" />

C语言没有Java里面的String类型 ，只有字节的char[ ]数组类型实现String，C语言中的字符串想要获取长度必须从头遍历，遇到\0就结束 ，但是SDS可以记录字符串长度len，**根据len判断字符串的结束解决了二进制安全的问题**；C语言的char[ ]超过空间分配长度就会数组下标越界溢出，而**SDS空间大小可以动态调整，有空间预分配**。 

|                    | C语言                                                        | SDS                                                          |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **字符串长度处理** | 需要从头开始遍历，直到遇到 '\0' 为止，时间复杂度O(N)         | 有变量记录当前字符串的长度，直接读取即可，时间复杂度 O(1)    |
| **内存重新分配**   | 分配内存空间超过后，会导致数组下标越级或者内存分配溢出       | 空间预分配 SDS 修改后，len 长度小于 1M，那么将会额外分配与 len 相同长度的未使用空间。如果修改后长度大于 1M，那么将分配1M的使用空间。 惰性空间释放 有空间分配对应的就有空间释放。SDS 缩短时并不会回收多余的内存空间，而是使用 free 字段将多出来的空间记录下来。如果后续有变更操作，直接使用 free 中记录的空间，减少了内存的分配。 |
| **二进制安全**     | 二进制数据并不是规则的字符串格式，可能会包含一些特殊的字符，比如 '\0' 等。前面提到过，C中字符串遇到 '\0' 会结束，那 '\0' 之后的数据就读取不上了 | 根据 len 长度来判断字符串结束的，二进制安全的问题就解决了    |



## list数据类型介绍

**list类型的参数配置**

- ziplist中entry配置：list-max-ziplist-size -2 

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804144152344.png" alt="image-20210804144152344" style="zoom:50%;" />

- ziplist压缩配置：list-compress-depth 0 

![image-20210804143837473](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804143837473.png)



**List数据类型底层的数据结构为QuickList，QuickList是LinkedList和ZipList的组合体**，它将 linkedList按段切分，每一段使用 zipList 来紧凑存储，多个 zipList 之间使用双向指针串接起来。 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804144859659.png" alt="image-20210804144859659" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804150923111.png" alt="image-20210804150923111" style="zoom:39%;" />



## hash数据结构介绍

hash类型的底层数据结构有两个：ZipList和HashTable

### ZipList

**参数：**

- hash-max-ziplist-entries：使用压缩列表保存时哈希集合中的最大元素个数， 默认数量小于 512 个

- hash-max-ziplist-value：使用压缩列表保存时哈希集合中单个元素的最大长度。 默认键和值的字符串长度都小于等于 64byte

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804143737705.png" alt="image-20210804143737705" style="zoom:50%;" />

上述两参数如果有一个不满足就会从ZipList转换为HashTable结构

​                    

一旦从ziplist压缩列表转化为hashtable就不会从hashtable转化成为ziplist，在节省内存空间方面哈希表就没有压缩列表高效。 					  					

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804113310128.png" alt="image-20210804113310128" style="zoom:50%;" />

**ziplist的结构**



ziplist不同于双向链表，它不存储前后节点的指针，取而代之的是**存储上一个节点长度和当前节点长度**。

去掉指针的原因：在存储数据很小的情况下， 我们存储的实际数据的大小可能还没有指针占用的内存大，为的是减少存储空间的浪费获更多的存储空间

它是由：**header+entry节点集合+end**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804114359179.png" alt="image-20210804114359179" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804114636322.png" alt="image-20210804114636322" style="zoom:50%;" />

```c
typedef struct zlentry {    // 压缩列表节点
   // prevrawlen是前一个节点的长度 ,prevrawlensize是指prevrawlen的大小，有1字节和5字节两种   
   unsigned int prevrawlensize, prevrawlen;   
    // len为当前节点长度 lensize为编码len所需的字节大小 
    unsigned int lensize, len;   
     // 当前节点的header大小 
    unsigned int headersize;    
    // 节点的编码方式 
    unsigned char encoding;  
    // 指向节点的指针 
    unsigned char *p;    
} zlentry; 
```



## Set数据类型介绍

Redis用IntSet或HashTable存储Set。**如果元素都是整数类型，就用intset存储。 如果不是整数类型，就用hashtable**（数组+链表的存来储结构）。key就是元素的值，value为null。 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804151726388.png" alt="image-20210804151726388" style="zoom:50%;" />



## ZSet数据类型介绍

**ZSet底层使用ZipList压缩列表和SkipList跳表**

两个属性：

- server.zset_max_ziplist_entries 的值（默认值为 128 ）： 包含最多键值对数量

- server.zset_max_ziplist_value 的值（默认值为 64 ）  ：新添加元素的 member 的长度大于

如果有一个不满足，redis会使用 跳跃表 作为有序集合的底层实现。 

否则会使用ziplist作为有序集合的底层实现 

### 跳表

跳表是将链表进行一维转二维，空间换取时间的结构 。 

对于一个单链表来说，即使链表中的数据是有序的，如果我们想要查找某个数据，也必须从头到尾的遍历链表，很显然这种查找效率是十分低效的，时间复杂度为O(n)。

那么我们如何提高查找效率呢？我们可以对链表建立一级“索引”，每两个结点提取一个结点到上一级，我们把抽取出来的那一级叫做索引或者索引层







**跳表 = 链表 + 多级索引**

一般每两个元素设一个索引，一层一层往上加

![img](https://img-blog.csdnimg.cn/20181029205153331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bjd2wxMjA2,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181029205547197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bjd2wxMjA2,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20181029211746472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bjd2wxMjA2,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/2018102921424840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Bjd2wxMjA2,size_16,color_FFFFFF,t_70)

**跳表查找任意数据的时间复杂度为O(logn)，空间复杂度还是O(N)**，索引数为N/2 +N/4 + N/8 + ....+ 2只需要增加相同大小的空间便可以建立索引

数组中插入数据困难，查询方便；链表插入数据容易，查询难。

跳表就是链表和数组的折中优化



## 总结

1. String

   - int:8个字节的长整型。 
   - embstr:小于等于44个字节的字符串。
   - raw:大于44个字节的字符串。 

   Redis会根据当前值的类型和长度决定使用哪种内部编码实现。 

2. Hash 
   - ziplist(压缩列表):
   - hashtable(哈希表) 

3. List

   底层使用QuickList其中包含：

   - ziplist(压缩列表)
   - linkedlist(链表)

4. Set 
   - intset(整数集合)。 
   - hashtable(哈希表)

5. 有序集合 
   - ziplist(压缩列表)
   - skiplist(跳跃表)

Redis的每个数据类型会根据指定的配置规则来选择底层使用哪种数据结构

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804153404886.png" alt="image-20210804153404886" style="zoom:33%;" />

# 三种特殊数据类型

**亿级系统中常见的四种统计**

1. 聚合统计： 交并差集和聚合函数的应用
2. 排序统计
3. 二值统计：集合元素的取值就只有0和1两种。签到打卡的场景中，我们只用记录有签到(1)或没签到(0)
4. 基数统计：指统计⼀个集合中不重复的元素个数



## BitMap

### 介绍

常用作亿级数据的收集+统计，使用bitmap真正实现了**存的进+取得快+多统计** ，bitmap最有价值的就是它的**统计**功能！

一句话：bitmap是由0和1状态表现的二进制位的bit数组

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730001314450.png" alt="image-20210730001314450" style="zoom:50%;" />

位图本质是数组，它是基于String数据类型的按位的操作。该数组由多个二进制位组成，每个二进制位都对应一个偏移量(我们可以称之为一个索引或者位格)。Bitmap支持的最大位数是2^32位，它可以极大的节约存储空间，使用512M内存就可以存储多大42.9亿的字节信息(2^32 = 4294967296) 





**应用场景**

签到打卡

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730001459261.png" alt="image-20210730001459261" style="zoom:50%;" />



**签到打卡案例说明**

- 按照天

  SETBIT key offset value : 设置 key 的第 offset 位为value (1或0)	

  ```bash
  # 使用 bitmap 来记录上述事例中一周的打卡记录如下所示： 
  # 周一：1，周二：0，周三：1，周四：1，周五：1，周六：0，周天：1 （1 为打卡，0 为不打卡）
  127.0.0.1:6379> setbit sign 0 1
  (integer) 0
  127.0.0.1:6379> setbit sign 1 1 
  (integer) 0
  127.0.0.1:6379> setbit sign 2 0
  (integer) 0
  127.0.0.1:6379> setbit sign 3 1
  (integer) 0
  127.0.0.1:6379> setbit sign 4 1
  (integer) 0
  127.0.0.1:6379> setbit sign 5 0
  (integer) 0
  127.0.0.1:6379> setbit sign 6 1
  (integer) 0
  -----------------------
  127.0.0.1:6379> getbit sign 1   #查看周一打卡情况
  (integer) 1
  127.0.0.1:6379> getbit sign 2   #查看周二打卡情况
  (integer) 0
  127.0.0.1:6379> bitcount sign   #查看一周打卡天数
  (integer) 5
  
  # bitcount 统计操作 
  #bitcount key [start, end] 统计 key 上位为1的个数
  127.0.0.1:6379> bitcount sign 0 6  #查看一周打卡天数
  (integer) 5
  ```

  -  按照年

    按年去存储一个用户的签到情况，365 天只需要 365 / 8 ≈ 46 Byte，1000W 用户量一年也只需要 44 MB 就足够了。 假如是亿级的系统， 每天使用1个 1亿位的Bitmap约占12MB的内存 （10^8/8/1024/1024），10天的Bitmap的内存开销约为120MB，内存压力不算太高。在实际使用时，最好对Bitmap设置过期时间，让Redis自动删除不再需要的签到记录以节省内存开销

    

    <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730002544249.png" alt="image-20210730002544249" style="zoom:50%;" />



### bitmap底层原理

实质是二进制的ascii编码对应，redis里用type命令可以看到bitmap实质是String类型

![image-20210730003014717](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730003014717.png)

两个setbit命令对k1进行设置后，对应的二进制串就是0100 0001 ,对应的10进制就是65，65的ASCII码就是'A'



## HyperLogLog

### 介绍

**去重统计的方式有哪些？**

1. HashSet

2. BitMap :bitmap是通过用位bit数组来表示各元素是否出现，每个元素对应一位，所需的总内存为N个bit。 

   基数计数则将每一个元素对应到bit数组中的其中一位，比如bit数组010010101(按照从零开始下标，有的就是1、4、6、8)。 

   新进入的元素只需要将已经有的bit数组和新加入的元素进行按位或计算。可以大大减少内存占用且位操作迅速。 

   But，假设一个样本案例就是一亿个基数位值数据，一个样本就是一亿 。如果要统计1亿个数据的基数位值, 大约需要内存100000000/8/1024/1024约等于12M ,内存减少占用的效果显著。 这样得到统计一个对象样本的基数值需要12M。 

    如果统计10000个对象样本(1w个亿级),就需要117.1875G将近120G，可见使用bitmaps还是不适用大数据量下(亿级)的基数计数场景， 但是bitmaps方法是精确计算的。 

   **总结**：样本元素越多内存消耗急剧增大，难以管控+各种慢，对于亿级统计不太合适，大数据害死人，o(╥﹏╥)o

3. **HyperLogLog** ： 通过牺牲准确率来换取空间 ，对于不要求 绝对准确率 的场景下可以使用，因为 概率算法不直接存储数据本身， 通过一定的概率统计方法预估基数值，同时保证误差在一定范围内，由于又不储存数据故此可以大大节约内存

**活跃用户统计名词**

- UV (Unique Visitor) ：独立访客，一般理解为客户端IP，需要去重考虑
- PV (Page View) ： Page View，页面浏览量，不需要去重
- DAU (Day ) ：日活跃用户量
- MAU （MonthIy Active User）：月活跃用户量

**用作去重功能的基数统计-就用HyperLogLog** 

基数统计：用于统计一个集合中不重复的元素个数，就是对集合去重复后剩余元素的计算

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730093904331.png" alt="image-20210730093904331" style="zoom:50%;" />



### 原理说明

- 只是进行不重复的基数统计，不是集合也不保存数据，只记录数量而不是具体内容。

- 有误差，HyperLogLog是非精确统计，**牺牲准确率来换取空间，误差仅仅只是0.81%左右**    1.04/sqrt(10384)

- 只需要12kb就可以统计2^64种不同的数据，实现了能够使用极少的内存来统计巨量的数据

  ![image-20210730104358621](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730104358621.png)

  每个桶取6位，16384*6÷8 = 12kb，每个桶有6位，最大全部都是1，值就是63 

  

  

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730103258259.png" alt="image-20210730103258259" style="zoom:30%;" />

  问：为什么redis集群的最大槽数为16384个

  ```
  正常的心跳数据包带有节点的完整配置
  这意味着它们包含原始节点的插槽配置，该节点使用2k的空间和16k的插槽，但是会使用8k的空间（使用65k的插槽）。 
  同时，由于其他设计折衷，Redis集群不太可能扩展到1000个以上的主节点。 
  因此16k处于正确的范围内，以确保每个主机具有足够的插槽，最多可容纳1000个矩阵，但数量足够少，可以轻松地将插槽配置作为原始位图传播。请注意，在小型群集中，位图将难以压缩，因为当N较小时，位图将设置的slot / N位占设置位的很大百分比
  ```

  

 

| 命令                             | 描述                                                |
| -------------------------------- | --------------------------------------------------- |
| [PFADD key element [element ...] | 添加指定元素到 HyperLogLog 中。                     |
| [PFCOUNT key [key ...]           | 返回给定 HyperLogLog 的基数估算值。                 |
| [PFMERGE destkey sourcekey       | 将多个 HyperLogLog 合并为一个 HyperLogLog，并集计算 |

```bash
127.0.0.1:6379> pfadd mykey a s d f g s d f a s
(integer) 1
127.0.0.1:6379> pfcount mykey
(integer) 5
127.0.0.1:6379> pfadd mykey1 1 2 3 4 5 2 3 1 
(integer) 1
127.0.0.1:6379> pfcount mykey1
(integer) 5
127.0.0.1:6379>  pfmerge mykey2 mykey mykey1
OK
127.0.0.1:6379> pfcount mykey2
(integer) 10
```



**应用场景：网站首页的亿级UV的访问量统计**

UV的统计需要去重，一个用户一天内的多次访问只能算作一次

淘宝、天猫首页的UV，平均每天是1~1.5个亿左右

每天存1.5个亿的IP，访问者来了后先去查是否存在，不存在加入



## GEO地理位置

该功能可以将用户给定的地理位置信息储存起来， 并对这些信息进行操作。来实现诸如附近位置、摇一摇这类依赖于地理位置信息的功能。geo的数据类型为zset。GEO 的数据结构总共有六个常用命令：geoadd、geopos、geodist、georadius、georadiusbymember、gethash 

### 原理

![image-20210730145845409](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730145845409.png)

**geoadd**

将给定的空间元素(纬度、经度、名字)添加到指定的键，这些数据会以有序集合的形式被储存在键里面，从而可以通过位置查询取得这些元素。 geoadd命令以标准的x,y格式接受参数,所以用户必须先输入经度,然后再输入纬度。 geoadd能够记录的坐标是有限的:非常接近两极的区域无法被索引。 有效的经度介于-180-180度之间，有效的纬度介于-85.05112878 度至 85.05112878 度之间,当用户尝试输入一个超出范围的经度或者纬度时,geoadd命令将返回一个错误。 

```bash
geoadd key longitude latitude member
```

```bash
127.0.0.1:6379> geoadd china:city 121.48 31.40 上海 113.88 22.55 深圳 120.21 30.20 杭州
(integer) 3
127.0.0.1:6379> geoadd china:city 106.54 29.40 重庆 108.93 34.23 西安 114.02 30.58 武汉
(integer) 3

```



**geopos**

```bash
# 语法 
geopos key member [member...] #从key里返回所有给定位置元素的位置（经度和纬度）
```

```bash
#从key里返回所有给定位置元素的位置（经度和纬度）
127.0.0.1:6379> geopos china:city 北京
1) 1) "116.23000055551528931" 
   2) "40.2200010338739844"
```



**geodist**

返回两个给定位置之间的距离，如果两个位置之间的其中一个不存在,那么命令返回空值。指定单位的参数unit必须是以下单位的其中一个： 

m表示单位为米 

km表示单位为千米 

mi表示单位为英里 

ft表示单位为英尺 

如果用户没有显式地指定单位参数,那么geodist默认使用米作为单位。 

geodist命令在计算距离时会假设地球为完美的球形,在极限情况下,这一假设最大会造成0.5%的误 差。

```bash
 geodist key member1 member2 [unit] 
```

```bash
127.0.0.1:6379> geodist china:city 北京 上海 
"1088785.4302" 
127.0.0.1:6379> geodist china:city 北京 上海 km 
"1088.7854" 127.0.0.1:6379> 
geodist china:city 重庆 北京 km 
"1491.6716"
```

**georadius**

以给定的经纬度为中心， 找出某一半径内的元素 

```bash
georadius key longitude latitude radius m|km|ft|mi [withcoord][withdist] [withhash][asc|desc][count count]
```



```bash
 # 在 china:city 中寻找坐标 100 30 半径为 1000km 的城市 
 127.0.0.1:6379> georadius china:city 100 30 1000 km 
 重庆
 西安
 
 # withdist 返回位置名称和中心距离 
 127.0.0.1:6379> georadius china:city 100 30 1000 km withdist 
 重庆
 635.2850
 西安
 963.3171
 
 # withcoord 返回位置名称和经纬度 
 127.0.0.1:6379> georadius china:city 100 30 1000 km withcoord 
 重庆
 106.54000014066696167 29.39999880018641676 
 西安
 108.92999857664108276 34.23000121926852302
 
 #withdist withcoord 返回位置名称 距离 和经纬度 ,count 限定寻找个数 
 127.0.0.1:6379> georadius china:city 100 30 1000 km withcoord withdist count 1
 重庆
 635.2850 
 106.54000014066696167 
 29.39999880018641676 
 127.0.0.1:6379> georadius china:city 100 30 1000 km withcoord withdist count 2
 重庆
 635.2850 
 106.54000014066696167 
 29.39999880018641676
 西安
 963.3171
 108.92999857664108276 
 34.23000121926852302
```



**geohash **

```bash
geohash key member [member...] 
# Redis使用geohash将二维经纬度转换为一维字符串，字符串越长表示位置更精确,两个字符串越相似 表示距离越近。
```

```bash
127.0.0.1:6379> geohash china:city 北京 重庆
wx4sucu47r0 
wm5z22h53v0 
127.0.0.1:6379> geohash china:city 北京 上海 
wx4sucu47r0
wtw6sk5n300

```



**georadiusbymember**

```bash
#找出位于指定范围内的元素，中心点是由给定的位置元素决定
georadiusbymember key member radius m|km|ft|mi [withcoord][withdist] [withhash][asc|desc][count count]
```

```bash
127.0.0.1:6379> GEORADIUSBYMEMBER china:city 北京 1000 km 
北京
西安
127.0.0.1:6379> GEORADIUSBYMEMBER china:city 上海 400 km 
杭州
上海
```



**zrem **

GEO没有提供删除成员的命令，但是因为**GEO的底层实现是zset**，所以可以借用zrem命令实现对地理位置信息的删除. 

```bash
127.0.0.1:6379> zrem china:city beijin # 移除元素
1
```



**GEO的类型其实就是zset**

# 布隆过滤器

## 介绍

布隆过滤器（英语：Bloom Filter）是 1970 年由布隆提出的。 

它实际上是一个很长的二进制数组+一系列随机hash算法映射函数，主要用于判断一个元素是否在集合中。  

通常我们会遇到很多要判断一个元素是否在某个集合中的业务场景，一般想到的是将集合中所有元素保存起来，然后通过比较确定。 

链表、树、散列表（又叫哈希表，Hash table）等等数据结构都是这种思路。 

但是随着集合中元素的增加，我们需要的存储空间也会呈现线性增长，最终达到瓶颈。同时检索速度也越来越慢，上述三种结构的检索时间复杂度分别为O(n),O(logn),O(1)。这个时候，布隆过滤器（Bloom Filter）就应运而生 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730150527819.png" alt="image-20210730150527819" style="zoom:50%;" />



由一个初值都为零的bit数组和多个哈希函数构成，用于**在大数据量下判断具体数据是否存在**

底层结构：set



## 布隆过滤器的特点

1. 高效地插入和查询，占用空间少，**返回的结果是不确定性的。**

2. 被布隆过滤器判断为存在的元素有可能是不存在的元素（误判），被布隆过滤器判断为不存在的元素那就一定是不存在的元素

3. 布隆过滤器可以添加元素，但是不能删除元素。因为删掉元素会导致误判率增加。

   

##  布隆过滤器的使用场景

**解决缓存穿透的问题**

缓存穿透是什么？ 一般情况下，先查询缓存redis是否有该条数据，缓存中没有时，再查询数据库。 当数据库也不存在该条数据时，每次查询都要访问数据库，这就是缓存穿透。 

缓存透带来的问题是，当有大量请求查询数据库不存在的数据时，就会给数据库带来压力，甚至会拖垮数据库。  

可以使用布隆过滤器解决缓存穿透的问题 ，把已存在数据的key存在布隆过滤器中，相当于mysql前面挡着一个redis，redis前面挡着一个布隆过滤器。 

 

当有新的请求时，先到布隆过滤器中查询是否存在：

- 如果布隆过滤器中不存在该条数据则直接返回； 

- 如果布隆过滤器中已存在，才去查询缓存redis，如果redis里没查询到则穿透到Mysql数据库 

  

**黑名单校验**

发现存在黑名单中的，就执行特定操作。比如：识别垃圾邮件，只要是邮箱在黑名单中的邮件，就识别为垃圾邮件。假设黑名单的数量是数以亿计的，存放起来就是非常耗费存储空间的，布隆过滤器则是一个较好的解决方案。 把所有黑名单都放在布隆过滤器中，在收到邮件时，判断邮件地址是否在布隆过滤器中即可。 

**网页爬虫对URL的去重，避免爬取相同的URL地址**

**反垃圾邮件，从数十亿个垃圾邮件列表中判断某邮箱是否垃圾邮箱**



## 布隆过滤器的原理

布隆过滤器(Bloom Filter) 是一种专门用来解决去重问题的高级数据结构。 

**实质就是 一个大型 位数组** 和几个不同的无偏hash函数 (无偏表示分布均匀)。由一个初值都为零的bit数组和多个个哈希函数构成，用来快速判断某个数据是否存在 。但是跟 HyperLogLog 一样，它也一样有那么一点点不精确，也存在一定的误判概率 



**添加key时** 

使用多个hash函数对key进行hash运算得到一个整数索引值，对位数组长度进行取模运算得到一个位置， 每个hash函数都会得到一个不同的位置，将这几个位置都置1就完成了add操作。 

 当有变量被加入集合时，通过N个映射函数将这个变量映射成位图中的N个点， 把它们置为 1（假定有两个变量都通过 3 个映射函数）。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730152339914.png" alt="image-20210730152339914" style="zoom:50%;" />

当我们向布隆过滤器中添加数据时，**为了尽量地址不冲突， 会使用多个 hash 函数对 key 进行运算** ，算得一个下标索引值，然后对位数组长度进行取模运算得到一个位置，每个 hash 函数都会算得一个不同的位置。再把位数组的这几个位置都置为 1 就完成了 add 操作。 

例如，我们添加一个字符串wmyskxz 

​												<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730152553072.png" alt="image-20210730152553072" style="zoom:33%;" /> 

**查询key时** 

只要有其中一位是零就表示这个key不存在，但如果都是1， 则不一定存在对应的key。 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730152451187.png" alt="image-20210730152451187" style="zoom:50%;" />

**由哈希冲突引发的误判**

此时我们查询一个没添加过的不存在的字符串inexistent-key，由于哈希冲突，经过一系列的计算导致它的索引下标和之前添加过的元素的下标相同，造成误判



**布隆过滤器为什么不能删除元素？**



因为布隆过滤器的每一个 bit位并不是独占的，可能有多个元素共享了其中的某一个比特位。 

如果我们直接删除这一位的话，会影响其他的元素 ，有可能该位置被其他位映射过。布隆过滤器在判断是否存在key是要判断该key的所有映射是否为"1"，如果有一个不为"1"则该元素不存在。因此删除元素直接会导致布隆过滤器的误判！



**布隆过滤器中元素量巨大时误判率会升高，如何减少呢？**

方案：开发定时任务，每隔几个小时或几天，自动创建一个新的布隆过滤器数组，替换老的，有点`CopyOnWriteArrayList`

 

**布隆过滤器使用的注意事项**

- 使用时最好不要让实际元素数量远大于初始化数量
- 当实际元素数量超过初始化数量时，应该对布隆过滤器进行重建，重新分配一个 size 更大的过滤器，再将所有的历史元素批量 add 进行

**布隆过滤器的优缺点**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730161754851.png" alt="image-20210730161754851" style="zoom:50%;" />



# Redis.conf



## 熟悉基本配置

**配置文件位置**

Redis 的配置文件位于 Redis 安装目录下，文件名为 **redis.conf**

我们一般情况下，会单独拷贝出来一份进行操作。来保证初始文件的安全





**Units (单位)**



1. 配置大小单位，开头定义了一些基本的度量单位，只支持bytes，不支持bit
2. 对大小写不敏感



**INCLUDES 包含**

和Spring配置文件类似，可以通过includes包含，redis.conf 可以作为总文件，可以包含其他文件



**NETWORK 网络配置 **

```bash
bind 127.0.0.1   #绑定ip
protected-mode yes   #保护模式
port 6379   #使用默认端口
```



**GENERAL 通用 **

```bash
daemonize yes # 默认情况下，Redis不作为守护进程运行。需要开启的话，改为 yes 
supervised no # 可通过upstart和systemd管理Redis守护进程 
pidfile /var/run/redis_6379.pid # 以后台进程方式运行redis，则需要指定pid 文件 
loglevel notice # 日志级别。可选项有： 
					# debug（记录大量日志信息，适用于开发、测试阶段）；
					# verbose（较多日志信息）；
					# notice（适量日志信息，使用于生产环境）； 
					# warning（仅有部分重要、关键信息才会被记录）。
logfile "" # 日志文件的位置，当指定为空字符串时，为标准输出 
databases 16 # 设置数据库的数目。默认的数据库是DB 0 
always-show-logo yes # 是否总是显示logo
```

守护进程：就是后台进程，控制端启动了redis关闭后redis依然运行

**SNAPSHOPTING 快照**

```bash
save 900 1     # 900秒（15分钟）内至少1个key值改变（则进行持久化操作）
save 300 10    # 300秒（5分钟）内至少10个key值改变（则进行持久化操作）
save 60 10000  # 60秒（1分钟）内至少10000个key值改变（则进行持久化操作）


stop-writes-on-bgsave-error yes  # 持久化出现错误后，是否继续工作

rdbcompression yes  #是否压缩rdb文件

rdbchecksum yes     #保存rdb文件时, 是否校验

dbfilename dump.rdb  # dbfilenamerdb文件名称

rdb-del-sync-files no #rdb文件是否删除同步锁

dir ./			    # dir 数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录

```



**SECURITY安全 **

```bash
# 启动redis
# 连接客户端 
# 获得和设置密码 
config get requirepass
config set requirepass "123456" 
#测试ping，发现需要验证 
127.0.0.1:6379> ping NOAUTH Authentication required. 
# 验证
127.0.0.1:6379> auth 123456 OK127.0.0.1:6379> ping PONG
```



**限制**

```bash
maxclients 10000 # 设置能连上redis的最大客户端连接数量 
maxmemory <bytes> # redis配置的最大内存容量 
maxmemory-policy noeviction # maxmemory-policy 内存达到上限的处理策略
					      #volatile-lru：利用LRU算法移除设置过过期时间的key。 
					      #volatile-random：随机移除设置过过期时间的key。 
					      #volatile-ttl：移除即将过期的key，根据最近过期时间来删除（辅以TTL） 
					      #allkeys-lru：利用LRU算法移除任何key。
                            #allkeys-random：随机移除任何key。 
                            #noeviction：不移除任何key，只是返回一个写错误。
```



**append only模式**

```bash
appendonly no # 是否以append only模式作为持久化方式，默认使用的是rdb方式持久化，这种 方式在许多应用中已经足够用了
appendfilename "appendonly.aof" # appendfilename AOF 文件名称
appendfsync everysec
Auto-aof-rewrite-min-size # 设置重写的基准值,当前aof文件大于多少字节后才触发。避免在aof较小的时候无谓行为。默认大小为64mb。
Auto-aof-rewrite-percentage 100 #设置重写的基准值（当前写入日志文件的大小超过上一次rewrite之后的文件大小的百分之100时就是2倍时触发Rewrite）
```



## 常见配置介绍

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

   > daemonize no

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定

   > pidfile /var/run/redis.pid

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名port 6379

4. 绑定的主机地址

   > bind 127.0.0.1
   
5. 当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

   > timeout 300

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为
   verbose

   > loglevel verbose

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

   > logfile stdout

8. 设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id

   > databases 16

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
   save 
   Redis默认配置文件中提供了三个条件：

   > save 900 1
   > save 300 10
   > save 60 10000
   >
   > 分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

    > rdbcompression yes

11. 指定本地数据库文件名，默认值为dump.rdb

    > dbfilename dump.rdb

12. 指定本地数据库存放目录

    > dir ./

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master
    进行数据同步

    > slaveof

14. 当master服务设置了密码保护时，slav服务连接master的密码

    > masterauth

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH 命令提供密码，
    默认关闭

    > requirepass foobared

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可
    以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，
    Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

    > maxclients 128

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

    > maxmemory
    
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

    > appendonly no

19. 指定更新日志文件名，默认为appendonly.aof

    > appendfilename appendonly.aof

20. 指定更新日志条件，共有3个可选值：
    no：表示等操作系统进行数据缓存同步到磁盘（快）
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
    everysec：表示每秒同步一次（折衷，默认值）

    > appendfsync everysec

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将
    访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔
    细分析Redis的VM机制）

    > vm-enabled no

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

    > vm-swap-file /tmp/redis.swap

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据
    都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有
    value都存在于磁盘。默认值为0
    vm-max-memory 0

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多
    个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page
    大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用
    默认值

    > vm-page-size 32

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中
    的，，在磁盘上每8个pages将消耗1byte的内存。

    > vm-pages 134217728

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都
    是串行的，可能会造成比较长时间的延迟。默认值为4

    > vm-max-threads 4

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

    > glueoutputbuf yes

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

    > hash-max-zipmap-entries 64
    > hash-max-zipmap-value 512

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

    > activerehashing yes

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各
    个实例又拥有自己的特定配置文件

    > include /path/to/local.conf



# Redis 的持久化

Redis 是内存数据库，如果不将内存中的数据库状态保存到磁盘，那么一旦服务器进程退出，服务器中的数据库状态也会消失。所以 Redis 提供了持久化功能



## RDB(Redis DataBase)



**什么是RDB**

RDB其实就是一种内存快照，记录的是某一时刻的数据，**时点与时点间的数据**，以序列化的方式保存

什么是内存快照？内存快照指内存中的数据在某一个时刻的状态记录。这就类似于照片，当你给朋友拍照时，一张照片就能把朋友一瞬间的形象完全记下来。

它实现类似照片记录效果的方式，就是把某一时刻的状态以文件的形式写到磁盘上，也就是快照。这样一来，即使宕机，快照文件也不会丢失，数据的可靠性也就得到了保证。这个快照文件就称为 RDB 文件



RDB执行的是全量快照，也就是说，把内存中的所有数据都记录到磁盘中

**save和bgsave**

这两个都是触发RDB快照落盘的命令

- save：执行save命令时，Redis只进行持久化，其他操作全部阻塞。在明确要备份的时候使用
- bgsave：通过fork创建一个专门用于写入 RDB 文件的子进程，避免了主线程的阻塞



**什么是fork**

创建fork的时候会阻塞主线程，因此fork必须要快！

Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量，环境变量，程序计数器等）都和原进程一致，这样的一个全新的进程**作为原进程的子进程**。fork后的子进程虽然和父进程数据一样，**但是并不是对父进程数据的复制，而是父子进程的指针指向同一块内存区域**，相当于浅克隆。



### bgsave的写时复制copy-on-write

在linux中，有个父进程和子进程的机制，就是父进程可以fork出子进程，常规情况下父子进程的数据是相互隔离的，但是父进程其实是可以让子进程看到数据的。

fork创建子进程的时候并不发生数据复制，因为fork的时候会阻塞主进程，fork就必须要快！如果相同的数据被子进程拷贝一份即占内存又消耗性能，**因此子进程复制的不是数据，而是指针**，和父进程指向同一块内存区域。 

如图，父进程有个key名键名叫key1，指向内存中value为a，地址为0101的的区域，子进程也指向这一块区域。当对已存在的键key1进行修改时，那么这个时候父进程就会真正拷贝这个key对应的内存数据，申请新的内存空间，主进程指向新的内存空间并在新的内存空间中进行修改，同时，bgsave中fork的子进程可以继续把原来的数据写入 RDB 文件。毕竟我们在生成快照时是不希望快照动的（你平时的照相或自拍的时候会乱动吗？）

**好处：通过bgsave的写时复制既保证了快照的完整性，也允许主线程同时对数据进行修改，避免了对正常业务的影响。**

![image-20210914152333173](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210914152333173.png)

**总结:bgsave指令会fork出子进程进行快照的保存，fork过程是阻塞的，在快照保存期间修改key，父进程会复制出相同的数据对其修改**



配置文件中触发RDB条件的配置如下

![image-20210805211724365](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210805211724365.png)



RDB 是整合内存的压缩过的Snapshot，RDB 的数据结构，可以配置复合的快照触发条件。 

**默认是：**

- 1分钟内修改超过1万次触发rdb

- 5分钟内修改超过10次 触发rdb

- 15分钟内修改1次触发rdb

如果想禁用RDB持久化，可以将save的配置删掉不设置，或者给save传入一个空字符串参数    `save ""`。 

若要想修改完毕要立马生效，可以在控制台动使用 save 命令。

**注：如果按照上面的配置300s内修改了12次，那么这300s的时间窗口内会触发几次RDB?** 

答：1次，11，12这两次会重新参与计数



**SNAPSHOPTING 快照，命令深度解析**

- Stop-writes-on-bgsave-error：如果配置为no，表示你不在乎数据不一致或者有其他的手段发现和控制，默认为yes。 

- rbdcompression：对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会LZF算法进行压缩，如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。 

- rdbchecksum：在存储快照后，还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能。默认为yes。



**快照的恢复**

将备份过的dump.rdb文件移动到redis安装目录(这里是/usr/local/bin)并启动服务即可 。换言之，在哪个目录启动的redis就把dump.rdb移动到哪个目录中。



**RDB的优缺点**

- 优点

  适合大规模的数据恢复，类似于Java中的序列化，数据恢复快 

  

- 缺点

  RDB是在一定时间间隔做一次备份，如果redis意外挂掉的话，就会丢失最后一次快照后的所有修改 ，时点与时点间的数据易丢失





## AOF（Append Only File）

**什么是AOF**

Aof保存的文件叫 appendonly.aof 

AOF 文件是**以追加的方式**，逐一记录接收到的写操作命令

AOF 日志是写后日志，“写后”的意思是 Redis 是先执行命令，把数据写入内存，然后才记录日志，这样的好处是不会阻塞当前的写操作。

<img src="https://static001.geekbang.org/resource/image/40/1f/407f2686083afc37351cfd9107319a1f.jpg" alt="img" style="zoom:15%;" />





**AOF里面记录了啥？**

AOF 里记录的是 Redis 收到的每一条命令，这些命令是以文本形式保存的。

我们以 Redis 收到“set testkey testvalue”命令后记录的日志为例，看看 AOF 日志的内容。其中，“*3”表示当前命令有三个部分，每部分都是由“$+数字”开头，后面紧跟着具体的命令、键或值。这里，“数字”表示这部分中的命令、键或值一共有多少字节。例如，“$3 set”表示这部分有 3 个字节，也就是“set”命令。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806145819250.png" alt="image-20210806145819250" style="zoom:50%;" />

**AOF的三种写回策略**

**AOF日志是由主线程来写的，因为是写后日志，虽然不会阻塞当前操作，但有可能阻塞下一个操作**，因此为了保证Redis的高性能有必要选择一个最佳的写回策略：

- Always，同步写回：每个写命令执行完，立马同步地将日志写回磁盘；

- Everysec，每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；

- No，操作系统控制的写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘

  

  同步写回：基本不会丢失数据；

  操作系统写回：落盘时机不在Redis手里，操作系统说了算，如果AOF记录没写回磁盘一旦宕机就会导致丢失数据

  每秒写回：最多丢失一秒的数据，相当于前两种策略的折中

  <img src="https://static001.geekbang.org/resource/image/72/f8/72f547f18dbac788c7d11yy167d7ebf8.jpg" alt="img" style="zoom:25%;" />



**AOF配置**

Redis中RDB和AOF可以同时开启，如果开启AOF就只用AOF恢复

```bash
appendonly no # redis默认使用的是rdb方式持久化的，如果改成yes就使用aof方式进行持久化
appendfilename "appendonly.aof" # appendfilename AOF 文件名称
Auto-aof-rewrite-min-size # 设置重写AOF的触发条件，超过指定的大小触发AOF的重写，最小64M
Auto-aof-rewrite-percentage 100 #设置重写的基准值。当前写入日志文件的大小超过上一次重写之后的文件大小的100%时（2倍时）触发
```

- replica-serve-stale-data yes
  - 就是当从挂掉了重新启动的时候，要从主机恢复数据，如果打开了，代表恢复的过程中也可以提供服务，关闭的话则要阻塞
- replica-read-only yes
  - 配置从节点是否只读,但是配置的修改还是可以的
- repl-diskless-sync no
  - redis由主向从正常情况下要经历一次主机redis到RDB，再把RDB通过网络IO发到从redis两次IO
  - 这个标记打开代表可以不经历RDB直接通过网络把数据发到从机上面
- repl-backlog-size 1mb
  - 增量复制
  - 先通过RDB传输，然后通过一个小的队列，来维护在两个RDB之间的数据
  - 要注意业务场景 调整一个合适的大小 有可能挂了没多久，但是这个队列已经冒了
- min-replicas-to-write 3
  - 最少写几个写成功
- min-replicas-max-lag 10



**AOF的弊端**

AOF的优点很明显：可以尽最大可能的减少因宕机和异常导致的数据丢失，**数据丢失少**。

但同时也存在缺点：

1. AOF是由主线程来写
2. AOF的体量可以无限大，这就会导致：

- 文件太大追加数据效率变低，
- 恢复数据缓慢

如何优化？ AOF的重写机制



**AOF的重写机制**

AOF的重写机制就是为了保证AOF文件要足够小，减少冗余。

比如

在list中进行了无数次的push和pop，最终会重写成一个操作，避免流水账似的记录

<img src="https://static001.geekbang.org/resource/image/65/08/6528c699fdcf40b404af57040bb8d208.jpg" alt="img" style="zoom:15%;" />

重写原理：

- **Redis4.0以前：重写原理就是将新增删除命令抵消，合并重复的命令，最终是一个纯指令文件**
- **Redis4.0以后：将老数据的RDB文件以指令的方式写入AOF中。（RDB到AOF中）**

每次 AOF 重写时，Redis 会先执行一个内存拷贝，复制了一个新的aof文件进行重写，这样的好处是保证新写入的数据不会丢失。而且Redis 采用额外的线程进行数据重写，所以这个过程并不会阻塞主线程，**可以和RDB的bgsave写时复制一同理解**

<img src="https://static001.geekbang.org/resource/image/6b/e8/6b054eb1aed0734bd81ddab9a31d0be8.jpg" alt="img" style="zoom:15%;" />

什么时候触发AOF重写？

- Auto-aof-rewrite-min-size # 设置重写AOF的触发条件，超过指定的大小触发AOF的重写，最小64M
- Auto-aof-rewrite-percentage 100 #设置重写的基准值。当前写入日志文件的大小超过上一次重写之后的文件大小的100%时（2倍时）触发

**AOF 启动/修复/恢复 **

- 正常恢复： 
  1. 启动：设置Yes，修改默认的appendonly no，改为yes 
  2. 将有数据的aof文件复制一份保存到对应目录（config get dir） 
  3. 恢复：重启redis然后重新加载 

- 异常恢复： 
  1. 启动：设置Yes，修改默认的appendonly no，改为yes
  2. 故意破坏 appendonly.aof 文件！ 
  3. 修复： 使用命令：`redis-check-aof --fix appendonly.aof` 进行修复 
  4. 恢复：重启 redis 然后重新加载 







# Redis事务



## 理论 



### Redis事务的概念：

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程中，**会按照顺序串行化执行队列中的命令**，其他客户端提交的命令请求不会插入到事务执行命令序列中。 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806211005801.png" alt="image-20210806211005801" style="zoom:50%;" />

总的来说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

 

**redis事务的特性**

- Redis事务没有隔离级别的概念
- Redis不保证原子性
- Redis的事务不支持回滚，事务中其中一条指令失败其他指令照常执行



**Redis事务相关命令：**

- watch key1 key2 ... :监视一或多个key,如果在事务执行之前，被监视的key被其他命令改动，则事务被打断
- multi : 标记一个事务块的开始（ queued ） 
- exec ：执行所有事务块的命令 （ 一旦执行exec后，之前加的监控锁都会被取消掉 ） 
- discard ： 取消事务，放弃事务块中的所有命令 
- unwatch ： 取消watch对所有key的监控

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806202810237.png" alt="image-20210806202810237" style="zoom:39%;" />

### 实践

1. 正常执行命令

   开启事务 ---> 命令入队--->执行事务--->数据被写入

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806171214987.png" alt="image-20210806171214987" style="zoom:30%;" />

2. 放弃事务

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806171258814.png" alt="image-20210806171258814" style="zoom:30%;" />

   

3.  若在事务队列中存在命令性错误（类似于java编译性错误），则执行EXEC命令时，所有命令都不会执行 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806171406769.png" alt="image-20210806171406769" style="zoom:30%;" />

   

4. 若在事务队列中存在语法性错误（类似于java的1/0的运行时异常），则执行EXEC命令时，其他正确命令会被执行，错误命令抛出异常。

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806171432983.png" alt="image-20210806171432983" style="zoom:33%;" />



Redis事务的核心就是将事务放入队列，将队列中的任务串行执行

### watch监控

可以监视一个(或多个) key，如果key 被其他会话中的命令所改动事务将被打断

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806210535343.png" alt="image-20210806210535343" style="zoom:50%;" />

# Redis发布订阅

## 什么是Redis发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。 Redis 客户端可以订阅任意数量的频道。



## 命令

这些命令被广泛用于构建即时通信应用，比如网络聊天室(chatroom)和实时广播、实时提醒等。

**测试**

创建频道名为redisChat

```bash
127.0.0.1:6379> SUBSCRIBE redisChat
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "redisChat"
3) (integer) 1

```

开启另一个新端口，在频道redisChat发布两个消息，另一个端口就能接收到消息

```bash
127.0.0.1:6379> publish redisChat HelloWorld
(integer) 1
127.0.0.1:6379> publish redisChat HelloWorld123
(integer) 1  
```



```bash
# 接收到的消息
1) "subscribe"
2) "redisChat"
3) (integer) 1
1) "message"
2) "redisChat"
3) "HelloWorld"
1) "message"
2) "redisChat"
3) "HelloWorld123"
```

**原理**（了解）

Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。通过 SUBSCRIBE 命令订阅某频道后，redis-server 里维护了一个字典**，字典的键就是一个个 channel，而字典的值则是一个链表，链表中保存了所有订阅这个 channel 的客户端。**SUBSCRIBE 命令的关键，就是将客户端添加到给定 channel 的订阅链表中。 通过 PUBLISH 命令向订阅者发送消息，redis-server 会使用给定的频道作为键，在它所维护的 channel 字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者。Pub/Sub 从字面上理解就是发布（Publish）与订阅（Subscribe）

使用场景 

- Pub/Sub构建实时消息系统 
- Redis的Pub/Sub系统可以构建实时的消息系统，比如很多用Pub/Sub构建的实时聊天系统的例子。 





# Redis双写一致性

只要用缓存，就可能会涉及到缓存与数据库双存储双写，只要是双写，就一定会有数据一致性的问题，那么如何解决一致性问题？

**分布式系统只有最终一致性，很难做到强一致性**

## cannel

工作原理：同mysql主从复制原理：读取mysql的binlog日志

canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave（mysql从机） ，向 MySQL master 发送dump 协议 

MySQL master 收到 dump 请求，开始推送 binary log 给  canal ，canal 解析 binary log 对象(原始为 byte 流) 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804162209729.png" alt="image-20210804162209729" style="zoom:50%;" />



## 数据库和缓存一致性更新的策略

1. 先更新数据库，再更新Redis缓存

   问题：假设更新完数据库，然后更新Redis缓存出现异常失败了，这就会导致Redis中的数据是以前的旧数据，**导致读到脏数据**

2. 先删除缓存，再更新数据库

   问题：

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804165155508.png" alt="image-20210804165155508" style="zoom:40%;" />

   解决方案：延时双删

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804165411307.png" alt="image-20210804165411307" style="zoom:50%;" />

   线程Asleep的时间，就需要大于线程B读取数据再写入缓存的时间。 

   **这个时间怎么确定呢？** 

   在业务程序运行的时候，统计下线程读数据和写缓存的操作时间，自行评估自己的项目的读数据业务逻辑的耗时， 

   以此为基础来进行估算。然后写数据的休眠时间则在读数据业务逻辑的耗时基础上加 百毫秒 即可。 

   目的就是确保读请求结束，写请求可以删除读请求造成的缓存脏数据。 

   **问：这种同步淘汰策略，如果吞吐量降低怎么办？**

   使用CompletableFuture再开一个线程进行**异步删除**

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804174856520.png" alt="image-20210804174856520" style="zoom:50%;" />

   

3. 先更新数据库，再删除缓存

   问题1：假如缓存删除失败或者来不及，导致请求再次访问redis时缓存命中， 读取到的是缓存旧值。 

   <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804173810088.png" alt="image-20210804173810088" style="zoom:50%;" />

   解决方案：

   1. 把要删除的缓存值或者是要更新的数据库值暂存到消息队列中（Kafka/RabbitMQ等）。

   2. 当程序删除失败或更新失败，可以从消息队列中重新读取这些值，然后再次尝试删除或更新。 

   3. 如果删除或更新成功，把值从消息队列中去除，避免重复操作

   4. 如果重试超过的一定次数后还是没有成功，我们就需要向业务层发送报错信息了，通知运维人员。 

      <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210804174613666.png" alt="image-20210804174613666" style="zoom:40%;" />

总结：建议优先 **使用先更新数据库，再删除缓存的方案** 。理由如下： 

- 先删除缓存值再更新数据库，有可能导致请求因缓存缺失而访问数据库，给数据库带来压力，导致缓存穿透。 
- 业务应用中读取数据库和写入缓存的时间不好估算，延迟双删中的等待时间就不好设置。



最后要说明，真的要落地，就用canal吧！



# Redis集群

**为什么需要Redis集群？**

单机单实例Redis的弊端有：

- 易发生单点故障
- 单机容量有限
- 读写并发过高可能导致压力过大

**因此，使用redis集群保证了高可用、缓解高并发情况下单台redis服务器的压力**



## Redis的AKF原理

afk是一种保障服务的策略，将集群的拓展分为x、y、z三个方向

- x：全量镜像，当一台redis容易故障的时候，则增加几台备用机

- y：按业务功能扩展，多个功能分布在不同的Redis中

- z：一个业务在一台redis中内容也太多了，按照逻辑拆分

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806223846946.png" alt="image-20210806223846946" style="zoom:50%;" />

**通过AKF一台Redis变成多台Redis会发生什么问题？**

数据的一致性！！！

我们的目的就是要最终所有节点的数据都做到一致，但redis能做到的只能是最终一致（AP)

- 强一致性

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806224441562.png" alt="image-20210806224441562" style="zoom:50%;" />

  如果想要做到强一致性，向主机写入数据，从机同步的过程中会发生阻塞，同步的过程中从机不可用---强一致性破坏可用性

- 弱一致性

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806232454747.png" alt="image-20210806232454747" style="zoom:50%;" />

  主从的同步是异步的，也就是说在满足从机可用的前提下，主机向从机发送同步的消息后就返回（该过程是异步非阻塞的）

  缺点：易发生数据的丢失

  改进：使用消息中间件做到数据的最终一致性

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210806232803786.png" alt="image-20210806232803786" style="zoom:40%;" />

  

## 主从复制

Redis提供了主从库模式，以保证数据副本的一致，主从库之间采用的是读写分离的方式。

有了读写分离，在主机上写数据，在从机上读数据。主库修改数据后将数据同步到从库，实现数据的同步

### 主从复制原理

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210807162646744.png" alt="image-20210807162646744" style="zoom:50%;" />



**主库执行bgsave命令，生成RDB文件，接着将文件发给从库。从库接收到RDB文件后，会先清空当前数据库，然后加载RDB文件**。这是因为从库在通过replicaof命令开始和主库同步前，可能保存了其他数据。为了避免之前数据的影响，从库需要先把当前数据库清空。

在主库将数据同步给从库的过程中，主库不会被阻塞，仍然可以正常接收请求。否则，Redis的服务就被中断了。但是，这些请求中的写操作并没有记录到刚刚生成的RDB文件中。为了保证主从库的数据一致性，主库会在内存中用专门的replication buffer，记录RDB文件生成后收到的所有写操作。

最后，也就是第三个阶段，主库会把第二阶段执行过程中新收到的写命令，再发送给从库。具体的操作是，当主库完成RDB文件发送后，就会把此时replication buffer中的修改操作发给从库，从库再重新执行这些操作。这样一来，主从库就实现同步了。

后序的增量主从同步全都是基于repl_backlog_buffer缓冲区



**主从复制原理总结：**

1. 建立连接
2. fork出子进程生成RDB，发给从库全量同步（主库向从库同步过程是fork出一个子进程，不阻塞主进程）
3. 同步过程中主库修改操作会记录在repl_backlog_buffer这个缓冲区
4. 同步完成后再把缓冲区中的操作发给从库进行增量同步
5. 后序增量同步全部基于**repl_backlog_buffer缓冲区**









## Redis集群模式

- 一主多从

- 多主多从

- 主--从--从（层层链路）

  **主从从这种层层链路集群模式的好处：**

  试想一下如果一个主机下面有大量的从机，在第一次进行全量复制时会通过fork出子进程生成RDB并发送到从机。如果从机过多会大量fork出子进，每一个子进程发送到每一个从机，程造成阻塞影响主线程处理正常请求，且传输RDB给大量的从机也会受网络带宽的限制。**层层链路分担了主机生成RDB和传输RDB的压力**



如何配置：

通过replicaof，命令行配置即可

```bash
redis-server redis6380.conf --replicaof 192.168.31.180 6379
```



## 哨兵模式



**哨兵模式配置**

举例

```bash
# 进入redis安装目录
[root@iZ2ze9te1lxhycarjfa8rlZ myredis]# vim sentinel26379.conf    #创建哨兵配置文件
```



<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210807171626728.png" alt="image-20210807171626728" style="zoom:50%;" />



```bash
[root@iZ2ze9te1lxhycarjfa8rlZ myredis]# redis-sentinel sentinel.conf  # 设置那两行配置并启动
```



**哨兵的作用**

在主从模式的集群中，没有哨兵，当主机挂掉了，需要人工维护。有了哨兵机制可以有效的监控主机的故障，并及时选出新的主机。

哨兵是一个运行在特殊模式下的Redis进程，主从库实例运行的同时，它也在运行。哨兵主要负责的就是三个任务：**监控、选主和通知。**

<img src="https://static001.geekbang.org/resource/image/ef/a1/efcfa517d0f09d057be7da32a84cf2a1.jpg" alt="img" style="zoom:20%;" />

- 监控：

  哨兵进程会不断的发送PING命令检测它自己和主、从库的网络连接情况，用来判断实例的状态，如果主库也没有在规定时间内响应哨兵的PING命令，哨兵就会判定主库下线

- 选主：

  主库挂了以后，哨兵就需要从很多个从库里，按照一定的规则选择一个从库实例，把它作为新的主库

- 通知：

  哨兵会把新主库的连接信息发给其他从库，让它们重新执行replicaof命令，和新主库建立连接，并进行数据复制。同时，哨兵会把新主库的连接信息通知给客户端，让它们把请求操作发到新主库上。



**主观下线和客观下线**

既然Redis要保持高可用，哨兵也如此，生产中一般会以哨兵集群的方式部署。

由于网络的延迟抖动，哨兵难免会对主机发生误判，在哨兵集群模式中，一个哨兵判断主库是否下线不好使，这时候少数哨兵判断主库下线的结果就是主观下线，大多数哨兵判断主库下线才能真正的下线，这叫客观下线。

<img src="../../../Users/liqun/Library/Application Support/typora-user-images/image-20210807211824399.png" alt="image-20210807211824399" style="zoom:50%;" />

哨兵通过发布订阅发现其他哨兵

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210807220024685.png" alt="image-20210807220024685" style="zoom:50%;" />

进入主机端查看订阅命令

```bash
PSUBSCRIBE *
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210807224328261.png" alt="image-20210807224328261" style="zoom:50%;" />

哨兵与哨兵之间通过发布订阅进行通信



将原有的主机挂掉，info replication 查看， 发现哨兵自动选出了个新的主机，查看哨兵配置文件，之前设置的是监控6379端口，通过选举出新主机，配置文件端口被自动修改了

**注：**如果之前的master 重启回来，就不属于这个集群了。



**哨兵模式的优缺点**

**优点** 

1. 哨兵集群模式是基于主从模式的，所有主从的优点，哨兵模式同样具有。 
2. 主从可以切换，故障可以转移，系统可用性更好。 
3. 哨兵模式是主从模式的升级，系统更健壮，可用性更高

**缺点** 

1. Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。 
2.  实现哨兵模式的配置也不简单，甚至可以说有些繁琐 



# 缓存穿透、缓存击穿、缓存雪崩

Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。目前，业界也都有比较流行的解决方案。 

## 缓存穿透

### 概念

用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向数据库查询。发现也没有，于是本次查询失败。此时，应用也无法从数据库中读取数据再写入缓存。当并发量极高时，缓存全部都没有命中，于是都去请求了数据库。缓存也就成了“摆设”。大量这样的并发请求会造成数据库压力激增。

**简单说就是说用户要查询的内容既不在Redis缓存中也不在数据库中**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210809231646110.png" alt="image-20210809231646110" style="zoom:30%;" />



**什么时候会发生缓存穿透**

- 业务层误操作，业务逻辑存在bug：缓存中的数据和数据库中的数据被误删除了，所以缓存和数据库中都没有数据；
- 恶意攻击：专门访问数据库中没有的数据。



### 解决方案



#### 方案1：空对象缓存或者缺省值

redis中查不到，数据库中也查不到，那我们就设一个缺省值吧，比如我们要查id为200的商品信息，redis和mysql都没查到，那就在redis中设一个 product200 - 0;

**弊端**

黑客会对你的系统进行攻击，拿一个不存在的id 去查询数据，当使用大量id相同的值打到系统时起到了良好的防御作用，但是使用大量的不重复的id且redis和mysql都不存在的数据打到系统时就会生成大量空对象缓存，空对象缓存会越积越多造成无用内存的堆积

解决方法：设置空对象缓存过期时间



#### 方案2：布隆过滤器解决缓存穿透

Guava 中布隆过滤器的实现算是比较权威的，所以实际项目中我们不需要手动实现一个布隆过滤器



**googgle的Guava案例测试**

```java
		 @Test
    void test100w() {
        int dataSize = 1000000; // 存放100w个数据
        BloomFilter<CharSequence> filter = BloomFilter.create(Funnels.stringFunnel(Charset.defaultCharset()), dataSize);
        for (int i = 1; i <= dataSize; i++) {
            filter.put("hello" + i);
        }
        int existWrongJudge = 0;
        int notExistWrongJudge = 0;
      	// 判断已存在的数据
        for (int i = 1; i <= dataSize; i++) {  
            if (!filter.mightContain("hello" + i))
                existWrongJudge++;
        }
      	// 判断未存在的数据
        for (int i = dataSize + 1; i <= (dataSize + 100000); i++) { 
            if (filter.mightContain("hello" + i))
                notExistWrongJudge++;
        }
        System.out.println("存在的数据总共误判了" + existWrongJudge + "个");
        System.out.println("不存在的数据总共误判了" + notExistWrongJudge + "个");
    }
```

```
存在的数据总共误判了0个
不存在的数据总共误判了2893个
```

这块之前懵逼了好久，明明原理上说的判断存在的有可能存在，判断不存在的就一定不存在，这里怎么反了？

这里注意了：原理说的判断是基于布隆过滤器的判断，而不是你的主观判断！看第三个for里面，100w~110w那一段中的数据在布隆过滤器是不存在的，布隆过滤器判断到有的数据存在----------说明布隆过滤器判断到存在的数据有可能不存在，印证了原理。

使用布隆过滤器的整体流程如下：

![image-20210914163104262](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210914163104262.png)



**源码剖析**

默认情况下底层的构造器

```java
public static <T> BloomFilter<T> create(Funnel<? super T> funnel, long expectedInsertions) {
    return create(funnel, expectedInsertions, 0.03); // FYI, for 3%, we always get 5 hash functions
  }
```

误判容错率为0.03，默认写好的，而且默认是有5个哈希函数进行计算，比特位数为7298440bit

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730234210308.png" alt="image-20210730234210308" style="zoom:30%;" />

当我们吧容错率写为0.01时，容错率变小了，判断精度要求高了，比特位也变高了，需要计算的hash函数也变多了。

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210730234041147.png" alt="image-20210730234041147" style="zoom:30%;" />

**总结：**并不是容错率越低越好，容错率越低系统执行所需的时间越长，当容错率写成很低很低时容易将系统卡死，因此生产中尽量不要低于0.01



**Guava的缺点**：Guava 提供的布隆过滤器的实现还是很不错的 （想要详细了解的可以看一下它的源码实现），但是它有一个重大的缺 陷就是只能单机使用  ，而现在互联网一般都是分布式的场景。 

 为了解决这个问题，我们就需要用到 Redis 中的布隆过滤器了 



**案例：白名单过滤器**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731093856179.png" alt="image-20210731093856179" style="zoom:40%;" />

全部合法的key都需要放入过滤器+redis里面，不然数据就是返回null



**流程总结**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731102630844.png" alt="image-20210731102630844" style="zoom:50%;" />





## 缓存击穿

**概念**

在高并发的前提下，大量的请求同时查询一个热点 key 时，热点key突然失效了，导致大量的请求都打到了数据库上面，造成某一时刻数据库请求量过大，压力剧增。

还有一种情况：有一种key，平时从来没有被访问过，Redis中也没有缓存这个key，突然客户端中的大量请求访问这个key





**解决方案 **

- 对于访问频繁的热点key，干脆就不设置过期时间
- **双重检查锁锁防止缓存击穿(DCL)**

```java
  public User findUserById(Integer id) {
        User user = null;
        // 先从redis中查
        user = (User) redisTemplate.opsForValue().get(CACHE_KEY_USER + id);
        // 如果redis中没有该数据，双重检查锁，防止缓存穿透
        if (user == null) {
            synchronized (User.class) {
                user = userMapper.selectById(id);
                if (user == null) return null;// 如果从数据库中查到的user为空就直接返回空
                else redisTemplate.opsForValue().set(CACHE_KEY_USER + user.getId(), user);
            }
        }
        return user;
    }
```

- **互斥更新防止缓存击穿**

**demo案例**

```java
@Service
@Slf4j
public class ProductServiceImpl implements IProductService {
    @Resource
    private RedisTemplate redisTemplate;

    @Override
    public List<Product> listProduct(int page, int size) {
        List list = redisTemplate.opsForList().range(JHS_KEY, page, size);
        if (CollectionUtils.isEmpty(list)){
            log.info("------------------mysql查询------------------------");
        }
        return list;
    }

    @PostConstruct
    public void initJHS() {
        log.info("启动定时器淘宝聚划算功能。。。。。。。" + DateUtil.now());
        new Thread(() -> {
            while (true){
                List<Product> productList = this.initProduct();
                this.redisTemplate.delete(JHS_KEY);
                this.redisTemplate.opsForList().leftPushAll(JHS_KEY,productList);
                try {
                    TimeUnit.MINUTES.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.info("定时刷新");
            }
        }, "thread1").start();
    }

    public List<Product> initProduct() {
        List list = new ArrayList<Product>();
        for (int i = 1; i <= 20; i++) {
            Random random = new Random();
            int productId = random.nextInt(2000);
            Product product = new Product((long) productId, "product" + productId, 999 + productId, "detail.....");
            list.add(product);
        }
        return list;
    }
}
```

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731112155031.png" alt="image-20210731112155031" style="zoom:39%;" />

解决方案：设定两个缓存，先更新A缓存再更新B缓存互斥更新，查询时A缓存没有再查B，这样失效时间也会不一样

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731112454664.png" alt="image-20210731112454664" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731113051830.png" alt="image-20210731113051830" style="zoom:50%;" />

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731113120617.png" alt="image-20210731113120617" style="zoom:40%;" />



## 缓存雪崩

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210809225928218.png" alt="image-20210809225928218" style="zoom:33%;" />

**概念**

**大量的key同时失效、或redis宕机，间接造成大量访问到达数据库。**引起数据库压力过大甚至宕机。和缓存击穿不同的是， 缓存击穿指并发查同一条数据，缓存雪崩是大量不同数据都过期了，很多数据都查不到从而查数据库。



**什么时候易发生**

- 常出现于零点时刻大量的当日的key集中过期
- redis宕机，无法处理缓存请求，这就会导致大量请求一下子积压到数据库层



**解决方案：**

- **差异失效时间，将过期时间微调**

  对于集中失效的缓存，将他们失效时间随机增加个1~3min，，这样一来，不同数据的过期时间有所差别，但差别又不会太大，既避免了大量数据同时过期，同时也保证了这些数据基本在相近的时间失效，仍然能满足业务需求。

- **服务降级**

  发生缓存雪崩时，针对不同的数据采取不同的处理方式。

  - 当业务应用访问的是非核心数据（例如电商商品属性）时，暂时停止从缓存中查询这些数据，而是直接返回预定义信息、空值或是错误信息；

  - 当业务应用访问的是核心数据（例如电商商品库存）时，仍然允许查询缓存，如果缓存缺失，也可以继续通过数据库读取。

    <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210809230633920.png" alt="image-20210809230633920" style="zoom:25%;" />

**注：缓存击穿是基于一个key而言，缓存雪崩是基于大量的key而言---------缓存击穿是缓存雪崩的子集**

# redis分布式锁



## 单机锁和分布式锁的区别

对于在单机上运行的多线程程序来说，锁本身可以用一个变量表示。锁变量初始值为0，表示没有加锁

- 变量值为0时，表示没有线程获取锁；
- 变量值为1时，表示已经有线程获取到锁了。

和单机上的锁类似，**分布式锁同样可以用一个变量来实现**。客户端加锁和释放锁的操作逻辑，也和单机上的加锁和释放锁操作逻辑一致：**加锁时判断锁变量的值，根据锁变量值来判断能否加锁成功；释放锁时需要把锁变量值设置为0，表明客户端不再持有锁**。



**为什么要使用分布式锁？**

单机版同一个JVM虚拟机内，使用synchronized或者Lock接口是可以的，但是在分布式环境下不同个JVM虚拟机内，单机的线程锁机制不再起作用，**竞争的线程可能不在同一个节点上**

因此：**锁变量需要由一个共享存储系统来维护**，只有这样，多个客户端才可以通过访问共享存储系统来访问锁变量。相应的，**通过判断共享储存变量锁的值来加锁和解锁**



## 分布式锁的介绍

**问题引申**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731114236704.png" alt="image-20210731114236704" style="zoom:50%;" />



**分布式锁所具备的刚需条件**

1. 独占性：OnlyOne，任何时刻只能有且仅有一个线程持有
2. 防死锁：杜绝死锁，必须有超时控制机制或者撤销操作，有个兜底终止跳出方案
3. 不乱抢：防止张冠李戴，不能私下unlock别人的锁，只能自己加锁自己释放。
4. 重入性：同一个节点的同一个线程如果获得锁之后，它也可以再次获取这个锁。
5. **原子性**：分布式锁的加锁和释放锁的过程，涉及多个操作。所以，在实现分布式锁时，我们需要保证这些锁操作的原子性；
6. **可靠性**：共享存储系统保存了锁变量，如果共享存储系统发生故障或宕机，那么客户端也就无法进行锁操作了。在实现分布式锁时，我们需要考虑保证共享存储系统的可靠性，进而保证锁的可靠性



## 如何实现分布式锁



### 基于单节点实现分布式锁



#### Setnx和Del组合命令实现

存在的问题：

- **问题1：**如果其中一个客户端（微服务）加锁和解锁命令之间如果发生异常，其他客户端（微服务）永远无法得到锁

  解决办法：设置锁的过期时间

- **问题2：**设置了锁的过期时间，但是本线程的锁提早过期，导致自己线程的锁被其他线程删除（张冠李戴）

  <img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210809214953626.png" alt="image-20210809214953626" style="zoom:35%;" />

  解决办法：需要**能区分来自不同客户端的锁操作**，给锁设置一个变量值（例如使用UUID）在释放锁操作时，对锁的变量值进行判断看是不是属于自己这个客户端的，然后再删除

- **问题3：** 解决了问题1，2。发现对锁的删除前多了个判断，不符合锁操作的原子性。

  解决办法：将锁的多个操作命令合并成一个命令，**使用Lua脚本**实现锁的释放，保证锁操作的原子性
  
- **问题4：**我们只用了一个Redis实例来保存锁变量，如果Redis宕机锁就没了，需要通过Redis集群保证锁的可靠性

  

  

  总结：基于单节点最终选择使用Lua脚本释放锁，保证了锁操作的原子性，并设置锁的过期时间保证锁可被删除

  

### 基于多个Redis节点实现高可靠的分布式锁



在多节点中基于setnx的分布式锁有什么缺点？

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210809215603384.png" alt="image-20210809215603384" style="zoom:50%;" />

多节点的话就用Redisson ，使用RedLock + watchdog为锁续命





**redlock加锁原理**

redlock加锁原理：集群中超过一半的节点同步好了锁数据，就算加锁成功

多台master节点，非主从节点

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210916101851722.png" alt="image-20210916101851722" style="zoom:30%;" />



**redlock红锁的缺点：**

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210916105018528.png" alt="image-20210916105018528" style="zoom:50%;" />

# Redis缓存过期淘汰策略



**Redis作为数据库和作为缓存的区别？**

缓存是随着访问变化的热数据，要即使清掉冷数据保留热数据，缓存不需要数据的持久化落盘，需要有过期时间，毕竟内存有限

![image-20210802164405354](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210802164405354.png)

常见面试题

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731221821594.png" alt="image-20210731221821594" style="zoom:50%;" />

**redis默认内存多少，怎么查看，如何改默认内存**

打开redis配置文件，设置maxmemory参数，maxmemory是bytes字节类型，注意单位转换。 

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210731221917003.png" alt="image-20210731221917003" style="zoom:50%;" />

默认内存：如果配置文件不设置或者设置内存大小为0  ---> 则在64位操作系统不限大小，在32位操作系统中限制3GB

生产上：一般推荐Redis设置内存为最大物理内存的四分之三



**真要打满了会怎么样？如果Redis内存使用超出了设置的最大值会怎样？**

加入内存淘汰策略



**往redis里写的数据是怎么没了的?它如何删除的？**

如果一个键是过期的，那它到了过期时间之后是不是 马上 就从内存中被被删除呢？？ 

 如果回答yes， 立即删除 ，你自己走还是面试官送你？ 

如果不是，那过期后到底什么时候被删除呢？？是个什么操作？ 



## Redis缓存删除策略

- **惰性删除**

  服务器不会主动释放过期键，当一些客户端尝试访问它时，key会被发现并主动的过期删除。等下次访问过期数据时，进行删除，并返回不存在。

  被动删除的缺点是，它对内存是最不友好的 。因为有些过期的keys，永远不会访问他们，就永远不会被删除。 

  

- **定期删除**

  定时随机测试设置keys的过期时间。所有这些过期的keys将会从密钥空间删除。

  1. 每十秒随机测试20个带有过期时间的key

  2. 发现测试的key已经过期就删除

  3. 如果有带ttl的keys多于25%的keys过期，重复步骤1，2

     

     目的就是**让过期的keys占所有带ttl的keys的百分比低于25%**

  

  

  

## Redis缓存淘汰策略

- noeviction:不会驱逐任何key
- **allkeys-Iru:对所有key使用LRU算法进行删除**
- volatile-Iru:对所有设置了过期时间的key使用LRU算法进行删除
- allkeys-random:对所有key随机删除
- volatile-random:对所有设置了过期时间的key随机删除
- volatile-ttl:删除马上要过期的key

  

lru的底层实现基于LinkedList

