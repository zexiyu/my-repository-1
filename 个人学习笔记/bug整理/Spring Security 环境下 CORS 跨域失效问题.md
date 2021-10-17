# Spring Security 环境下 CORS 跨域失效问题

## 一、背景 

  之前写过一篇文章介绍了跨域以及其解决方案，[传送门](http://www.jetchen.cn/jsonp-cors/)，但是在目前的项目中又踩了坑，发现跨域失效。
  配置方式见下图：
![cors01.png](http://www.jetchen.cn/wp-content/uploads/image/20180927/1538055456712875.png)   项目基础框架：spring boot   安全框架：spring security





## 二、原因

  看文末的参考文章（spring boot 的一个 issue），看到了一个很棒的实验（下述代码取于其中）。

```
@Configuration
@Order(Ordered.HIGHEST_PRECEDENCE)
public class DCorsConfiguration {
    @Bean
    public FilterRegistrationBean dgCorsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        org.springframework.web.cors.CorsConfiguration config = new org.springframework.web.cors.CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("http://localhost:5000");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        source.registerCorsConfiguration("/**", config);
        FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
    }
}
```

  然后我们启动 spring boot 项目，观察控制台打印的过滤器 chain 的顺序便会有惊人的发现：     cors filter 是在 spring security filter chain 之后的。

```
2016-11-23 13:35:44.883  INFO 17404 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2016-11-23 13:35:44.884  INFO 17404 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2016-11-23 13:35:44.884  INFO 17404 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2016-11-23 13:35:44.884  INFO 17404 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'OAuth2ClientContextFilter' to: [/*]
2016-11-23 13:35:44.884  INFO 17404 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2016-11-23 13:35:44.887  INFO 17404 --- [ost-startStop-1] .s.DelegatingFilterProxyRegistrationBean : Mapping filter: 'springSecurityFilterChain' to: [/*]
2016-11-23 13:35:44.887  INFO 17404 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'corsFilter' to: [/*]
```

  所以，问题很清晰了，问题在于 spring security 的 过滤器链，我们只需要将手动创建的跨域 filter 置于 spring security filter chain 之前即可。





## 三、解决

####   ① 写过滤器 

```
/**
 * @Author: Jet
 * @Description: 手动生成过滤器，目的是跨域
 * @Date: 2018/5/23 16:55
 */
public class WebSecurityCorsFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse res = (HttpServletResponse) response;
        res.setHeader("Access-Control-Allow-Origin", "*");
        res.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE, PUT");
        res.setHeader("Access-Control-Max-Age", "3600");
        res.setHeader("Access-Control-Allow-Headers", "Authorization, Content-Type, Accept, x-requested-with, Cache-Control");
        chain.doFilter(request, res);
    }
    @Override
    public void destroy() {
    }
}
```

####   ② 将跨域的过滤器置于 spring security filter chain 之前

​    见下图红框中代码

```
.and().addFilterBefore(new WebSecurityCorsFilter(), ChannelProcessingFilter.class) // 保证跨域的过滤器首先触发
```


![cors02.png](http://www.jetchen.cn/wp-content/uploads/image/20180927/1538056851542273.png)

## 四、小结

  spring boot 这种用于快速开发 java 项目的轮子，使用很便捷，大大加快了前期开发的效率，但是，正因为其采用了大量的默认配置，有时候碰上问题的时候，我们需要大肆翻源码，逛论坛，找解决帖，双刃剑呐。
  正如此文所描述的使用 spring security 的过程中碰到的跨域的坑，不下点功夫，着实难解决。   其实开发中还遇到过 spring security 带来的 iframe 的问题、HSTS 问题等，暂不赘述。

