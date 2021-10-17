



# 简介

**回顾SpringBoot的使用步骤**

1. 创建SpringBoot应用，选中我们需要的模块； 
2.  SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来 
3. 自己编写业务代码；



xxxxAutoConfiguration：帮我们给容器中自动配置组件； 

xxxxProperties:配置类来封装配置文件的内容；









# 1、SpringMVC自动配置概览

使用SpringBoot 在大多场景我们都无需自定义配置SpringMvc

在SpringMVC自动配置上添加了如下功能

- - 内容协商视图解析器和BeanName视图解析器

- - 支持静态资源（包括webjars）

- - 自动注册 `Converter，GenericConverter，Formatter `

- - 支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）

- - 自动注册 `MessageCodesResolver` （国际化用）

- - 静态index.html 页支持

- - 自定义 `Favicon`  

- - 自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）

# 2、简单功能分析



## 2.1、静态资源访问

我们开发项目是要引入很多前端页面，项目中还有很多静态资源如css，jQuery，js文件等，这些SpringBoot都会帮我们处理，以前的非SpringBoot项目，我们的静态资源都放在web/statics下面，但是现在使用SpringBoot，我们打包时会以为jar包的方式打出，就不能用以前的方式放置静态资源，要按照SpringBoot的规定来放置静态资源！





## 2.2、什么是webjars？



webjars是以导jar包的方式引入静态资源，例如我们想引入静态资源jQuery，以前要引入Jquery.js，现在好了，我们只需要在SpringBoot项目中引入Jquery的jar包就可以导入Jquery！

```xml
   			<dependency>
            <groupId>org.webjars</groupId>
            <artifactId>jquery</artifactId>
            <version>3.5.1</version>
        </dependency>
```

访问：                                         http://localhost:8080/webjars/jquery/3.5.1/jquery.js 

就会去找到这个jar包的类路径下/META-INF/resources/webjars/jquery/3.5.1/jquery.js 资源！

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210122200957326.png" alt="image-20210122200957326" style="zoom:30%;" />



<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210122201109701.png" alt="image-20210122201109701" style="zoom:50%;" />



```java
if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl)
						.setUseLastModified(this.resourceProperties.getCache().isUseLastModified()));
			}
```

访问`/webjars/**`下面的所有资源都去`classpath:/META-INF/resources/webjars/`找资源





## 2.3、静态资源放置路径

如果是自己的静态资源该放哪儿

我们只要将静态资源放入下面路径都会被SpringBoot识别

- "classpath:/META‐INF/resources/", 
- "classpath:/resources/", 
- "classpath:/static/", 
- "classpath:/public/" 

优先级由高到低

**classpath：指的是resources资源目录**



例如我们要访问localhost:8080/abc，那就去去静态资源文件夹里面找abc这个文件



**静态资源访问前缀**

默认是无前缀的，我们可以通过配置文件加前缀

```yaml
spring:
  mvc:
    static-path-pattern: /res/**
```

访问路径：当前项目根路径/res/静态资源名 



## 2.4、欢迎页的定制

**欢迎页**：静态资源文件夹下的所有index.html页面即可被识别成主页

- 可以自定义配置静态资源路径
- 但是不可以自定义配置静态资源的访问前缀。否则导致 index.html不能被默认访问

我们访问localhost:8080/，就会去静态资源放置路径下找index.html页面 

注：我们使用controller也可以定制欢迎页的访问路径



源码分析

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201211114502686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjQwNzAy,size_16,color_FFFFFF,t_70#pic_center)

HandlerMapping是SpringMvc的底层注解，它来保存每一个请求对应谁来处理

```java
private Optional<Resource> getWelcomePage() {
			String[] locations = getResourceLocations(this.resourceProperties.getStaticLocations());
			return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
		}

private Resource getIndexHtml(String location) {
			return this.resourceLoader.getResource(location + "index.html");
		}
```

`getWelcomePage（）`的返回对应由 `this.mvcProperties.getStaticPathPattern()`来处理

