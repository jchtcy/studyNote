# 一、web开发简介
* 内容协商视图解析器和BeanName视图解析器
* 静态资源（包括webjars）
* 自动注册 Converter，GenericConverter，Formatter
* 支持 HttpMessageConverters （后来我们配合内容协商理解原理）
* 静态index.html 页支持
* 自定义 Favicon
* 自动使用 ConfigurableWebBindingInitializer
# 二、静态资源规则与定制化
* 1、静态资源目录
````
只要静态资源放在类路径下: /static or /public or /resources or /META-INF/resources

访问 ： 当前项目根路径/ + 静态资源名

原理： 静态映射/**。
请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面。

也可以改变默认的静态资源路径，/static，/public,/resources, /META-INF/resources失效
resources:
static-locations: [classpath:/haha/]
````
* 2、静态资源访问前缀
````
spring:
  mvc:
    static-path-pattern: /res/**
````
当前项目 + static-path-pattern + 静态资源名 = 静态资源文件夹下找
* 3、webjar
````
可用jar方式添加css，js等资源文件，
https://www.webjars.org/
例如，添加jquery

引入依赖
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>

访问地址：http://localhost:8080/webjars/jquery/3.5.1/jquery.js 后面地址要按照依赖里面的包路径。
````
# 三、welcome与favicon功能
* 1、欢迎页支持
````
静态资源路径下index.html。(可以配置静态资源路径。但是不可以配置静态资源的访问前缀。)
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page功能失效
  resources:
    static-locations: [classpath:/haha/]

controller能处理/index。
````
* 2、自定义Favicon
````
指网页标签上的小图标。favicon.ico 放在静态资源目录下即可。

spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
````
# 四、静态资源原理-源码分析
* 1、SpringBoot启动默认加载
````
1、xxxAutoConfiguration 类（自动配置类）

2、SpringMVC功能的自动配置类WebMvcAutoConfiguration，生效

@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
    ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
    ...
}

3、给容器中配置的内容：

配置文件的相关属性的绑定：WebMvcProperties==spring.mvc、ResourceProperties==spring.resources

@Configuration(proxyBeanMethods = false)
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {
    ...
}
4、配置类只有一个有参构造器
有参构造器所有参数的值都会从容器中确定
public WebMvcAutoConfigurationAdapter(WebProperties webProperties, WebMvcProperties mvcProperties,
		ListableBeanFactory beanFactory, ObjectProvider<HttpMessageConverters> messageConvertersProvider,
		ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
		ObjectProvider<DispatcherServletPath> dispatcherServletPath,
		ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
	this.mvcProperties = mvcProperties;
	this.beanFactory = beanFactory;
	this.messageConvertersProvider = messageConvertersProvider;
	this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
	this.dispatcherServletPath = dispatcherServletPath;
	this.servletRegistrations = servletRegistrations;
	this.mvcProperties.checkConfiguration();
}
ResourceProperties resourceProperties；获取和spring.resources绑定的所有的值的对象
WebMvcProperties mvcProperties 获取和spring.mvc绑定的所有的值的对象
ListableBeanFactory beanFactory Spring的beanFactory
HttpMessageConverters 找到所有的HttpMessageConverters
ResourceHandlerRegistrationCustomizer 找到 资源处理器的自定义器。
DispatcherServletPath
ServletRegistrationBean 给应用注册Servlet、Filter…
````
* 3、资源处理的默认规则
````
...
public class WebMvcAutoConfiguration {
    ...
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {
        ...
		@Override
		protected void addResourceHandlers(ResourceHandlerRegistry registry) {
			super.addResourceHandlers(registry);
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			ServletContext servletContext = getServletContext();
			addResourceHandler(registry, "/webjars/**", "classpath:/META-INF/resources/webjars/");
			addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
				registration.addResourceLocations(this.resourceProperties.getStaticLocations());
				if (servletContext != null) {
					registration.addResourceLocations(new ServletContextResource(servletContext, SERVLET_LOCATION));
				}
			});
		}
        ...
        
    }
    ...
}

根据上述代码，我们可以同过配置禁止所有静态资源规则。
spring:
  resources:
    add-mappings: false   #禁用所有静态资源规则
````
* 4、静态资源规则
````
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {

    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
            "classpath:/resources/", "classpath:/static/", "classpath:/public/" };

    /**
     * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
     * /resources/, /static/, /public/].
     */
    private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
    ...
}
````
* 5、欢迎页的处理规则
````
public class WebMvcAutoConfiguration {
    ...
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {
        ...
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
  }
}

WelcomePageHandlerMapping的构造方法如下：

WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
                          ApplicationContext applicationContext, Resource welcomePage, String staticPathPattern) {
    if (welcomePage != null && "/**".equals(staticPathPattern)) {
        //要用欢迎页功能，必须是/**
        logger.info("Adding welcome page: " + welcomePage);
        setRootViewName("forward:index.html");
    }
    else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
        //调用Controller /index
        logger.info("Adding welcome page template: index");
        setRootViewName("index");
    }
}
````