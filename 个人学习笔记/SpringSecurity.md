# 简介



## 概述

SpringSecurity正如它的名字一样，是Spring提供的一套安全框架，提供Web安全应用方面的一站式解决方案。使用SpringSecurity安全框架，我们可以通过配置的方式实现对资源的访问限制。



关于安全方面主要是**认证**和**授权**两大功能

**认证**：通俗点说就是验证登陆的用户。通过认证来检验登陆的用户在系统中是否合法，是否能正常访问系统中的资源，一般通过验证用户名和密码进行认证

**授权**：通俗点说就是对登陆的不同用户给予哪些可以执行的权限。通过授权，有些资源可以使该用户进行访问（读）和操作（增删改），某些资源则不允许用户的访问和操作。通过授权可以给予不同用户不同的角色，进而不同的用户就会拥有不同的权限



**同款产品**：shiro安全框架

和shiro框架相比：

优点

- SpringSecurity和当今最流行Spring框架无缝结合，项目中如果以Spring/SpringBoot为基础，对SpringSecurity进行整合将非常方便
- SpringSecurity有着比Shiro更加丰富的权限控制和安全防护功能
- SpringSecurity有着比shrio更活跃的社区

缺点

- SpringSecurity是一款重量级安全框架，配置和使用上比较复杂，相比较Shiro上手要困难些
- SpringSecurity依赖性强，需要依赖Spring框架，而Shiro可以依附于任何框架



# SpringSecurity的基本原理

想要对Web资源进行保护，最好的办法莫过于Filter

所以Spring Security在我们进行用户认证以及授予权限的时候，通过各种各样的拦截器来控制权限的访问，从而实现安全。

SpringSecurity提供若干个过滤器，形成一个**过滤器链**，通过拦截servlet请求，将这些请求转给认证和访问决策管理器处理，从而增强安全性

需要注意的是，SpringSecurity中的过滤器在spring项目的基础上由spring帮我们配置好了，不需要我们额外的进行配置，但是如果我们不在spring框架的基础上使用SpringSecurity的话，需要我们额外的配置过滤器



## 一.过滤器链（了解）

>1. WebAsyncManagerIntegrationFilter：将 Security 上下文与 Spring Web 中用于处理异步请求映射的 WebAsyncManager 进行集成。
>
>2. SecurityContextPersistenceFilter：在每次请求处理之前将该请求相关的安全上下文信息加载到 SecurityContextHolder 中，然后在该次请求处理完成之后，将SecurityContextHolder 中关于这次请求的信息存储到一个“仓储”中，然后将SecurityContextHolder 中的信息清除，例如在 Session 中维护一个用户的安全信息就是这个过滤器处理的。
>
>3. HeaderWriterFilter：用于将头信息加入响应中。
>
>4. CsrfFilter：用于处理跨站请求伪造。
>
>5. LogoutFilter：用于处理退出登录。
>
>6. **UsernamePasswordAuthenticationFilter** 处理身份验证表单提交,验证表单中的username和password，只对 /login的post请求有效。该过滤器是过滤器链的最后一个过滤器，根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果访问受限会抛出相关异常，并由 ExceptionTranslationFilter 过滤器进行捕获和处理。
>
>7. DefaultLoginPageGeneratingFilter：如果没有配置登录页面，那系统初始化时就会配置这个过滤器，并且用于在需要进行登录时生成一个登录表单页面。
>
>8. BasicAuthenticationFilter：检测和处理 http basic 认证。
>
>9. RequestCacheAwareFilter：用来处理请求的缓存。
>
>10. SecurityContextHolderAwareRequestFilter：主要是包装请求对象 request。 
>
>11. AnonymousAuthenticationFilter：检测 SecurityContextHolder 中是否存在Authentication 对象，如果不存在为其提供一个匿名 Authentication。 
>
>12. SessionManagementFilter：管理 session 的过滤器
>
>13. **ExceptionTranslationFilter,**异常过滤器，用来处理认证过程中抛出的异常,该过滤器不需要我们配置，对于前端提交的请求会直接放行，捕获后续抛出的异常并进行处理（例如：权限访问限制）。
>
>14. **FilterSecurityInterceptor**：方法级别的权限过滤器，位于过滤器链的最底部，该过滤器是过滤器链的最后一个过滤器，根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果访问受限会抛出相关异常，并由 ExceptionTranslationFilter 过滤器进行捕获和处理。
>
>    
>
>15. RememberMeAuthenticationFilter：当用户没有登录而直接访问资源时, 从 cookie 里找出用户的信息, 如果 Spring Security 能够识别出用户提供的 remember me cookie, 用户将不必填写用户名和密码, 而是直接登录进入系统，该过滤器默认不开启。