即所有静态资源文件夹下的index.html，都被/**映射，即localhost:8080 -> 找index页面



**static文件夹和templates文件夹**

SpringBoot里面没有我们之前常规web开发的 WebContent（WebApp），它只有src目录（正常的maven结构）

在src/main/resources下面有两个文件夹，static和templates。springboot默认static中放静态页面，而templates中放动态页面

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210123144335643.png" alt="image-20210123144335643" style="zoom:50%;" />

## 2.5、自定义 Favicon定制页面图标

**定制页面图标** ： 在静态资源目录下任意层级放置favicon.ico，如果存在这样的文件， 它将自动用作应用程序的favicon。

**favicon.ico 放在静态资源目录下即可。**

注：自定义静态资源访问前缀会导致Favicon失效

```yaml
spring:
  mvc:
    static-path-pattern: /res/**
```



## 2.6、自定义静态资源路径

我们也可以自己通过配置文件来指定一下，哪些文件夹是需要我们放静态资源文件的，在 

application.properties中配置； 但是一旦自己定义了静态文件夹的路径，原来的自动配置就都会失效！ 

```yaml
spring:
	resources:
		static-locations: classpath:/coding/,classpath:/zlq/
```



## 2.7、静态资源配置原理



```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
         customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```



我们知道SpringBoot启动会默认加载各种xxxAutoConfiguration类（自动配置类），其中SpringMVC功能的自动配置类 WebMvcAutoConfiguration是生效的

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
        ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
  
}
```

- 给容器中配了什么。

```java
	  @Configuration(proxyBeanMethods = false)
    @Import({WebMvcAutoConfiguration.EnableWebMvcConfiguration.class})
    @EnableConfigurationProperties({WebMvcProperties.class, ResourceProperties.class, WebProperties.class})
    @Order(0)
    public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
    }
```

- 配置文件的相关属性和xxx进行了绑定。WebMvcProperties绑定配置中的**spring.mvc**、ResourceProperties绑定配置中的**spring.resources**





#### 1、配置类只有一个有参构造器

```java
//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory Spring的beanFactory
//HttpMessageConverters 找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。=========
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
    public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, WebMvcProperties mvcProperties,
                ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
                ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
                ObjectProvider<DispatcherServletPath> dispatcherServletPath,
                ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
            this.resourceProperties = resourceProperties;
            this.mvcProperties = mvcProperties;
            this.beanFactory = beanFactory;
            this.messageConvertersProvider = messageConvertersProvider;
            this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
            this.dispatcherServletPath = dispatcherServletPath;
            this.servletRegistrations = servletRegistrations;
        }
```





#### 2、资源处理的默认规则

```java
@Override
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
                return;
            }
            Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
            CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
            //webjars的规则,所有的 /webjars/** ， 都需要去 classpath:/META-INF/resources/webjars/ 找对应的资源
				
            if (!registry.hasMappingForPattern("/webjars/**")) {
                customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                        .addResourceLocations("classpath:/META-INF/resources/webjars/")
                        .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
            }
            
            //
            String staticPathPattern = this.mvcProperties.getStaticPathPattern();
            if (!registry.hasMappingForPattern(staticPathPattern)) {
                customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                        .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                        .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
            }
        }

```



```yaml
spring:
  web:
    resources:
      add-mappings: false   # 禁用所有静态资源规则
```


​                                 

```java
@Deprecated
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties extends Resources {
  
}

 public static class Resources {
       private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{
         "classpath:/META-INF/resources/",
         "classpath:/resources/",
         "classpath:/static/",
         "classpath:/public/"
       };
 }
```



#### 3、欢迎页的处理规则

```java
    HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。  

    @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
                FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
            WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
                    new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
                    this.mvcProperties.getStaticPathPattern());
            welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
            welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
            return welcomePageHandlerMapping;
        }

    WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
            ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
        if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
            //要用欢迎页功能，必须是/**
            logger.info("Adding welcome page: " + welcomePage.get());
            setRootViewName("forward:index.html");
        }
        else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
            // 调用Controller  /index
            logger.info("Adding welcome page template: index");
            setRootViewName("index");
        }
    }
```

# 3、请求参数处理



## 3.1、rest使用与原理

@xxxMapping：包含@GetMapping、@PostMapping、@PutMapping、@DeleteMapping

- Rest风格支持（*使用**HTTP**请求方式动词来表示对资源的操作*）

- 以前我们用/getUser  获取用户、  /deleteUser 删除用户 、  /editUser  修改用户   、 /saveUser 保存用户。现在我们只需要用 /user一种请求路径即可将增删改查请求分离出来

-    *GET-获取用户*  、 *DELETE-删除用户*   、*PUT-修改用户*    、*POST-保存用户*

```java
@RequestMapping(value = "/user",method = RequestMethod.GET)
@RequestMapping(value = "/user",method = RequestMethod.POST)
@RequestMapping(value = "/user",method = RequestMethod.PUT)
@RequestMapping(value = "/user",method = RequestMethod.DELETE)

//等价于：
@GetMapping("/user")
@PostMapping("/user")
@PutMapping("/user")
@DeleteMapping("/user")
  
```

用法：默认是不识别put和delete请求的，需要我们在隐藏域添加：

 <input type="hidden" name="_method" value="put">

<input type="hidden" name="_method" value="delete">

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210123103506682.png" alt="image-20210123103506682" style="zoom:36%;" />

并在SpringBoot中手动开启隐藏域过滤器,开启页面表单的Rest功能

```yaml

spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true
```



- 核心Filter；HiddenHttpMethodFilter

- 扩展：如何把_method 这个名字换成我们自己喜欢的。

```java
		@Bean
    @ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
    @ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
    public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
        return new OrderedHiddenHttpMethodFilter();
    }


//自定义filter
    @Bean
    public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
        HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
        methodFilter.setMethodParam("_m");
        return methodFilter;
    }
```





## 3.2、请求映射原理

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181171918-b8acfb93-4914-4208-9943-b37610e93864.png)

SpringMVC功能分析都从 org.springframework.web.servlet.DispatcherServlet  --> doDispatch（）



```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = (processedRequest != request);

                // 找到当前请求使用哪个Handler（Controller的方法）处理
                mappedHandler = getHandler(processedRequest);
                
                //HandlerMapping：处理器映射。/xxx->>xxxx
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181460034-ba25f3c0-9cfd-4432-8949-3d1dd88d8b12.png)

**RequestMappingHandlerMapping**：保存了所有@RequestMapping 和handler的映射规则。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1603181662070-9e526de8-fd78-4a02-9410-728f059d6aef.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_14%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

所有的请求映射都在HandlerMapping中。



- SpringBoot自动配置欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping
- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。

- - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping

- 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**

```java
    protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            for (HandlerMapping mapping : this.handlerMappings) {
                HandlerExecutionChain handler = mapping.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }
        return null;
    }
```





## 3.3、矩阵变量

定义：路径片段中可以包含键值对，多个键值对可以由分号隔开

SpringBoot默认禁用矩阵变量功能，因为SpringBoot默认是将路径变量中的分号移除的，我们需要关闭这个功能

在配置类中写入：

```java
@Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                //不移除分号"；"后面的内容，矩阵变量才能生效
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }
        };
    }
```



**案例1：**

```java

    @GetMapping("/cars/{path}")
    public Map sellCars(@MatrixVariable("sellCounts") Integer sellCounts,
                        @MatrixVariable("carTypes") List<String> carTypes,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("sellCounts",sellCounts);
        map.put("carTypes",carTypes);
        map.put("path",path);
        return map;
    }
```

访问测试下：

<http://localhost:8080/cars/sell;sellCounts=80;carTypes=奔驰,宝马,奥迪>

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210123160330417.png" alt="image-20210123160330417" style="zoom:50%;" />



**案例二：**

```java
GetMapping(value = "/owners/{ownerId}/pets/{petId}")
    public void ownerAndPet(@PathVariable String ownerId,
                   @PathVariable String petId,
                    //该注解中的value指定路径片段中矩阵变量的变量名；pathvar指定该变量对应哪个路径片段
                   @MatrixVariable(value="ownerAge", pathVar="ownerId") Integer ownerAge,
                   @MatrixVariable(value="petAge", pathVar="petId") Integer perAge) {
        System.out.println("ownerId:" + ownerId + ":" + "petId:" + petId + "\n" +
                "ownerAge:" + ownerAge + ":" + "perAge:" + perAge);


    }
```

访问     http://localhost:8080/owners/1;ownerAge=12/pets/3;petAge=2

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210123162736323.png" alt="image-20210123162736323" style="zoom:50%;" />

访问 http://localhost:8080/owners/1;ownerAge=12,13,15/pets/3;petAge=2,12,14

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210123162813177.png" alt="image-20210123162813177" style="zoom:50%;" />



**结论：**

- **每个路径片段可以有多个变量，以key=value的形式存在，多个k-v之间用分号分隔**

- **路径片段中的每个变量可以有多个值，多个值用逗号分隔**

# 4、Thymeleaf



## 模板引擎

前端交给我们的页面是html页面。如果是我们以前开发，我们需要把他们转成jsp页面，我们现在的这种情况，SpringBoot这个项目首先是以jar的方式打包，不是war，我们用的还是嵌入式的Tomcat，所以他现在默认是不支持jsp。

**SpringBoot**推荐使用模板引擎：

SpringBoot给我们推荐的Thymeleaf模板引擎，它是一个高级语言的模板引擎，语法更简单，功能更强大。

 

引入

```xml
			  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```



```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
	public static final String DEFAULT_PREFIX = "classpath:/templates/";
	public static final String DEFAULT_SUFFIX = ".html";
	private boolean checkTemplate = true;
	private boolean checkTemplateLocation = true;
	private String prefix = DEFAULT_PREFIX;
	private String suffix = DEFAULT_SUFFIX;
	private String mode = "HTML";
	private Charset encoding = DEFAULT_ENCODING;

```

我们只需要把我们的html页面放在类路径下的templates下，thymeleaf就可以帮我们自动渲染



<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/image-20210124195023306.png" alt="image-20210124195023306" style="zoom:50%;" />

# 5、拦截器



编写自定义拦截器实现HandlerInterceptor接口

重写preHandle，postHandle，afterCompletion

```java
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        System.out.println("preHandle拦截的路径为：" + requestURI);
        if (request.getSession().getAttribute("loginUser") == null) {
            request.setAttribute("msg","请先登录");
            request.getRequestDispatcher("/").forward(request,response);
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
       log.info("postHandle执行-----------",modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常---------",ex);
    }
}
```

将拦截器注册到容器中，并制定拦截路径

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")//拦截包括静态资源的所有请求
                .excludePathPatterns("/", "/login","/css/**","/fonts/**","/images/**","/js/**");
        //主页面和登录页面和静态资源不能拦截
    }
}
```

**拦截器原理**

1. 根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】
2. 先来**顺序执行** 所有拦截器的 preHandle方法

- 如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
- 如果当前拦截器返回为false。直接   倒序执行所有已经执行了的拦截器的  afterCompletion；

3. 如果任何一个拦截器返回false。直接跳出不执行目标方法*
4. 所有拦截器都返回True。执行目标方法
5. 倒序执行所有拦截器的postHandle方法。
6. 前面的步骤有任何异常都会直接倒序触发 afterCompletion
7. 页面成功渲染完成以后，也会倒序触发 afterCompletion



![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1605764129365-5b31a748-1541-4bee-9692-1917b3364bc6.png?x-oss-process=image%2Fresize%2Cw_1500)



![image.png](https://cdn.nlark.com/yuque/0/2020/png/1354552/1605765121071-64cfc649-4892-49a3-ac08-88b52fb4286f.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_YXRndWlndS5jb20g5bCa56GF6LC3%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

# SpringMVC自动配置 



@ConditionalOnMissingBean(value= exampleClass):标注该注解意味着容器中没有这个exampleClass配置的时候才会把这段配置信息注入组件，如果容器中有这个exampleClass配置（比如我们手动重新配置了这个exampleClass组件）那么原先自带的exampleClass组件就会失效，就不会注入到容器中了



# 错误处理机制以及404页面的定制



## SpringBoot默认处理错误机制



### 浏览器和其他客户端的错误页面



在实际开发中，我们制作的页面不一定会显示给电脑上的客户端，也有可能显示给手机或者平板上的客户端，如何能良好的适配错误页面呢？





- 浏览器显示的错误页面

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/1604995391409.png" alt="1604995391409" style="zoom:50%;" />

正常情况下，浏览器给我们返回一个SpringBoot默认的一个错误页面，里面有时间戳，错误类型type，状态status



- 其他客户端显示的错误页面

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/1604996042929.png" alt="1604996042929" style="zoom:50%;" />

默认以json数据的形式显示



- 浏览器发送的请求头

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/1604995651912.png" alt="1604995651912" style="zoom:50%;" />



- 其他客户端发送的请求头

<img src="/Users/liqunzhang/Library/Application Support/typora-user-images/1604995781037.png" alt="1604995781037" style="zoom:50%;" />



可以看出浏览器发送的请求头和奇特客户端发送的请求头得accept参数是有区别的，而SpringBoot正是在这种区别之上进行了适配，才导致显示的页面的不同



### 错误页面显示原理： 

可以参照ErrorMvcAutoConfifiguration；错误处理的自动配置； 

里面注入了很多组件

1. DefaultErrorAttributes

   ```java
   @Bean
   	@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
   	public DefaultErrorAttributes errorAttributes() {
   		return new DefaultErrorAttributes();
   	}
   
   ```

点进DefaultErrorAttributes：

该方法会帮我们在错误页面添加time时间戳，status

```java
 @Deprecated
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> errorAttributes = new LinkedHashMap();
        errorAttributes.put("timestamp", new Date());
        this.addStatus(errorAttributes, webRequest);
        this.addErrorDetails(errorAttributes, webRequest, includeStackTrace);
        this.addPath(errorAttributes, webRequest);
        return errorAttributes;
    }
```



2. BasicErrorController

   ```java
   	@Bean
   	@ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)
   	public BasicErrorController basicErrorController(ErrorAttributes errorAttributes,
   			ObjectProvider<ErrorViewResolver> errorViewResolvers) {
   		return new BasicErrorController(errorAttributes, this.serverProperties.getError(),
   				errorViewResolvers.orderedStream().collect(Collectors.toList()));
   	}
   ```

   点进BasicErrorController

   ```java
   @Controller
   @RequestMapping("${server.error.path:${error.path:/error}}")  //默认走/error请求
   public class BasicErrorController extends AbstractErrorController {
   
   
   	@RequestMapping(produces = "text/html") //产生html类型的数据；浏览器发送的请求来到这个方法处理
   	public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
   		HttpStatus status = getStatus(request);
   		Map<String, Object> model = Collections
   				.unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
   		response.setStatus(status.value());  //设置status的值
           
           //决定了去哪个错误页面
           			//解析错误页面的数据
   		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
           		    //如果有我们自己的错误页面就去我们自己的，否则就去默认的error页面
   		return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
   	}
   
   	@RequestMapping
   	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
   		HttpStatus status = getStatus(request);
   		if (status == HttpStatus.NO_CONTENT) {
   			return new ResponseEntity<>(status);
   		}
   		Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL)); 
   		return new ResponseEntity<>(body, status);  //产生json数据，其他客户端来到这个方法处理；
   	}
   
   
   }
   
   ```

3. ErrorPageCustomizer 

   ```java
   @Value("${error.path:/error}") 
   private String path = "/error";  //系统出现错误以后来到error请求进行处理；（web.xml注册的错误页 面规则）
   ```



```java
@Override 
public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) { 
    ModelAndView modelAndView = resolve(String.valueOf(status), model); 
    if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) { 
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model); 
    }
    return modelAndView;
}
private ModelAndView resolve(String viewName, Map<String, Object> model) { 
    //默认SpringBoot可以去找到一个页面？ error/404 
    String errorViewName = "error/" + viewName; //模板引擎可以解析这个页面地址就用模板引擎解析 
    TemplateAvailabilityProvider provider = this.templateAvailabilityProviders .getProvider(errorViewName, this.applicationContext); 
    if (provider != null) { //模板引擎可用的情况下返回到errorViewName指定的视图地址 
        return new ModelAndView(errorViewName, model); 
    }//模板引擎不可用，就在静态资源文件夹下找errorViewName对应的页面 error/404.html 
    return resolveResource(errorViewNme, model); 
}
```



## 定制错误相应



### 一.如何定制错误页面

1. 有模板引擎的情况下；将错误页面命名为 错误状态码.html 放在模板引擎文件夹里面的 error文件夹，发生此状态码的错误就会来到 对应的页面； 我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态 码.html）； 

页面能获取的信息； 

- timestamp：时间戳 

- status：状态码 

- error：错误提示 

- exception：异常对象 

- message：异常消息 

- errors：JSR303数据校验的错误都在这里 

2. 没有模板引擎（模板引擎找不到这个错误页面），静态资源文件夹下找； 

3. 以上都没有错误页面，就是默认来到SpringBoot默认的错误提示页面； 



### 二.定制错误页面的Json数据、

1. 定制异常

```java
public class UserNotExistException extends RuntimeException {

    public UserNotExistException() {
        super("用户不存在");
    }
}
```

2. 编写异常处理类MyExceptionHandler

- 1）自定义异常处理&返回定制json数据；

```java
@ControllerAdvice 
public class MyExceptionHandler {
    @ResponseBody
    @ExceptionHandler(UserNotExistException.class) 
    public Map<String,Object> handleException(Exception e){ 
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexist"); 
        map.put("message",e.getMessage()); 
        return map;
    }
}
```

该方法没有自适应效果，虽然其他客户端上多显示了json数据，但是电脑上的客户端也显示了json数据了，不显示定制的错误页面了！



- 2）转发到/error进行自适应响应效果处理

```java

    @ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        //传入我们自己的错误状态码 4xx 5xx，否则就不会进入定制错误页面的解析流程
        /**
          Integer statusCode = (Integer)
         request .getAttribute("javax.servlet.error.status_code"); */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message","用户出错啦");
        request.setAttribute("ext",map);
        //转发到/error
        return "forward:/error";
    }
