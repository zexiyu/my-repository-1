

# Nginx简介



## Nginx的概述，什么是Nginx

Nginx是一个高性能的 HTTP 服务器和反向代理服务器，是个服务器

**Nginx 可以作为静态页面的 web 服务器**，同时还支持 CGI 协议的动态语言，比如 perl、php等。可惜不支持 java。Java 程序只能通过与 tomcat 配合完成。

Nginx 专为性能优化而开发，性能是其最重要的考量,实现上非常注重效率 ，能经受高负载的考验



中国大陆使用nginx网站用户：百度、京东、新浪、网易、腾讯、淘宝等，因此Nginx的学习尤为重要！



## Nginx的优点

- **占有内存少 **

  一般情况下10000个非活跃的keep-alive连接仅消耗2.5M的内存

- **并发能力强 **

  nginx支持的并发连接上限取决于内存，10万远没封顶

- **高扩展性**

  它由多个不同功能、不同层次、不同类型且耦合度极高的模块组成，这种低耦合的设计，造就了它庞大的第三方模块

- **高可用性**

  每个worker进程相对独立，master进程在某个worker进程出错时能迅速拉起新的worker进程，nginx的可靠性来源于其核心框架代码的优秀设计、模块设计的简单性。官方提供的常用模块都很稳定。

## Nginx可以做什么事情

### 正向代理

什么是正向代理：比如我们想要访问谷歌，Twitter，Ins，油管等网站就需要翻墙，而在我们国内的网络中因为有一道墙（The Great Wall！），是不允许我们访问外网的，如果想访问外网，就需要vpn等代理软件，也就是通过代理服务器来实现翻墙，这就是正向代理！

Nginx 不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。正向代理：如果把局域网外的 Internet 想象成一个巨大的资源库，则局域网中的客户端要访问 Internet，则需要通过代理服务器来访问，这种代理服务就称为正向代理。

### 反向代理

拨打10086客服电话，可能一个地区的10086客服有几个或者几十个，你永远都不需要关心在电话那头的人是男的还是女的，漂亮的还是帅气的，你关心的是你的问题能不能得到专业的解答，你只需要拨通了10086的总机号码，电话那头总会有人会回答你。那么这里的10086总机号码就是我们说的反向代理。客户不知道真正提供服务人的是谁

反向代理隐藏了真实的服务端，当我们请求www. baidu.com的时候，就像拨打10086一样，背后能有成千上万台服务器为我们服务，但具体是哪一台，你不知道，也不需要知道，你只需要知道反向代理服务器是谁就好了，www. baidu.com就是我们的反向代理服务器，反向代理服务器会把请求转发到真实的服务器那里去。 Nginx就是性能非常好的反向代理服务器

两者的区别在于代理的对象不一样：**正向代理代理的对象是客户端，反向代理代理的对象是服务端**

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201204232235694.png" alt="image-20201204232235694" style="zoom: 50%;" />

### 负载均衡

在如今高并发、大数据访问量下，增加服务器物理配置来解决问题的办法已然行不通，只能横向增加服务器的数量，这时候集群的概念产生了，*单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器*，也就是我们所说的负载均衡

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201204233610912.png" alt="image-20201204233610912" style="zoom:50%;" />

### 动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，静态资源（如html、css）用Nginx解析，动态资源（如jsp、servlet）用tomcat解析，这样可以加快解析速度。降低原来单个服务器的压力。



# 安装Nginx



## 先安装pcre依赖

prce为Nginx运行所需的必须依赖

1. 联网下载 pcre 压缩文件依赖

   https://sourceforge.net/projects/pcre/files/

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201204234931395.png" alt="image-20201204234931395" style="zoom: 25%;" />

​	点击进去以后选择版本进行下载即可



2. 将压缩包拖进/usr/src/目录，进入目录解压压缩文件，使用命令 tar –xvf pcre2-10.35.tar.gz 

3. 执行  ./configure 

4. 去解压缩的pcre 目录下执行： make && make install

   <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205000235536.png" alt="image-20201205000235536" style="zoom:50%;" />

3.检查是否安装成功

![image-20201205000418153](/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205000418153.png)

4. 安装 openssl 、zlib 、 gcc 依赖

   ```bash
   yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
   ```



## 安装Nginx

1. 去官网下载：http://nginx.org/en/download.html

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205001103875.png" alt="image-20201205001103875" style="zoom: 25%;" />

后面的步骤和安装pcre基本一样，不要想得很难

