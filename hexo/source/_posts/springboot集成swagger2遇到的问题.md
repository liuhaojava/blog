---
title: springboot集成swagger2遇到的问题
copyright: true
date: 2019-05-13 16:41:28
tags: 问题及解决
categories: 问题及解决
---

# springboot项目集成swagger2的问题
项目是从SSM项目整合过来的，swagger原本就有的，没有问题。整合到springboot项目时候，http://ip:port/swagger-ui.html会报错。  
**swagger**配置如下：  
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    @Bean
    public Docket docketBean() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo("教育厅一级平台"))
                .select()
                .apis(RequestHandlerSelectors.basePackage("qgs.education.platform.province.controller"))
                //开启swagger
                .paths(PathSelectors.any())
                //禁用swagger
//                .paths(PathSelectors.none())
                .build()
                .globalOperationParameters(setHeaderToken());
    }

    private ApiInfo apiInfo(String title) {
        return new ApiInfoBuilder()
                .title(title)
                .build();
    }

    private Docket docket(String groupName, String title, String pathRegex) {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName(groupName).apiInfo(apiInfo(title))
                .ignoredParameterTypes(ApiIgnore.class)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(regex(pathRegex))
                .build()
                .globalOperationParameters(setHeaderToken());
    }

    private List<Parameter> setHeaderToken() {
        ParameterBuilder tokenPar = new ParameterBuilder();
        List<Parameter> pars = new ArrayList<>();
        tokenPar.name("Token").description("令牌").modelRef(new ModelRef("string")).parameterType("header").required(false).build();
        pars.add(tokenPar.build());
        return pars;
    }
}
```  
**拦截器**配置如下：  
```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Autowired
    private ControllerInterceptor controllerInterceptor;
    @Autowired
    private AuthBusinessInterceptor authBusinessInterceptor;
    @Autowired
    private AuthPlatformInterceptor authPlatformInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(controllerInterceptor);
        registry.addInterceptor(authBusinessInterceptor);
        registry.addInterceptor(authPlatformInterceptor)
                .addPathPatterns("/**");
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("/");

        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/");

        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }

    /**
     * 配置servlet处理
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```  
如上配置访问http://ip:port/swagger-ui.html会报错
![](springboot集成swagger2遇到的问题\swagger1.jpg)  
访问http://ip:port/v2/api-docs正常  
猜想应该是没有找到静态资源的问题，但是拦截器配置那里是加了的
```java
@Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("/");

        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/");

        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
```  
以上代码不起作用。  
网上找了好久，偶然看到一个swagger配置继承WebMvcConfigurationSupport，试了一下，是可以的。修改后的配置：  
**swagger**配置修改后如下：  
```java
@Configuration
@EnableSwagger2
public class SwaggerConfig  extends WebMvcConfigurationSupport {

    @Bean
    public Docket docketBean() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo("教育厅一级平台"))
                .select()
                .apis(RequestHandlerSelectors.basePackage("qgs.education.platform.province.controller"))
                //开启swagger
                .paths(PathSelectors.any())
                //禁用swagger
//                .paths(PathSelectors.none())
                .build()
                .globalOperationParameters(setHeaderToken());
    }

    private ApiInfo apiInfo(String title) {
        return new ApiInfoBuilder()
                .title(title)
                .build();
    }

    private Docket docket(String groupName, String title, String pathRegex) {
        return new Docket(DocumentationType.SWAGGER_2)
                .groupName(groupName).apiInfo(apiInfo(title))
                .ignoredParameterTypes(ApiIgnore.class)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(regex(pathRegex))
                .build()
                .globalOperationParameters(setHeaderToken());
    }

    private List<Parameter> setHeaderToken() {
        ParameterBuilder tokenPar = new ParameterBuilder();
        List<Parameter> pars = new ArrayList<>();
        tokenPar.name("Token").description("令牌").modelRef(new ModelRef("string")).parameterType("header").required(false).build();
        pars.add(tokenPar.build());
        return pars;
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {

        registry.addResourceHandler("/**")
                .addResourceLocations("classpath:/static/");

        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```  
**拦截器**配置修改后如下:  
```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {

    @Autowired
    private ControllerInterceptor controllerInterceptor;
    @Autowired
    private AuthBusinessInterceptor authBusinessInterceptor;
    @Autowired
    private AuthPlatformInterceptor authPlatformInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {

        registry.addInterceptor(controllerInterceptor);
        registry.addInterceptor(authBusinessInterceptor);
        registry.addInterceptor(authPlatformInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/swagger-resources/**",
                "/swagger-ui.html",
                "/v2/api-docs",
                "/webjars/**");
    }
}
``` 
> 注意：`.excludePathPatterns("/swagger-resources/**",
                      "/swagger-ui.html",
                      "/v2/api-docs",
                      "/webjars/**")` 是为了避免swagger-ui.html被拦截。  


重启项目，访问http://ip:port/swagger-ui.html正常。