```



- 3）将我们的定制数据携带出去；

  出现错误以后，会来到/error请求，会被BasicErrorController处理，响应出去可以获取的数据是由 getErrorAttributes得到的（是AbstractErrorController（ErrorController）规定的方法）； 

  

  1、我们可以完全来编写一个ErrorController的实现类（或者是编写AbstractErrorController的子类）放在容器中； 

  2、页面上能用的数据，或者是json返回能用的数据都是通过errorAttributes.getErrorAttributes得到； 

  容器中getErrorAttributes()；默认进行数据处理的； 

```java
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, options);
        errorAttributes.put("name", "Larry");
        return errorAttributes;
    }
}
```



# 配置嵌入式Servlet容器

![1605003713297](/Users/liqunzhang/Library/Application Support/typora-user-images/1605003713297.png)

由依赖树可以看出SpringBoot默认以tomcat作为嵌入式Servlet容器



## 如何定制和修改Servlet容器的相关配置

在application.properties写入server有关的配置

```properties
server.port=8088
server.servlet.context-path=/crud
server.tomcat.uri-encoding=utf-8
```

注：通用的Servlet容器设置 ：server.xxx 

Tomcat的设置  ：server.tomcat.xxx 



##  如何注册Servlet三大组件【Servlet、Filter、Listener】

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件，因此我们需要以Java代码的方式注入组件



以前在JavaWeb中的方式

```xml
 <servlet>
        <servlet-name>ExampleClass</servlet-name>
        <servlet-class>com.zlq.ExampleClass</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>ExampleClass</servlet-name>
        <url-pattern>/exampleClass</url-pattern>
    </servlet-mapping>