2. 将压缩包拖进/usr/src/目录，进入目录解压压缩文件，使用命令 tar –xvf  [nginx压缩文件]
3. 在当前目录执行  ./configure 
4. 去解压缩的nginx目录下（/usr/local/nginx）执行： make && make install



**查看开放的端口号**

firewall-cmd --list-all

**设置开放的端口号**

firewall-cmd --add-service=http –permanent

sudo firewall-cmd --add-port=80/tcp --permanent

**重启防火墙**

firewall-cmd –reload

如果是阿里云服务器要去控制台开放80端口

浏览器输入linux服务器的ip即可进入nginx主页面

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205002552544.png" alt="image-20201205002552544" style="zoom: 50%;" />





# Nginx常用命令和配置文件



## Nginx常用命令

1. 启动命令

   在/usr/local/nginx/sbin 目录下执行 ./nginx 

2. 关闭命令

   在/usr/local/nginx/sbin 目录下执行 ./nginx -s stop 

3. 重新加载命令

   在/usr/local/nginx/sbin 目录下执行 ./nginx -s reload



# Nginx配置文件



## 位置

```bash
cd/usr/local/nginx/conf
```

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205094802801.png" alt="image-20201205094802801" style="zoom:50%;" />

对Nginx的使用都是对这个配置文件进行修改

