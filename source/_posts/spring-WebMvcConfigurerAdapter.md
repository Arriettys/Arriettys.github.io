---
title: Spring MVC 配置类 WebMvcConfigurerAdapter
date: 2019-03-28 19:50:39
tags: SpringBoot
categories: Spring
---
WebMvcConfigurerAdapter配置类是spring提供的一种配置方式，采用JavaBean的方式替代传统的基于xml的配置来对spring框架进行自定义的配置。因此，在spring boot提倡的基于注解的配置，采用“约定大于配置”的风格下，当需要进行自定义的配置时，便可以继承WebMvcConfigurerAdapter这个抽象类，通过JavaBean来实现需要的配置。
<!-- more -->
## 1. WebMvcConfigurerAdapter常用的方法
### 1.1 addInterceptors：拦截器
在之前Xml配置形式天下的时候，我们都是在spring-mvc.xml配置文件内添加<mvc:interceptor\>标签配置拦截器。
```
@Override
public void addInterceptors(InterceptorRegistry registry) 
{
	super.addInterceptors(registry);
	registry.addInterceptor(new TestInterceptor()).addPathPatterns("/**");
}
```
InterceptorRegistry内的addInterceptor需要一个实现HandlerInterceptor接口的拦截器实例，addPathPatterns方法用于设置拦截器的过滤路径规则。
```
public class LoginInterceptor extends HandlerInterceptorAdapter 
{
	private static final Logger logger = LoggerFactory.getLogger(LoginInterceptor.class);
	@Override
    	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception 
    	{
        logger.info("-----------------------------");
        logger.info(request.getRequestedSessionId());
        logger.info("-----------------------------");
        return true;
    }
}
```

### 1.2 addCorsMappings：跨域
CORS（Cross-Origin Resource Sharing）"跨域资源共享"，是一个W3C标准，它允许浏览器向跨域服务器发送Ajax请求，打破了Ajax只能访问本站内的资源限制，CORS在很多地方都有被使用，微信支付的JS支付就是通过JS向微信服务器发送跨域请求。开放Ajax访问可被跨域访问的服务器大大减少了后台开发的工作，前后台工作也可以得到很好的明确以及分工
```
/**
 * 跨域CORS配置
 * @param registry
 */
@Override
public void addCorsMappings(CorsRegistry registry) 
{
        super.addCorsMappings(registry);
        registry.addMapping("/**")
                .allowedHeaders("*")
                .allowedMethods("POST","GET")
                .allowedOrigins("http://...")
                .allowCredentials(true);
}
```
同样是在上面的webConfig中重写

### 1.3 addViewControllers：跳转指定页面
这一个配置在之前是经常被使用到的，最经常用到的就是"/"、"/index"路径请求时不通过@RequestMapping配置，而是直接通过配置文件映射指定请求路径到指定View页面，当然也是在请求目标页面时不需要做什么数据处理才可以这样使用，配置内容如下所示：
```
/**
  * 视图控制器配置
  * @param registry
  */
@Override
public void addViewControllers(ViewControllerRegistry registry) 
{
        super.addViewControllers(registry);
        registry.addViewController("/").setViewName("/index");
}
```

### 1.4 resourceViewResolver：视图解析器
这个对我们来说很熟悉，只要我们配置html、Jsp页面视图时就会用到WebMvcConfigurerAdapte里InternalResourceViewResolver配置类（内部类），然后设置preffix、suffix参数进行配置视图文件路径前缀与后缀。这个我们以前经常在xml文件中配置