```

现在的方式

1. 注册Servlet

- 编写MyServlet

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req,resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws Servlet Exception, IOException {
        resp.getWriter().write("Hello MyServlet");
        super.doPost(req, resp);
    }
}
```

- 注入MyServlet

```java
  @Bean
    public ServletRegistrationBean myServlet() {
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(), "/myServlet");
        return registrationBean;
    }
```

2.注册Fliter过滤器

- 编写MyFilter

```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("MyFilter process...");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```



- 注入MyFilter

```java
  @Bean
    public FilterRegistrationBean myFilter() {
        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(new MyFilter());
        registrationBean.setUrlPatterns(Arrays.asList("/hello", "/myFilter"));
        return registrationBean;
    }
```



3. 注册Listenner监听器

- 编写MyListener

```java
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println("contextInitialized...web应用启动");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println("contextDestroyed...当前web项目销毁");
    }
}

```

- 注入MyListener

```java
 @Bean
    public ServletListenerRegistrationBean myListener(){
        ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
        return registrationBean;
    }
```

![1605008041671](/Users/liqunzhang/Library/Application Support/typora-user-images/1605008041671.png)



SpringBoot帮我们自动注册了SpringMVC的dispatherServlet前置控制器，在DispatcherServletAutoConfifiguration中

```java
	@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
		@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
		public DispatcherServletRegistrationBean dispatcherServletRegistration(DispatcherServlet dispatcherServlet,	WebMvcProperties webMvcProperties, ObjectProvider<MultipartConfigElement> multipartConfig) {
            //默认拦截的是"/": 拦截所有请求，包括静态资源，但是不拦截jsp请求。 /*会拦截jsp请求
            //我们可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径
			DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(dispatcherServlet,
					webMvcProperties.getServlet().getPath());
			registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
			registration.setLoadOnStartup(webMvcProperties.getServlet().getLoadOnStartup());
			multipartConfig.ifAvailable(registration::setMultipartConfig);
			return registration;
		}

	}
```



