

# MVC是什么

MVC是model（模型） + video（视图） + controller（控制器）,是一种软件设计规范作用：降低视图层与业务逻辑层之间的耦合度

MVC并不是设计模式，而是一种架构模式



# 回顾servlet
1. 编写一个类继承HttpServlet，实现doPost和doGet，写入业务逻辑
2. 在web.xml中配置Servlet，如：
```xml
 <servlet>
			<!--起别名-->
      <servlet-name>HelloServlet</servlet-name>
			<!--全类名-->
      <servlet-class>com.zlq.servlet.HelloServlet</servlet-class>
</servlet>

<servlet-mapping>      
		<!--要和别名相同-->
    <servlet-name>HelloServlet</servlet-name>
		<!--设置访问路径-->
    <url-pattern>/helloServlet</url-pattern>
</servlet-mapping>
```
3. 编写java业务类，该类继承HttpServlet,实现doGet和doPost方法
```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getParameter("method");
        if (method.equals("add")){
            req.getSession().setAttribute("msg" ,"执行了add方法");
        }
        if (method.equals("delete")){
            req.getSession().setAttribute("msg","执行了delete方法");
        }
        req.getRequestDispatcher("/WEB-INF/jsp/Hello.jsp").forward(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }
```

#  第一个SpringMVC程序



害！弄了整整一天，这个SpringMVC 的helloWorld程序终于能跑起来了，主要问题是狂神的依赖有问题，
换个依赖就好了，总的来说这个问题也相当隐蔽了，还好解决了！
依赖用这个！

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```
总结下程序的写入步骤：
## 配置版
### 一. 确定导入SpringMVC的依赖后配置web.xml,注册DispatcherServlet

DispatcherServlet表示前置控制器，是整个SpringMVC的控制中心，用来接收或拦截用户发起的请求

1. 先引入命名空间
```xml
<?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
            version="4.0">
   </web-app>
```
2. 注册DispatcherServlet，并关联SpringMVC的配置文件
```xml
<servlet>
     <servlet-name>SpringMVC</servlet-name> 
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <!--关联SpringMVC的配置文件--> 
    <init-param> 
         <param-name>contextConfigLocation</param-name> 
         <param-value>classpath:SpringMVC-servlet.xml</param-value> 
    </init-param> 
    <!--设置启动级别为1--> 
    <load-on-startup>1</load-on-startup> 
 </servlet> 