```bash
#第一部分：全局块
worker_processes  1;

#第二部分：events块
events {
    worker_connections  1024;
}

#第三部分：http块
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;

    keepalive_timeout  65;

  server {
        listen       80;
        server_name  47.93.242.4;

        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

- 第一部分：全局块

  主要是设置一些影响 nginx 服务器整体运行的配置指令。

  包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

  比如上面第一行配置的：

  worker_processes  1;

  这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约

  

- 第二部分：events块

  events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

  

  ```bash
  events {
      worker_connections  1024;
  }
  ```

  上述例子就表示每个 work process 支持的最大连接数为 1024.

  这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

  

- 第三部分：Http块

  算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。http 块包括 **http 全局块**、**server 块**。

    - http 全局块
  
      http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。
  
  - server块
  
    为了节省互联网服务器硬件成本而产生，server块和虚拟主机有密切关系。
  
    每个 http 块可以包括多个 server 块，每个 server 块就相当于一个虚拟主机。而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。
  
       - 全局 server 块
    
         最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。
    
       - location 块
    
         一个 server 块可以配置多个 location 块。主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。
         
         访问nginx路径默认就会跳到这里来
         
         <img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210202100552820.png" alt="image-20210202100552820" style="zoom:50%;" />

# Nginx配置动静分离

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205153552444.png" alt="image-20201205153552444" style="zoom: 33%;" />



通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码 304，如果有修改，则直接从服务器重新下载，返回状态码 200。的工作原理

**准备工作**

在linux的根目录下准备个data文件夹，文件夹里准备image 和www文件夹

image文件夹里面放一个图片，www文件夹放一个html文件

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205154045694.png" alt="image-20201205154045694" style="zoom: 50%;" />

配置文件

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205154411943.png" alt="image-20201205154411943" style="zoom: 50%;" />

http://47.93.242.4/image/1.png

http://47.93.242.4/image/

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205154623541.png" alt="image-20201205154623541" style="zoom:33%;" />

http://47.93.242.4/www/a.html

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205154644385.png" alt="image-20201205154644385" style="zoom:50%;" />

# Nginx 工作原理



Nginx由内核和模块组成。

　　Nginx本身做的工作实际很少，当它接到一个HTTP请求时，它仅仅是通过查找配置文件将此次请求映射到一个location block，而此location中所配置的各个指令则会启动不同的模块去完成工作，因此模块可以看做Nginx真正的劳动工作者。通常一个location中的指令会涉及一个handler模块和多个filter模块（当然，多个location可以复用同一个模块）。handler模块负责处理请求，完成响应内容的生成，而filter模块对响应内容进行处理。

用户根据自己的需要开发的模块都属于第三方模块。正是有了这么多模块的支撑，Nginx的功能才会如此强大。

Nginx的模块从结构上分为核心模块、基础模块和第三方模块：

- 核心模块：HTTP模块、EVENT模块和MAIL模块
- 基础模块：HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块，
- 第三方模块：HTTP Upstream Request Hash模块、Notice模块和HTTP Access Key模块。

Nginx的模块从功能上分为如下三类：

- Handlers（处理器模块）。此类模块直接处理请求，并进行输出内容和修改headers信息等操作。Handlers处理器模块一般只能有一个。
- Filters （过滤器模块）。此类模块主要对其他处理器模块输出的内容进行修改操作，最后由Nginx输出。
- Proxies （代理类模块）。此类模块是Nginx的HTTP Upstream之类的模块，这些模块主要与后端一些服务比如FastCGI等进行交互，实现服务代理和负载均衡等功能。

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205141410696.png" alt="image-20201205141410696" style="zoom:50%;" />





master-worker工作机制

Nginx默认采用多进程工作方式，Nginx启动后，会运行一个master进程和多个worker进程。其中master充当整个进程组与用户的交互接口，同时对进程进行监护，管理worker进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。worker用来处理基本的网络事件，worker之间是平等的，他们共同竞争来处理来自客户端的请求。

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205142324723.png" alt="image-20201205142324723" style="zoom: 33%;" />

在创建master进程时，先建立需要监听的socket（listenfd），然后从master进程中fork()出多个worker进程，如此一来每个worker进程多可以监听用户请求的socket。一般来说，当一个连接进来后，所有在Worker都会收到通知，但是只有一个进程可以接受这个连接请求，其它的都失败，这是所谓的惊群现象。nginx提供了一个accept_mutex（互斥锁），有了这把锁之后，同一时刻，就只会有一个进程在accpet连接，这样就不会有惊群问题了。

先打开accept_mutex选项，只有获得了accept_mutex的进程才会去添加accept事件。nginx使用一个叫ngx_accept_disabled的变量来控制是否去竞争accept_mutex锁。ngx_accept_disabled = nginx单进程的所有连接总数 / 8 -空闲连接数量，当ngx_accept_disabled大于0时，不会去尝试获取accept_mutex锁，ngx_accept_disable越大，于是让出的机会就越多，这样其它进程获取锁的机会也就越大。不去accept，每个worker进程的连接数就控制下来了，其它进程的连接池就会得到利用，这样，nginx就控制了多进程间连接的平衡。

每个worker进程都有一个独立的连接池，连接池的大小是worker_connections。这里的连接池里面保存的其实不是真实的连接，它只是一个worker_connections大小的一个ngx_connection_t结构的数组。并且，nginx会通过一个链表free_connections来保存所有的空闲ngx_connection_t，每次获取一个连接时，就从空闲连接链表中获取一个，用完后，再放回空闲连接链表里面。一个nginx能建立的最大连接数，应该是worker_connections * worker_processes。当然，这里说的是最大连接数，对于HTTP请求本地资源来说，能够支持的最大并发数量是worker_connections * worker_processes，而如果是HTTP作为反向代理来说，最大并发数量应该是worker_connections * worker_processes/2。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20201205142728991.png" alt="image-20201205142728991" style="zoom: 33%;" />

**master-workers 的机制的好处**

首先，对于每个 worker 进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销，同时在编程以及问题查找时，也会方便很多。其次，采用独立的进程，可以让互相之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快启动新的worker 进程。当然，worker 进程的异常退出，肯定是程序有 bug 了，异常退出，会导致当前 worker 上的所有请求失败，不过不会影响到所有请求，所以降低了风险



**需要设置多少个** **worker**

Nginx 同 redis 类似都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，但每个进程里只有一个主线程，通过异步非阻塞的方式来处理请求， 即使是千上万个请求也不在话下。每个 worker 的线程可以把一个 cpu 的性能发挥到极致。所以 worker 数和服务器的 cpu数相等是最为适宜的。设少了会浪费 cpu，设多了会造成 cpu 频繁切换上下文带来的损耗。



**连接数** **worker_connection**

这个值是表示每个 worker 进程所能建立连接的最大值，所以，一个 nginx 能建立的最大连接数，应该是 worker_connections * worker_processes。当然，这里说的是最大连接数，对于HTTP 请 求 本 地 资 源 来 说 ， 能 够 支 持 的 最 大 并 发 数 量 是 worker_connections * worker_processes，如果是支持 http1.1 的浏览器每次访问要占两个连接，所以普通的静态访问最大并发数是： worker_connections * worker_processes /2，而如果是 HTTP 作为反向代理来说，最大并发数量应该是 worker_connections * worker_processes/4。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。

问题1：发送请求，占用了 woker 的几个连接数？
答案：2 或者 4 个

问题2：nginx 有一个 master，有四个 woker，每个 woker 支持最大的连接数 1024，支持的
最大并发数是多少？
普通的静态访问最大并发数是： worker_connections * worker_processes /2， 而如果是 HTTP 作为反向代理来说，最大并发数量应该是 worker_connections * worker_processes/4。