思考：过滤器和拦截器的区别？

## 替换为其他嵌入式Servlet容器

SpringBoot默认支持的是tomcat的Servlet容器

 引入web模块默认使用嵌入式的Tomcat作为Servlet容器； 

```xml

<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



我们也可以使用Jetty和Undertow作为SpringBoot的Servlet容器

使用Jetty

```xml
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring‐boot‐starter‐tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
<!--        !‐‐引入其他的Servlet容器Jetty-->
        <dependency>
            <artifactId>spring-boot-starter-jetty</artifactId>
            <groupId>org.springframework.boot</groupId>
        </dependency>
```

使用undertow

```xml
     <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring‐boot‐starter‐tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
<!--        !‐‐引入其他的Servlet容器undertow-->
        <dependency>
            <artifactId>spring-boot-starter-undertow</artifactId>
            <groupId>org.springframework.boot</groupId>
        </dependency>
```

## 嵌入式Servlet容器的自动配置与案例

## 嵌入式Servlet容器的启动原理

## 使用外置的Servlet容器

在SpringBoot中使用嵌入式Servlet容器是将应用打成可执行的jar包

其优点：简单、便携； 

缺点：默认不支持JSP、优化定制比较复杂（使用定制器【ServerProperties、自定义 EmbeddedServletContainerCustomizer】，自己编写嵌入式Servlet容器的创建工厂【EmbeddedServletContainerFactory】） 我们可以使用外置的Servlet容器：使用我们外部安装的Tomcat并将应用以war包的方式打包；



1. 必须创建一个war项目；（利用idea创建好目录结构） 

2. 将嵌入式的Tomcat指定为provided； 

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐tomcat</artifactId> 
    <scope>provided</scope>
</dependency>
```