**Spring Security 采取过滤链实现认证与授权，只有当前过滤器通过，才能进入下一个过滤器**

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420200147563.png" alt="image-20210420200147563" style="zoom:50%;" />





## 二.UserDetailsService接口

当我们什么都没配置的时候在启动项目的时候SpringSecurity会直接给我们生成一个密码用于用户登录，但是实际项目中我们的用户名和密码都要从数据库中查到。我们可以通过自定义逻辑实现认证，**只需要实现UserDetailsService接口即可**

该接口加载用户特定数据的核心接口。我们可以通过该接口来实现自定义的权限控制功能

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210418151102997.png" alt="image-20210418151102997" style="zoom:50%;" />

返回一个UserDetails

UserDetails源码如下

```java
public interface UserDetails extends Serializable {
	//返回用户的权限	
	Collection<? extends GrantedAuthority> getAuthorities();
  
	//返回的密码
	String getPassword();
  
	//返回的用户名
	String getUsername();
  
	//用户的帐户是否已过期
	boolean isAccountNonExpired();
  
	//用户是否锁定
	boolean isAccountNonLocked();
  
	//指示用户的凭据（密码）是否已过期。 过期的凭据会阻止身份验证
	boolean isCredentialsNonExpired();
  
	//用户是否可用。 禁用的用户无法通过身份验证
	boolean isEnabled();
}
```

要想返回 UserDetails 的实例就只能返回接口的实现类。SpringSecurity 中提供了如下的实例。对于我们只需要使用里面的 User 类即可。

![image-20210418152025706](/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210418152025706.png)

UserDetails的实现类是User类,以后我们只需要return一个User类即可,其中构造方法有两个，调用其中任何一个都可以实例化。

UserDetails 实现类 User 类的实例。而三个参数的构造方法实际上也是调用 7 个参数的构造方法。

```java
 public User(String username, String password, Collection<? extends GrantedAuthority> authorities) {
        this(username, password, true, true, true, true, authorities);
 }
```

username :用户名  password :密码  authorities ：用户具有的权限。此处不允许为 null

此处的用户名应该是客户端传递过来的用户名。而密码应该是从数据库中查询出来的密码。Spring Security 会根据 User 中的 password 和客户端传递过来的 password 进行比较。如果相同则表示认证通过，如果不相同表示认证失败。

authorities包含的所有内容为此用户具有的权限，如有里面没有包含某个权限，而在做某个事情时必须包含某个权限则会出现 403。

## 三.PasswordEncoder接口

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210418153018088.png" alt="image-20210418153018088" style="zoom:43%;" />

用来编码密码的服务接口，官方建议我们首选使用其实现类BCryptPasswordEncoder，他是一个强大的密码解析器，BCryptPasswordEncoder 是对 bcrypt 强散列方法的具体实现。是基于 Hash 算法实现的单向加密。可以通过 strength 控制加密强度，默认 10.



## 四.设置用户名和密码的基本方式

1. 通过配置文件

   <img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210418154406172.png" alt="image-20210418154406172" style="zoom:50%;" />

2. 通过配置类

   ```java
   @Configuration
   @EnableWebSecurity
   public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
       @Override
       protected void configure(AuthenticationManagerBuilder auth) throws Exception {
           BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
           String password = encoder.encode("123456");
           auth.inMemoryAuthentication().withUser("admin")
                   .password(password).roles("admin");
       }
       @Bean
       public PasswordEncoder passwordEncoder() {
           return new BCryptPasswordEncoder();
       }
   }
   ```

   