<!--/ 匹配所有的请求；（不包括.jsp）--> 
<!--/* 匹配所有的请求；（包括.jsp）--> 
 <servlet-mapping> 
    <servlet-name>SpringMVC</servlet-name> 
    <url-pattern>/</url-pattern>
 </servlet-mapping> 
```

### 二.编写SpringMVC的配置文件
配置文件名字可以随便起，但是官方要求我们最好是用 【servlet-name】-servlet.xml 来给配置文件起名字，
例如我起的名字叫SpringMVC-servlet.xml。

1. 引入命名空间
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd"> 
</beans> 
```
2. 添加处理映射器，处理适配器，视图解析器
试题解析器中要把所有视图都存放在WEB-INF目录中，这个目录客户端不能直接访问，进而保证视图安全
```xml
 <!--    添加处理器映射器-->
<beans>
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <!--    添加处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

    <!--    添加视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" id="internalResourceViewResolver">
        <!--        设置前缀-->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!--        设置后缀-->
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```


### 三.编写操作业务controller
一种是使用注解，另一种是使用实现controller接口的方法，这里我们先使用实现controller接口的方法，
更有助于我们理解SpringMVC的执行过程

```java
public class HelloController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
//      创建一个模型和视图
        ModelAndView modelAndView = new ModelAndView();
//      封装对象，放在ModelAndView中
        modelAndView.addObject("msg","HelloSpringMVC");

//      封装要跳转的视图，放在ModelAndView中
        modelAndView.setViewName("hello");    //: /WEB-INF/jsp/hello.jsp
        return modelAndView;
    }
}
```
将自己写的业务类交给SpringIOC进行管理，在SpringMVC-servlet.xml中注册bean
```xml
<bean id="/hello" class="com.zlq.controller.ControllerTest.HelloController"/>
```

### 四.编写要跳转的jsp页面，显示ModelAndView存放的对象
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Hello~~~</title>
</head>
<body>
    ${msg}
</body>
</html>
```
接下来就可以配置并启动tomcat进行测试了
访问地址为 http://localhost:8080/工程名/hello

开发中一般不这么写，一般都是用注解版



## 注解版

### 一. 由于maven可能出现资源过滤问题，在pom文件中完善配置
```xml
 <build>
        <properties>
              <spring.version>5.0.2.RELEASE</spring.version>
         </properties>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
```
### 二.确定导入SpringMVC的依赖后配置web.xml,注册DispatcherServlet(和配置版相同)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
<!--注册DispatcherServlet-->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<!--        关联SpringMVC的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:SpringMVC-servlet.xml</param-value>
        </init-param>
<!--        设置启动级别为1-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <!--/ 匹配所有的请求；（不包括.jsp）-->
        <!--/* 匹配所有的请求；（包括.jsp） -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    
</web-app>
```
### 三.编写SpringMVC的配置文件
1. 引入命名空间
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation=" http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```
2. 设置自动扫描包，让包下的注解生效，由ioc容器统一管理
```xml
 <context:component-scan base-package="com.zlq.controller.ControllerTest"/>
```
3. 让Spring MVC不处理静态资源
```xml
<mvc:default-servlet-handler />
```

4. 配置支持MVC注解驱动
```xml
<mvc:annotation-driven />
```
5. 配置视图解析器
```xml
 <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver "  id="internalResourceViewResolver">
        <!-- 前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!-- 后缀 -->
        <property name="suffix" value=".jsp"/>
 </bean>
```
### 四.编写操作业务controller
@RequestMapping：该注解可以写在类上也可以写在方法上，如果想写个多级路径就可以在类上整一个，
在方法上整一个。
```java
@Controller   //为了让Spring IOC容器初始化时自动扫描
@RequestMapping("/HelloController")    //为了映射请求路径，这里因为类与方法上都有映射所以访问时应该是/HelloController/hello；
public class HelloController {
    @RequestMapping("/hello")   //此时真实访问地址为 : 项目名/HelloController/hello
    public String sayHello(Model model) { 
        //向模型中添加属性msg与值，可以在JSP页面中取出并渲染,封装对象
        model.addAttribute("msg", "hello,SpringMVC"); //web-inf/jsp/hello.jsp
        return "hello";       //会被视图解析器处理，返回的字符串就是视图的名字供给视图解析器进行拼接
    }
}
```
### 五.编写视图层

```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Hello~~~</title>
</head>
<body>
    ${msg}
</body>
</html>
```
接下来运行tomcat，
访问地址为 http://localhost:8080/工程名/HelloController/hello

<mvc:default-servlet-handler />的讲解说明（摘自csdn博客）

> 通常在进行Spring MVC配置的时候会将DispatcherServlet的url-pattern配置成"/"，Spring MVC将捕获所有的请求，包括静态资源的请求，Spring MVC会将它们当成一个普通请求处理，所以当请求一个图片或者其他静态资源时会出现404的错误。
>
>  在springMVC-servlet.xml中配置<mvc:default-servlet-handler />后，会在Spring MVC上下文中定义一个org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，它会像一个检查员，对进入DispatcherServlet的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。一般Web应用服务器默认的Servlet名称是"default"，因此DefaultServletHttpRequestHandler可以找到它。如果你所有的Web应用服务器的默认Servlet名称不是"default"，则需要通过default-servlet-name属性显示指定：
>
> <mvc:default-servlet-handler default-servlet-name="所使用的Web服务器默认使用的Servlet名称" />

# controller

### 实现controller获得控制器
还是不建议使用实现接口定义controller控制器方式比较，
存在的缺点是一个控制器只能写一个方法，如果需要定义多个方法，则需要定义多个controller类，
定义的方式较麻烦

### 使用注解获得控制器
使用注解的方式可以在一个类里面创建多个请求的方法，多个请求都可以指向同一个视图，每个请求的页面结果都可以不同，但是视图是被复用的

### @RequestMapping()的解释
@RequestMapping()：用来映射访问路径，可以写在类上，也可以写在方法上，可以在类和方法上同时写
代表多级路径，同时也是父子路径关系
Spring MVC 的 @RequestMapping 注解也能够处理 HTTP 请求的方法, 
比如 GET, PUT, POST, DELETE 以 及 PATCH。
```java
@Controller
public class RestfulController {
 @RequestMapping(value = "/hello",method = {RequestMethod.GET})
public String testMethod(Model model){
        model.addAttribute("msg","hello");
        return "test1";
    }
}
```
方法级别的注解变体有如下几个： 组合注解
@GetMapping 、@PostMapping 、@PutMapping 、@DeleteMapping 、@PatchMapping
@GetMapping 是一个组合注解
它所扮演的是 @RequestMapping(method =RequestMethod.GET) 的一个快捷方式。
平时使用的会比较多！

# restful


### restful风格 

restful意为宁静的休闲的
```java
@Controller
public class RestfulController {
    @RequestMapping("commit/{p1}/{p2}")
    //在Spring MVC中可以使用 @PathVariable 注解，让方法参数的值对应绑定到一个URI模板变量上。
    public String testRestful(@PathVariable int p1, @PathVariable int p2, Model model){
        int result = p1 + p2;
        model.addAttribute("msg","result = "+ result);
        return "test1";
    }
}
```
@PathVariable注解的作用：在Spring MVC中使用@PathVariable 注解，让方法参数的值对应绑定到一个URI模板变量上。

注：使用restful风格时如果变量上不写@PathVariable就会出现500错误，写上的目的就是为了把参数值绑定到url



### 转发和重定向
使用ServletAPI实现：

1. 通过HttpServletResponse进行输出
2. 通过HttpServletResponse实现重定向
3. 通过HttpServletResponse实现转发
以上都是通过ServletAPI实现的操作并没有经过SpringMVC
```java

@Controller
public class ResultGo {
    @RequestMapping("/result/t1")
    public void test1(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
        rsp.getWriter().println("Hello,Spring BY servlet API");
    }

    @RequestMapping("/result/t2")
    public void test2(HttpServletRequest req, HttpServletResponse rsp) throws IOException {
        rsp.sendRedirect("/SpringMVC04_controller_war_exploded/index.jsp");
    }

    @RequestMapping("/result/t3")
    public void test3(HttpServletRequest req, HttpServletResponse rsp) throws Exception {
        //转发
        req.setAttribute("msg", "/result/t3");
        req.getRequestDispatcher("/WEB-INF/jsp/test1.jsp").forward(req, rsp);
    }
}
```

2. 通过SpringMVC实现
```java

//使用SpringMVC实现转发
//需要注掉视图解析器
@Controller
public class ResultSpringMVC {
//方式1
    @RequestMapping("/mvc/t1")
    public String test1() {
        return "/index.jsp";
    }
//方式2
    @RequestMapping("/mvc/t2")
    public String test2(){
        return "forward:/index.jsp";
    }
    @RequestMapping("/mvc/t3")
    public String test3(){
        return "redirect:/index.jsp";
    }



}
```

### 通过后台将数据返回到前端的方法

    1. 方式1：通过ModelAndView,上面的都是这种方法，不必多述
    2. 方式2：通过ModelMap
```java
@RequestMapping("/testModelMap")
        public String test1(@RequestParam("username") String name, ModelMap modelMap){
            modelMap.addAttribute("msg",name);
            System.out.println(name);
            return "test1";
        }
```
    3. 方式3：通过Model 
```java
        @RequestMapping("/testModel")
        public String test2(@RequestParam("username") String name, Model model){
            model.addAttribute("msg",name);
            System.out.println(name);
            return "test1";
        }
```

ModelMap和Model的对比

- Model 只有寥寥几个方法只适合用于储存数据，简化了新手对于Model对象的操作和理解； 

- ModelMap 继承了 LinkedMap ，除了实现了自身的一些方法，同样的继承 LinkedMap 的方法和特 性；

- ModelAndView 可以在储存数据的同时，可以进行设置返回的逻辑视图，进行控制展示层的跳转。

### SpringMVC乱码问题的解决

乱码问题是我们开发中十分常见的问题，它是个十足的袁大头（令程序猿头大的问题）
解决方法一 ：使用SpringMVC自带的CharacterEncodingFilter过滤器
```xml
<filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    
```
解决方法二：使用网上某大神写的自定义过滤器,并在xml中配置这个过滤器即可
```java

```
乱码问题，需要平时多注意，在尽可能能设置编码的地方，都设置为统一编码 UTF-8！



### 使用var、let、const关键字的区别
- 使用 var 关键字声明的变量不具备块级作用域的特性，它在 {} 外依然能被访问到。
使用了 var 关键字，它声明的变量是全局的，包括循环体内与循环体外。
- let 声明的变量只在 let 命令所在的代码块 {} 内有效，在 {} 之外不能访问。
使用 let 关键字， 它声明的变量作用域只在循环体内，循环体外的变量不受影响。
- const 用于声明一个或多个常量，声明时必须进行初始化，且初始化后值不可再修改：

# 拦截器
### 介绍 
- SpringMVC的拦截器类似于JavaWeb使用servlet开发中的过滤器filter
- 拦截器是SpringMVC自己所独有的，只有在SpringMVC的工程中才能使用拦截器
- 拦截器只能拦截定义controller控制器中的方法，如果是静态资源（html，css，jpg，MP4，AVI)是不会拦截的

### 回顾过滤器
- servlet规范中的一部分，任何java web工程都可以使用
- 在url-pattern中配置了/*之后，可以对所有要访问的资源进行拦截

### 自定义拦截器
必须实现HandlerInterceptor接口，或者继承HandlerInterceptorAdapter类。

HandlerInterceptor接口中有三个抽象方法，preHandle()，postHandle()，afterCompletion()
preHandle():在请求处理的方法之前执行,如果返回true执行下一个拦截器,如果返回false就不执行下一个拦截器
postHandle():在请求处理方法执行之后执行
afterCompletion():在dispatcherServlet处理后执行,做清理工作.

```java
public class MyInterceptor implements HandlerInterceptor {
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("拦截器处理前");
        return false;
    }

    public void  postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }

}
```

拦截器的配置
```xml
  <mvc:interceptors>
        <mvc:interceptor>
            <!--/** 包括路径及其子路径-->
            <!--/admin/* 拦截的是/admin/add等等这种 , /admin/add/user不会被拦截-->
            <!--/admin/** 拦截的是/admin/下的所有-->
            <mvc:mapping path="/**"/>
            <mvc:exclude-mapping path= > <!—指定不拦截-->
            <bean class="com.zlq.interceptor.MyInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>
```

# JSON

### 什么是Json
JSON(JavaScript Object Notation, JS 对象标记) 是一种目前使用特别广泛的，轻量级的数据交换格式
主要用来作数据交换的
JSON 是 JavaScript 对象的字符串表示法，它使用文本表示一个 JS 对象的信息，本质是一个字符
串。

### JSON 和JavaScript对象的互转
- 将js对象转换成json字符串
```html
<script type="text/javascript">
        
    let json = JSON.stringify({a: 'Hello', b: 'World'}); 
    console.log(json)   //结果是 '{"a": "Hello", "b": "World"}'

    let user = {name: "张立群", age: 16, sex: "男"};
    let str = JSON.stringify(user);
    console.log(str)    //{"name":"张立群","age":16,"sex":"男"}
</script>
```

- 将json字符串转换为js对象
```html
<script type="text/javascript">
    let obj = JSON.parse('{"a": "Hello", "b": "World"}'); //结果是{a: "Hello", b: "World"}
    console.log(obj)

    let parsedUser = JSON.parse(str);
    console.log(parsedUser.name,parsedUser.age,parsedUser.sex);   //张立群 16 男
</script>
```

### controller 返回JSON数据
1. 使用jackson，导入依赖
```xml
 <dependency> 
        <groupId>com.fasterxml.jackson.core</groupId> 
        <artifactId>jackson-databind</artifactId> 
        <version>2.9.8</version> 
 </dependency>
```
2.同时配置web.xml（也就是配置dispatcherServlet，关联config.xml配置文件，设置启动级别为1，再配置CharacterEncodingFilter处理乱码的过滤器）

3.创建配置文件(设置自动扫描包和视图解析器)
```xml
<!-- 自动扫描指定的包，下面所有注解类交给IOC容器管理 --> 
<context:component-scan base-package="com.kuang.controller"/> 
<!-- 视图解析器 --> 
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver " id="internalResourceViewResolver"> 
        <!-- 前缀 --> 
        <property name="prefix" value="/WEB-INF/jsp/" /> 
        <!-- 后缀 --> 
        <property name="suffix" value=".jsp" /> 
</bean>
```


##### 1，测试对象输出
```java
@Controller
public class UserController {
        @RequestMapping("/json1")
        @ResponseBody
        public String json1() throws JsonProcessingException {
            //创建一个jackson的对象映射器，用来解析数据
            ObjectMapper mapper = new ObjectMapper();
            //创建一个对象
            User user = new User( 3,"张立群", 15);
            //将我们的对象解析成为json格式
            String str = mapper.writeValueAsString(user);
            //由于@ResponseBody注解，这里会将str转成json格式返回；十分方便
            return str;
        }
    }
```
@ResponseBody注解的介绍：
由于@ResponseBody注解，这里会将字符串转成Json数据。使用它可以不走视图解析器

4.运行发现出现乱码  
解决方法：
方法1-在@RequestMapping加上produces元素 
@RequestMapping(value = "/json1",produces = "application/json;charset=utf-8")

方法2-通过Spring配置来统一，这样就不必每次都处理了。
在SpringMVC的配置文件上添加一段消息StringHttpMessageConverter转换配置
注：该配置在我这使用会出现500问题，这段配置是在狂神说的笔记中引入的
```xml
<bean class="org.springframework.http.converter.StringHttpMessageConverter">
        <constructor-arg value="UTF-8"/>
    </bean>
    <bean class="org.springframework.http.converter.smile.MappingJackson2SmileHttpMessageConverter">
        <property name="objectMapper">
            <bean class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
                <property name="failOnEmptyBeans" value="false"/>
            </bean>
        </property>
    </bean>
```
@RestController的介绍：前后端分离开发中，我们一般都会使用@RestController来代替@Controller，
因为使用@RestController就不用在每个方法上添加@ResponseBody

以上代码展现的是使用json返回一个对象的字符串，那么返回多个呢？比如集合，接着看：
##### 2，测试集合输出

```java
  @RequestMapping(value = "/json2",produces = "application/json;charset=utf-8")
        public String json2() throws JsonProcessingException {
            ObjectMapper objectMapper = new ObjectMapper();
            User user1 = new User(1, "名字", 12);
            User user2 = new User(2, "名字", 12);
            User user3 = new User(3, "名字", 12);
            User user4 = new User(4, "名字", 12);
            User user5 = new User(5, "名字", 12);
            List<User> userList = new ArrayList<User>();
            userList.add(user1);
            userList.add(user2);
            userList.add(user3);
            userList.add(user4);
            userList.add(user5);
            String string = objectMapper.writeValueAsString(userList);
            return string;
        }
```

##### 3.测试时间输出
```java
  @RequestMapping(value = "/json3", produces = "application/json;charset=utf-8")
    public String json3() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        Date date = new Date();
        String string = objectMapper.writeValueAsString(date);
        return string;
    }
```
运行时可以看到页面上显示一串数字，且随着刷新不断变化，上面的数字代表时间戳，是1970年1月1日到现在的毫秒数。
那如何测试具体的时间呢？
```java
  @RequestMapping(value = "/json4",produces = "application/json;charset=utf-8")
    public String json4() throws JsonProcessingException {
        ObjectMapper objectMapper = new ObjectMapper();
        //使用configure方法设置不使用时间戳
        objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS,false);
        SimpleDateFormat sdf = new SimpleDateFormat("YY-MM-dd HH:mm:ss");
        objectMapper.setDateFormat(sdf);
        Date date = new Date();
        String string = objectMapper.writeValueAsString(date);
        return string;
    }
```

### 使用fastJson
fastJson.jar是阿里开发的一款专门用于Java开发的包，可以方便的实现json对象与JavaBean对象的转
换，实现JavaBean对象与json字符串的转换，实现json对象与json字符串的转换。实现json的转换方法
很多，最后的实现结果都是一样的。





# Ajax
### Ajax是什么？
Ajax是一种无序重新加载网页便能使网页局部更新的技术，是一种为了创建更好、更快、交互性更强的
web应用的技术
### Ajax的起源
2005 年，Google 通过Google Suggest 推动了Ajax的流行。Google Suggest能够自动帮助完成单词的搜索
Google Suggest 使用 AJAX 创造出动态性极强的 web 界面：当您在谷歌的搜索框输入关键字时，
JavaScript 会把这些字符发送到服务器，然后服务器会返回一个搜索建议的列表。
就和国内百度的搜索框一样：

![1602682942706](/Users/liqunzhang/Library/Application Support/typora-user-images/1602682942706.png)

### 使用Ajax可以做什么？
- 注册时，输入用户名自动检测用户是否已经存在。
- 登陆时，提示用户名密码错误
- 删除数据行时，将行ID发送到后台，后台在数据库中删除，数据库删除成功后，在页面DOM中将
数据行也删除。
- ....等等

### JQuery.Ajax()的参数
主要掌握 url type data success error
部分参数：
- url：请求地址 
- type：请求方式，GET、POST（1.9.0之后用method）
- headers：请求头 
- data：要发送的数据 
- contentType：即将发送信息至服务器的内容编码类型(默认: "application/x-www- form-urlencoded; charset=UTF-8") 
- async：是否异步 
- timeout：设置请求超时时间（毫秒）
- beforeSend：发送请求前执行的函数(全局) 
- complete：完成之后执行的回调函数(全局) 
- success：成功之后执行的回调函数(全局) 
- error：失败之后执行的回调函数(全局) 
- accepts：通过请求头发送给服务器，告诉服务器当前客户端课接受的数据类型 
- dataType：将服务器端返回的数据转换成指定类型 
- "xml": 将服务器端返回的内容转换成xml格式 
- "text": 将服务器端返回的内容转换成普通文本格式 
- "html": 将服务器端返回的内容转换成普通文本格式，在插入DOM中时，如果包含 JavaScript标签，则会尝试去执行。 
- "script": 尝试将返回值当作JavaScript去执行，然后再将服务器端返回的内容转换成 普通文本格式 
- "json": 将服务器端返回的内容转换成相应的JavaScript对象 
- "jsonp": JSONP 格式使用 JSONP 形式调用函数时，如 "myurl?callback=?" jQuery 将自动替换 ? 为正确的函数名，以执行回调函数

### SpringMVC实现Ajax
1. 编写实体类User
```java
@Data 
@AllArgsConstructor 
@NoArgsConstructor 
public class User {
    private String name;
    private int age;
    private String sex;
 }
```
2. 获取集合对象，展示给前端
```java
 @RequestMapping("/a2")
    public List<User> ajax2() {
        List<User> list = new ArrayList<User>();
        list.add(new User("***", 18, "男"));
        list.add(new User("***", 18, "女"));
        list.add(new User("***", 18, "女"));
        return list; //由于@RestController注解，将list转成json格式返回 }
    }
```

3. 前端页面
```xml
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<script src="${pageContext.request.contextPath}/statics/js/jquery-3.4.1.js"></script>
<html>
<head>
    <title>Title</title>
</head>
<body>

<input type="button" id="btn" value="获取数据"/>
<table width="80%" align="center">
    <tr>
        <td>姓名</td>
        <td>年龄</td>
        <td>性别</td>
    </tr>
    <tbody id="content">
    </tbody>
</table>


<script>
    $(function () {
        $("#btn").click(function () {
            $.post("${pageContext.request.contextPath}/ajax/a2",function (data) {
                console.log(data);
                var html = "";
                for (var i=0;i<data.length;i++){
                    html += "<tr>" +
                        "<td>"+data[i].name + "</td>" +
                        "<td>"+data[i].age + "</td>" +
                        "<td>"+data[i].sex + "</td>" +
                        "</tr>"
                }
                $("#content").html(html);
            })
        })
    })
</script>

</body>
</html>

```
### 使用Ajax实现注册提示效果
1. 编写controller
```java
@RequestMapping("/a3")
    public String ajax3(String name,String pwd){
        String msg = "";
        if (name != null) {
            if ("zlq".equals(name)){
                msg = "ok";
            }else {
                msg = "wrong username!";
            }
        }
        if (pwd != null) {
            if ("123456".equals(pwd)){
                msg = "ok";
            }else {
                msg = "wrong password!";
            }
        }
        return msg;  //有@RestController就不走视图解析器，直接将msg转换成json格式
    }
```
2. 前端页面
```xml
<%--
  Created by IntelliJ IDEA.
  User: Administrator
  Date: 2020/10/13
  Time: 21:39
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<script src="${pageContext.request.contextPath}/statics/js/jquery-3.4.1.js"></script>
<html>
<head>
    <title>注册页面</title>

    <script type="text/javascript">
        function a1() {
            $.post(
                {
                    url: "${pageContext.request.contextPath}/ajax/a3",
                    data: {'name': $("#name").val()},
                    success: function (data) {
                        if (data.toString() == 'OK') {
                            $("#userInfo").css("color", "green");
                        } else {
                            $("#userInfo").css("color", "red");
                        }
                        $("#userInfo").html(data);
                    }
                }
            )
        }

        function a2() {
            $.post(
                {
                    url: "${pageContext.request.contextPath}/ajax/a3",
                    data: {'pwd': $("#pwd").val()},
                    success: function (data) {
                        if (data.toString() == 'OK') {
                            $("#pwdInfo").css("color", "green");
                        } else {
                            $("#pwdInfo").css("color", "red");
                        }
                        $("#pwdInfo").html(data);
                    }
                }
            )
        }
    </script>
</head>
<body>
<p> 用户名:<input type="text" id="name" onblur="a1()"/> <span id="userInfo"></span></p>
<p> 密码:<input type="text" id="pwd" onblur="a2()"/> <span id="pwdInfo"></span></p>
</body>
</html>

```
主要掌握javaScript

效果如图

![1602682994464](/Users/liqunzhang/Library/Application Support/typora-user-images/1602682994464.png)




# 文件的上传和下载
- application/x-www=form-urlencoded：默认方式，只处理表单域中的 value 属性值，采用这种编
码方式的表单会将表单域中的值处理成 URL 编码方式。
- multipart/form-data：这种编码方式会以二进制流的方式来处理表单数据，这种编码方式会把文
件域指定文件的内容也封装到请求参数中，不会对字符编码。
- text/plain：除了把空格转换为 "+" 号外，其他字符都不做编码处理，这种方式适用直接通过表单
发送邮件。

### 导入依赖
```xml
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>1.3.3</version>
        </dependency> <!--servlet-api导入高版本的-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
        </dependency>
```

### 配置：multipartResolver
```xml
<!--文件上传配置-->
    <bean id="multipartResolver"
          class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 请求的编码格式，必须和jSP的pageEncoding属性一致，以便正确读取表单的内容， 默认为ISO-8859-1 -->
        <property name="defaultEncoding" value="utf-8"/>
        <!-- 上传文件大小上限，单位为字节（10485760=10M） -->
        <property name="maxUploadSize" value="10485760"/>
        <property name="maxInMemorySize" value="40960"/>
    </bean>
```

CommonsMultipartFile 的 常用方法：
String getOriginalFilename()：获取上传文件的原名
InputStream getInputStream()：获取文件流
void transferTo(File dest)：将上传文件保存到一个目录文件中



### 文件上传

编写controller
```java
package com.zlq.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

import javax.servlet.http.HttpServletRequest;
import java.io.*;

/**
 * @ProjectName:SpringMVC
 * @Package:com.zlq.controller
 * @ClassName: DownLoadController
 * @description:
 * @author: Larry
 * @CreateDate:2020/10/14 21:01
 */