发现多了一个SpringBootServletInitializer的子类

```java
public class ServletInitializer extends SpringBootServletInitializer { 
    @Override 
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) { 
        //传入SpringBoot应用的主程序 
        return application.sources(SpringBootWebJspApplication.class); 
    } 
}
```

4. 启动服务器就可以使用； 发现是先启动Servlet容器tomcat，在启动SpringBoot应用的



# SpringBoot操作数据库

## 集成JDBC

1. 创建测试数据

2. 创建项目springbootdata-mysql

 注意引入的模块

![1605356975499](/Users/liqunzhang/Library/Application Support/typora-user-images/1605356975499.png)

项目创建好以后，发现系统自动给我们导入了依赖

```xml
 	   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

3. 编写yml文件以连接数据库

```properties
spring:
	datasource: 
		username: root 
		password: 123456 #?serverTimezone=UTC解决时区的报错 
		url: jdbc:mysql://localhost:3306/springboot	serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8 
		driver-class-name: com.mysql.cj.jdbc.Driver
```

4. 配置完后，我们就可以直接去使用了，因为SpringBoot已经默认帮我们进行了自动配 置

```java
  @Test
    public void testConnection() throws SQLException {
        System.out.println(dataSource.getClass());
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        connection.close();
    }
```

我们可以看到他默认给我们配置的数据源为 : HikariDataSource 

因为在DataSourceAutoConfiguration类中，默认帮我们配置的是Hikari数据源

```java
	@Conditional({DataSourceAutoConfiguration.PooledDataSourceCondition.class})
    @ConditionalOnMissingBean({DataSource.class, XADataSource.class})
    @Import({Hikari.class, Tomcat.class, Dbcp2.class, OracleUcp.class, Generic.class, DataSourceJmxConfiguration.class})
    protected static class PooledDataSourceConfiguration {
        protected PooledDataSourceConfiguration() {
        }
    }
```

**HikariDataSource** **号称** **Java WEB** **当前速度最快的数据源，相比于传统的** **C3P0** 、**DBCP**、**Tomcat**  、 **jdbc** **等连接池更加优秀；** 

我们可以使用 **spring.datasource.type** 指定自定义的数据源类型，值要使用的连接池实现的完全限定名。

5. JdbcTemplate 

关于数据源我们并不做介绍，有了数据库连接，显然就可以 CRUD 操作数据库了。但是我们需要先了解 一个对象 JdbcTemplate



关于JdbcTemplate，在JDBC的时候已经接触过

1. 有了数据源(com.zaxxer.hikari.HikariDataSource)，然后可以拿到数据库连接 (Connection)，有了连接，就可以使用原生的 JDBC 语句来操作数据库； 
2. 即使不使用第三方第数据库操作框架，如 MyBatis等，Spring 本身也对原生的JDBC 做了轻量级的封装，即 JdbcTemplate 。 

3. 数据库操作的所有 CRUD 方法都在 JdbcTemplate 中。 
4. SpringBoot不仅提供了默认的数据源，同时默认配置好了JdbcTemplate放在了容器中，只需自行注入即可使用 

5. JdbcTemplate在SpringBoot的自动配置依赖JdbcTemplateConfifiguration 类 



**JdbcTemplate**主要提供以下几类方法 

- execute()：可以用于执行任何SQL语句，一般用于执行DDL语句； 

- update()及batchUpdate())：update方法用于执行增删改查等语句；batchUpdate 方法用于执行批处理相关语句； 

- query方法及queryForXXX方法：用于执行查询相关语句； 

- call方法：用于执行存储过程、函数相关语句



## 集成Druid

1 .添加Druid依赖

```xml
   <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.3</version>
        </dependency>
```



2. 配置相关参数

```properties
spring:
  datasource:
    username: root
    password: zaxscd
    url: jdbc:mysql://localhost:3306/springboot?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    driver-class-name: com.mysql.cj.jdbc.Driver
    #Spring Boot 默认是不注入这些属性值的，需要自己绑定 #druid 数据源专有配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #配置监控统计拦截的filters，stat:监控统计、log4j：日志记录、wall：防御sql注入 
    #如果允许时报错 java.lang.ClassNotFoundException: org.apache.log4j.Priority
    #则导入 log4j 依赖即可，Maven 地址： https://mvnrepository.com/artifact/log4j/log4j
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
    type: com.alibaba.druid.pool.DruidDataSource