![](https://s2.ax1x.com/2019/03/28/AwofxS.png)
```
/**
  * 配置请求视图映射
  * @return
  */
@Bean
public InternalResourceViewResolver resourceViewResolver()
{
        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        //请求视图文件的前缀地址
        internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
        //请求视图文件的后缀
        internalResourceViewResolver.setSuffix(".jsp");
        return internalResourceViewResolver;
}
/**
  * 视图配置
  * @param registry
  */
@Override
public void configureViewResolvers(ViewResolverRegistry registry) 
{
        super.configureViewResolvers(registry);
        registry.viewResolver(resourceViewResolver());
        /*registry.jsp("/WEB-INF/jsp/",".jsp");*/
}
```
上述代码中方法resourceViewResolver上配置了@Bean注解，该注解会将方法返回值加入到SpringIoc容器内。
而在configureViewResolvers方法内配置视图映射为resourceViewResolver方法返回的InternalResourceViewResolver实例，这样完成了视图的配置。在下面还有注释掉的一部分代码，这块代码很神奇，我们先来看看org.springframework.web.servlet.config.annotation.ViewResolverRegistry源码：
```
package org.springframework.web.servlet.config.annotation;

public class ViewResolverRegistry 
{
    ...//省略代码
    public UrlBasedViewResolverRegistration jsp() 
    {
        return this.jsp("/WEB-INF/", ".jsp");
    }

    public UrlBasedViewResolverRegistration jsp(String prefix, String suffix) 
    {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix(prefix);
        resolver.setSuffix(suffix);
        this.viewResolvers.add(resolver);
        return new UrlBasedViewResolverRegistration(resolver);
    }
}
...//省略代码
```
可以看到上述源码中有两个jsp方法，而没有参数的方法恰恰跟我们配置的内容一样，这一点看来是Spring早就根据用户使用习惯添加的默认配置，同样也提供了自定义配置Jsp相关的前缀、后缀内容的方法，
方法内部同样是实例化了一个InternalResourceViewResolver视图映射类，并将实例添加到了viewResolvers集合内。
### 1.5 configureMessageConverters：信息转换器
这个配置一般针对于Api接口服务程序，配置在请求返回时内容采用什么转换器进行转换，我们最常用到的就是fastJson的转换，配置如下所示：
```
/**
  * 消息内容转换配置
  * 配置fastJson返回json转换
  * @param converters
  */
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) 
{
        //调用父类的配置
        super.configureMessageConverters(converters);
        //创建fastJson消息转换器
        FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
        //创建配置类
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        //修改配置返回内容的过滤
        fastJsonConfig.setSerializerFeatures(
                SerializerFeature.DisableCircularReferenceDetect,
                SerializerFeature.WriteMapNullValue,
                SerializerFeature.WriteNullStringAsEmpty
        );
        fastConverter.setFastJsonConfig(fastJsonConfig);
        //将fastjson添加到视图消息转换器列表内
        converters.add(fastConverter);
}
```
内容转换都是针对面向接口进行编写的实现类，都必须implements HttpMessageConverter接口完成方法的实现。有关HttpMessageConverter接口的讲解可以[看这里](https://my.oschina.net/lichhao/blog/172562)

### 1.6 addFormatters 
当请求的参数中带有日期的参数的时候，可以在此配置formatter使得接收到日期参数格式统一。
```
@Override
public void addFormatters(FormatterRegistry registry) 
{
        registry.addFormatter(new Formatter<Date>() 
        {
            @Override
            public Date parse(String date, Locale locale) 
            {
                return new Date(Long.parseLong(date));
            }

            @Override
            public String print(Date date, Locale locale) 
            {
                return Long.valueOf(date.getTime()).toString();
            }
        });
}
```

### 1.7 addResourceHandlers：静态资源
```
 @Override
 public void addResourceHandlers(ResourceHandlerRegistry registry) 
 {
 	//处理静态资源的，例如：图片，js，css等
     	registry.addResourceHandler("/resource/**").addResourceLocations("/WEB-INF/static/");
 }
```

## 2. WebMvcConfigurerAdapter使用方式
### 2.1 过时方式：继承WebMvcConfigurerAdapter
**Spring 5.0 以后WebMvcConfigurerAdapter会取消掉
WebMvcConfigurerAdapter是实现WebMvcConfigurer接口**
```
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter 
{
	//TODO
}
```
### 2.2 实现WebMvcConfigurer
```
@Configuration
public class WebMvcConfg implements WebMvcConfigurer 
{
	//TODO
}
```
### 2.3 直接继承WebMvcConfigurationSupport
```
@Configuration
public class WebMvcConfg extends WebMvcConfigurationSupport 
{
        //TODO
}
```
建议后两种


参考自：
https://www.jianshu.com/p/2c2cdb80fe47
https://www.cnblogs.com/jy107600/p/9484850.html