3. **自定义登录逻辑（推荐且常用）**

   - 编写配置类

     ```java
     @Configuration
     @EnableWebSecurity
     public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
         @Resource
         private MyUserDetailService userDetailsService;
     
         @Override
         protected void configure(AuthenticationManagerBuilder auth) throws Exception {
             auth.userDetailsService(userDetailsService).passwordEncoder(encoder());
         }
         @Bean
         PasswordEncoder passwordEncoder(){
             return new BCryptPasswordEncoder();
         }
     }
     ```

	- 编写实现类返回User对象
	
	  ```java
	  @Service
	  public class MyUserDetailService implements UserDetailsService {
	      @Resource
	      private UsersMapper usersMapper;
	    
	     @Autowired
	     private PasswordEncoder passwordEncoder;
	  
	      @Override
	      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
	     	    //1.查询数据库判断用户名是否存在，如果不存在抛出UsernameNotFoundException异常
	          QueryWrapper<Users> wrapper = new QueryWrapper<>();
	          wrapper.eq("username",username);
	          Users users = usersMapper.selectOne(wrapper);
	          if (null == users){
	              throw new UsernameNotFoundException("用户名不存在！");
	          }
	          List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role,admin");
	      	//2.把查询出来的密码（注册时已经加密过）进行解析，或直接把密码放入构造方法中
	          String password = passwordEncoder.encode("123");
	          return new User(username,password,auths);
	     }
	  }
	  ```
	  
	  



此外，我们还可以配置跳转到自己定制的登录页，登陆成功后的跳转页，以及登录失败后的跳转页

```java
配置类
@Override
protected void configure(HttpSecurity http) throws Exception {
    // 配置认证
    http.formLogin()
    	   // 配置登录页面
        .loginPage("/login.html") 
      	 // 设置登录的url,必须和表单的提交路径相同
        .loginProcessingUrl("/login") 
      	 // 登录成功之后跳转的页面
        .successForwardUrl("/success") 
        	// 登录成功之后跳转的页面
        .failureForwardUrl("/fail");
  
    http.authorizeRequests()
      	 //login.html不需要被认证
        .antMatchers("/login.html").permitAll()
      	//所有请求都必须被认证，必须登录后被访问
        .anyRequest().authenticated();
    // 关闭 csrf
    http.csrf().disable();
}
```

csrf：跨站请求伪造

跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去运行。这利用了web中用户身份验证的一个漏洞：**简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的**。

**例子**

假如一家银行用以运行转账操作的URL地址如下：http://www.examplebank.com/withdraw?account=AccoutName&amount=1000&for=PayeeName

那么，一个恶意攻击者可以在另一个网站上放置如下代码：<img src="http://www.examplebank.com/withdraw?account=Alice&amount=1000&for=Badman">

如果有账户名为Alice的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失1000资金





**注：我们通过前端表单传过来的用户名和密码必须叫username和password**，原因：

我们在登录的时候会执行UsernamePasswordAuthenticationFilter中的attemptAuthentication方法进行身份验证

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420160129379.png" alt="image-20210420160129379" style="zoom:40%;" />

而obtainUsername和obtainPassword方法中接受的参数是在类初始化的时候定义好的

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420160221647.png" alt="image-20210420160221647" style="zoom:40%;" />

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420160351980.png" alt="image-20210420160351980" style="zoom:30%;" />

如果想修改自己定义的名字可以在http.formLogin( )中追加调用

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420160447863.png" alt="image-20210420160447863" style="zoom:39%;" />





## 五：认证流程分析



<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420201558059.png" alt="image-20210420201558059" style="zoom:39%;" />

1.  UsernamePasswordAuthenticationFilter

   <img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420202126520.png" alt="image-20210420202126520" style="zoom:33%;" />

   当前端提交一个post方式登录的表单请求就会被该过滤器拦截，执行其父类AbstractAuthenticationProcessingFilter中的diFilter方法