```

注意：不要忘了切换数据源的type类型



3. 我们需要 自己添加 DruidDataSource 组件到容器中，并绑定属性

```java
@Configuration
public class DruidConfig {
    @ConfigurationProperties(value = "spring.datasource")
    @Bean
    public DruidDataSource getDruidDataSource() {
        return new DruidDataSource();
    }
}
```

4. 测试类中注入DataSource测试

```java
 
	@Autowired 
	DataSource dataSource;

	@Test
    public void testConnection() throws SQLException {
        System.out.println(dataSource.getClass());
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        DruidDataSource druidDataSource = (DruidDataSource) dataSource;
        System.out.println("最大连接数" +druidDataSource.getMaxActive());
        System.out.println("最大初始化连接数" +druidDataSource.getInitialSize());
        connection.close();
    }
```

Druid 数据源具有监控的功能，并提供了一个 web 界面方便用户查看，类似安装路由器时提供的一个默认的 web 页面。 

我们可以设置 Druid 的后台管理页面，比如 登录账号、密码 等；配置后台管理； 

```java
 //配置 Druid 监控管理后台的Servlet；
    // 内置 Servlet 容器时没有web.xml文件，所以使用 Spring Boot 的注册 Servlet 方式
    @Bean
    public ServletRegistrationBean statViewServlet(){
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid/*");
        Map<String, String> initParam = new HashMap<>();
        initParam.put("loginUsername", "admin");//后台管理界面的登录账号
        initParam.put("loginPassword", "123456");//后台管理界面的登录密码
        //仅允许本机访问
        initParam.put("allow", "localhost");
        //initParams.put("allow", "localhost")：表示只有本机可以访问 
        //initParams.put("allow", "")：为空或者为null时，表示允许所有访问
        bean.setInitParameters(initParam);//设置初始化参数
        return bean;
    }
```

配置完毕后，我们可以选择访问 ： http://localhost:8080/druid/login.html 

**配置** **Druid web** **监控** **fifilter** **过滤器**

```java
 //配置 Druid web 监控 filter 过滤器
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean<javax.servlet.Filter> bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        //exclusions：设置哪些请求进行过滤排除掉，从而不进行统计
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js,*.css,/druid/*,/jdbc/*");
        bean.setInitParameters(initParams);
        //"/*" 表示过滤所有请求
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
```



## 集成Mybatis

1. 导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.3</version>
</dependency>

```

2. 配置数据库连接池

使用Druid连接池，和之前一样

3. 创建实体类

```java
public class Department {
    private int id;
    private String departmentName;
}
```



### 注解版

4. 创建Mapper目录和接口(注解版为例)

```java
@Mapper   //可以在此处写@Mapper注解或者在主启动类上加@MapperScan(value = "com.zlq.mapper")都可以表示这是个Mapper文件
@Repository
public interface DepartmentMapper {
    @Select("select * from department")
    List<Department> getDepartments();

    @Select("select * from department where id = #{id}")
    Department getDepartment(int id);
}
```

问题：在数据库表中的字段是department_name,而在实体类中的属性为departmentName，所以我们要自定义一个Mybatis配置规则，给容器中添加一个ConfifigurationCustomizer

来修改驼峰命名规则

```java
@org.springframework.context.annotation.Configuration
public class MybatisConfig {
    @Bean
    public ConfigurationCustomizer configurationCustomizer(){
        return new ConfigurationCustomizer() {
            @Override
            public void customize(Configuration configuration) {
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

5. 自行编写Controller类



### 配置版

4. 创建Mapper接口 ，EmployeeMapper

```java
@Mapper
@Repository
public interface EmployeeMapper {
    int addEmployee(Employee employee);
    int deleteEmployee(int id);
    int updateEmployee(Employee employee);
    int queryEmployeeById(int id);    
    int queryAll();
}

```

5. 解决Maven静态资源过滤问题

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

6. 既然已经提供了myBatis的映射配置文件，自然要告诉spring boot这些文件的位置

```properties
#指定myBatis的核心配置文件与Mapper映射文件
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
##指定对应实体类的路径
mybatis.type-aliases-package=com.zlq.pojo

```

7. 创建Mapper.xml文件 EmployeeMapper.xml 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zlq.mapper.EmployeeMapper">
    
    <resultMap id="EmployeeMap" type="Employee">
        <id property="id" column="eid"/>
        <result property="lastName" column="lastName"/>
        <result property="email" column="email"/>
        <result property="gender" column="gender"/>
        <result property="birth" column="birth"/>
        <association property="Department" javaType="Department">
            <id property="id" column="did"/>
            <result property="departmentName" column="dname"/>
        </association>
    </resultMap>
    <insert id="addEmployee" parameterType="Employee">
        insert into employee (last_name,email,gender,department,birth)
        values(#{lastName},#{email},#{gender},#{department},#{birth})
    </insert>

    <delete id="deleteEmployee" parameterType="int">
        delete from employee where id = #{id}
    </delete>

    <update id="updateEmployee" parameterType="employee">
        update employee set last_name = #{lastName},email = #{email},gender = #{gender},
        department = #{department},birth = #{birth} where id = #{id}
    </update>

    <select id="queryEmployeeById" parameterType="int" resultType="Employee">
        select * from employee where id = #{id}
    </select>

    <select id="queryAll" resultType="Employee">
        select * from employee
    </select>
</mapper>
```

8.自行编写Controller类 

# 异步任务
异步处理是比较常用的，比如我们在网站上发送邮件，后台会去发送邮件，此时前台会造成响应
延迟不动，直到邮件发送完毕，响应才会成功，所以我们一般会采用多线程的方式去处理这些任务。
编写方法：假装正在处理数据，使用线程设置一些延时，模拟同步等待的情况

1. 编写Service类
```java
@Service
@Async
public class AsyncService {
    public void Process(){
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("数据在处理中。。。。。。");
    }
}
```
2. 编写Controller类
```java
@Controller
public class AsyncController {
    @Autowired
    AsyncService asyncService;
    @RequestMapping("/process")
    @ResponseBody
    public String process(){
        asyncService.Process();
        return "数据处理成功！";
    }
}
```
发现访问http://localhost:8080/process进行测试，5秒后才出现success，
这是同步等待的情况
问题：我们如果想让用户直接得到消息，就在后台使用多线程的方式进行处理即可，但是每次都需
要自己手动去编写多线程的实现的话，太麻烦了，我们只需要用一个简单的办法，在我们的方法上
加一个简单的注解即可，如下：
- 给hello方法添加@Async注解，SpringBoot就会自己开一个线程池，进行调用
- 要让这个@Async注解生效，我们需要在主程序上添加@EnableAsync ，开启异步注解功能；

# 定时任务
  项目开发中经常需要执行一些定时任务，比如需要在每天凌晨的时候，
  分析一次前一天的日志信息，Spring为我们提供了异步执行任务调度的方式，提供了两个接口。
  - TaskExecutor接口
  - TaskScheduler接口

  两个注解：
  - @EnableScheduling
  - @Scheduled

 1. 在被执行的方法上添加@Schediled，里面的正则表达式用法 见<http://www.bejson.com/othertools/cron/>
 ```java
@Service
public class ScheduledService {

    @Scheduled(cron = "0 * * * * 0-7")
    public void scheduledMethod(){
        System.out.println("方法被执行。。。。。。");
    }
}
 ```

2.调用定时方法
```java
@Controller
public class ScheduledController {
    @Autowired
    ScheduledService scheduledService;
    @RequestMapping("/scheduled")
    @ResponseBody
    public String scheduled(){
        scheduledService.scheduledMethod();
        return "Hello";
    }
}
```

3. 需要在主程序上增加@EnableScheduling 开启定时任务功能

# 邮件任务
邮件任务在日常生活中非常常见，SpringBoot帮我们做了支持

1. 需要导入的依赖
```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-mail</artifactId>
           </dependency>
           <dependency>
               <groupId>com.sun.mail</groupId>
               <artifactId>jakarta.mail</artifactId>
               <version>1.6.4</version>
               <scope>compile</scope>
           </dependency>
```

2. 原理：
```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({MimeMessage.class, MimeType.class, MailSender.class})
@ConditionalOnMissingBean({MailSender.class})
@Conditional({MailSenderAutoConfiguration.MailSenderCondition.class})
@EnableConfigurationProperties({MailProperties.class})
@Import({MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class})
public class MailSenderAutoConfiguration {
    public MailSenderAutoConfiguration() {
    }
 
   
}

```
在MailSenderJndiConfiguration类里面，JavaMailSenderImpl对象注入到了容器中
```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnClass({Session.class})
@ConditionalOnProperty(
    prefix = "spring.mail",
    name = {"jndi-name"}
)
@ConditionalOnJndi
class MailSenderJndiConfiguration {
    private final MailProperties properties;

    @Bean
    JavaMailSenderImpl mailSender(Session session) {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        sender.setDefaultEncoding(this.properties.getDefaultEncoding().name());
        sender.setSession(session);
        return sender;
    }
}
```
在MailProperties封装了这么多属性，因此我们需要在配置文件中以spring.mail为前缀注入相关属性

```java
@ConfigurationProperties(
    prefix = "spring.mail"
)
public class MailProperties {
    private static final Charset DEFAULT_CHARSET;
    private String host;
    private Integer port;
    private String username;
    private String password;
    private String protocol = "smtp";
    private Charset defaultEncoding;
    private Map<String, String> properties;
    private String jndiName;
}
```

3. 测试邮件发送
```java
    @Autowired
    JavaMailSenderImpl Sender;
    @Test   //发送简单邮件
    void contextLoads() {
        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
        simpleMailMessage.setSubject("早上好！");
        simpleMailMessage.setText("今天又是元气满满的一天啊！~~~~~" );
        simpleMailMessage.setTo("804492004@qq.com");
        simpleMailMessage.setFrom("804492004@qq.com");
        Sender.send(simpleMailMessage);
    }


    @Test  //发送复杂邮件，包括文件
    void contextLoads2() throws MessagingException {
        MimeMessage mimeMessage = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
        helper.setSubject("晚上好打工人！");
        helper.setText("<b style='color:blue'>明天也是开开森森写代码的一天哦~~~</b>",true);
        helper.addAttachment("天气.jpeg",new File("C:\\Users\\Administrator\\Desktop\\1.jpeg"));
        helper.setTo("804492004@qq.com");
        helper.setFrom("804492004@qq.com");
        javaMailSender.send(mimeMessage);
```