@Controller
public class DownLoadController {
    @RequestMapping("/upload")
    public String fileUpload(@RequestParam("file") CommonsMultipartFile file, HttpServletRequest request) throws IOException {
        //获取文件名 :
        file.getOriginalFilename();
        String uploadFileName = file.getOriginalFilename();
        //如果文件名为空，直接回到首页！
        if ("".equals(uploadFileName)) {
            return "redirect:/index.jsp";
        }
        System.out.println("上传文件名 : " + uploadFileName);
        //上传路径保存设置
        String path = request.getServletContext().getRealPath("/upload");
        //如果路径不存在，创建一个
        File realPath = new File(path);
        if (!realPath.exists()) {
            realPath.mkdir();
        }
        System.out.println("上传文件保存地址：" + realPath);
        InputStream is = file.getInputStream();//文件输入流
        OutputStream os = new FileOutputStream(new File(realPath, uploadFileName));//文件输出流
        // 读取写出
        int len = 0;
        byte[] buffer = new byte[1024];
        while ((len = is.read(buffer)) != -1) {
            os.write(buffer, 0, len);
            os.flush();
        }
        os.close();
        is.close();
        return "redirect:/index.jsp";
    }
}

```

前端页面

```xml
<form action="${pageContext.request.contextPath}/upload" enctype="multipart/form-data" method="post">
    <input type="file" neme="file">
    <input type="submit" value="上传">