AbstractAuthenticationProcessingFilter中的doFliter放方法如下

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
			throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;
		//1.判断该请求是否需要认证，如果需要认证直接进入下一个过滤器
		if (!requiresAuthentication(request, response)) {
			chain.doFilter(request, response);
      
			return;
		}

		if (logger.isDebugEnabled()) {
			logger.debug("Request is to process authentication");
		}
		//Authentication类用来储存用户认证信息
		Authentication authResult;

		try {
      //2.这里调用的是UsernamePasswordAuthenticationFilter中的attemptAuthentication方法进行验证，返回的authResult封装到successfulAuthentication中
			authResult = attemptAuthentication(request, response);
			if (authResult == null) {
				// return immediately as subclass has indicated that it hasn't completed
				// authentication
				return;
			}
     	//session策略处理
			sessionStrategy.onAuthentication(authResult, request, response);
		}
  	//3.如果认证失败，调用认证失败的方法
		catch (InternalAuthenticationServiceException failed) {
			logger.error(
					"An internal error occurred while trying to authenticate the user.",
					failed);
			unsuccessfulAuthentication(request, response, failed);

			return;
		}
		catch (AuthenticationException failed) {
			// Authentication failed
			unsuccessfulAuthentication(request, response, failed);

			return;
		}

		// continueChainBeforeSuccessfulAuthentication是用来标记认证是否成功的，默认为false，如果认证成功不会走下一个过滤器
		if (continueChainBeforeSuccessfulAuthentication) {
			chain.doFilter(request, response);
		}
  	//4.调用认证成功的方法
		successfulAuthentication(request, response, chain, authResult);
	}
```
第二步中调用的attemptAuthentication源码

```java
public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
		if (postOnly && !request.getMethod().equals("POST")) { //如果方法不是post就抛出异常
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();
		//使用username，password构造UsernamePasswordAuthenticationToken对象
		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password);

		// Allow subclasses to set the "details" property，将一些详细属性设置到Authentication中如sessionId等
		setDetails(request, authRequest);
  	//调用Authentication中的authenticate方法进行认证
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

UsernamePasswordToken

```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {

	//用于封装未认证的用户权限信息
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
		super(null);
		this.principal = principal;
		this.credentials = credentials;
		setAuthenticated(false);
	}

  //用于封装认证后的用户权限信息
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
		super(authorities);
		this.principal = principal;
		this.credentials = credentials;
		super.setAuthenticated(true); // must use super, as we override
	}
}

```



**Authentication 接口的实现类用于存储用户认证信息：**

```java
public interface Authentication extends Principal, Serializable {
	//用户权限集合
	Collection<? extends GrantedAuthority> getAuthorities();
	//用户的密码
	Object getCredentials();
  //请求携带的属性信息例如sessionId
  Object getDetails();
  //获取登录的主体，自定义登录逻辑中是 UserDetails
	Object getPrincipal();
  //是否被认证
	boolean isAuthenticated();
	//设置是否被认证
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}

```







## 六：鉴权流程分析



- 如果是基于 Session，那么 Spring-security 会对 cookie 里的 sessionid 进行解析，找到服务器存储的 session 信息，然后判断当前用户是否符合请求的要求。

  

- 如果是 token，则是解析出 token，然后将当前请求加入到 Spring-security 管理的权限信息中去

  

**基于token的鉴权流程分析：**

根据用户名密码认证成功  --> 获取用户角色权限值，生成一对key-value键值对（key为username，value为权限值列表 ）

--> 将获取的键值对存入redis中 -->将username相关信息生成token -->浏览器将token存入cookie -->

每次调用api接口验证时都会将token携带到header请求头 -->SpringSecurity解析请求头中的token中的username -->

根据解析的username从redis获取权限列表 -->鉴定当前用户是否有权访问某些资源



总而言之，token就是存入cookie中的一串保存用户相关的信息

<img src="/Library/Java学习文件/笔记整理/SpringSecurity.assets/image-20210420195352019.png" alt="image-20210420195352019" style="zoom:50%;" />