</form>
```



### 文件下载

文件下载步骤：
1. 设置 response 响应头
2. 读取文件 -- InputStream
3. 写出文件 -- OutputStream
4. 执行操作

```java
 @RequestMapping(value = "/download")
    public String downloads(HttpServletResponse response, HttpServletRequest request) throws Exception {
        //要下载的图片地址
        String path = request.getServletContext().getRealPath("/upload");
        String fileName = "基础语法.jpg";
        //1、设置response 响应头
        response.reset();//设置页面不缓存,清空buffer
        response.setCharacterEncoding("UTF-8");//字符编码
        response.setContentType("multipart/form-data");//二进制传输数据
        // 设置响应头
        response.setHeader("Content-Disposition", "attachment;fileName=" + URLEncoder.encode(fileName, "UTF-8"));
        File file = new File(path, fileName);
        //2、 读取文件--输入流
        InputStream input = new FileInputStream(file);
        //3、 写出文件--输出流
        OutputStream out = response.getOutputStream();
        byte[] buff = new byte[1024];
        int index = 0;
        //4、执行 写出操作
        while ((index = input.read(buff)) != -1) {
            out.write(buff, 0, index);
            out.flush();
        }
        out.close();
        input.close();
        return null;
    }
```

前端页面
```xml
<a href="/download">点击下载</a>
```

# JSR303数据校验

使用JSR303数据校验用来保证数据的准确性，常用的校验规则有：

- @Null 验证对象是否为null 

- @NotNull 验证对象是否不为null, 无法查检长度为0的字符串 

- @NotBlank 检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格. 

- @NotEmpty 检查约束元素是否为NULL或者是EMPTY. Booelan检查 

- @AssertTrue 验证 Boolean 对象是否为 true 

- @AssertFalse 验证 Boolean 对象是否为 false 

- @Size(min=, max=) 长度检查，验证对象（Array,Collection,Map,String）长度是否在给定的范围之内 

- @Length(min=, max=) string is between min and max included. 

- @Past 日期检查 ，验证 Date 和 Calendar 对象是否在当前时间之前 

- @Future 验证 Date 和 Calendar 对象是否在当前时间之后 

- @Pattern 验证 String 对象是否符合正则表达式的规则 

.......等等 

除此以外，我们还可以自定义一些数据校验规则

例如

```java
public class Employee {
    private Integer empId;

    @Pattern(regexp="(^[a-zA-Z0-9_-]{6,16}$)|(^[\u2E80-\u9FFF]{2,5})"
            ,message="用户名必须是2-5位中文或者6-16位英文和数字的组合")
    private String empName;

    private String gender;

    @Pattern(regexp="^([a-z0-9_\\.-]+)@([\\da-z\\.-]+)\\.([a-z\\.]{2,6})$",
            message="邮箱格式不正确")
    private String email;

    private Integer dId;

    private Department department;
}
```



```java
@Component //注册bean
@Validated //数据校验 
public class Person {
    @Email(message="邮箱格式错误") //name必须是邮箱格式 
    private String email; 
    
    @NotNull(message="名字不能为空") 
    private String userName; 
    
    @Max(value=120,message="年龄最大不能查过120") 
    private int age; 
    
    @Email(message="邮箱格式错误") 
    private String email;
}
```

