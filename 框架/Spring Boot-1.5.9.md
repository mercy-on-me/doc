# Spring Boot (Version : 1.5.9)
## Spring Boot 简介
- 简化 Spring 应用开发的一个框架.
- 整个 Spring 技术栈的一个大整合.
- J2EE 开发的一站式解决方案.

## 1. 微服务介绍
- 传统的应用 : 所有的功能都写在一个应用内,然后放在服务器进行运行,比如如果压力过大的话,就可以通过负载均衡进行请求分发.但是在日益增长的需求增加,一个应用维护,开发都比较困难,所以微服务应运而生.
- 微服务 : ==*一种架构风格,将每一个功能元素独立出来,放到一个独立的服务器中. 每一个功能单元都是一个可独立替换,独立升级的软件单元.*==

## 2. 急速入门
- 功能描述 : 浏览器发送请求,服务器接收请求并处理,响应 "hello spring boot" 字符串
==**代码实现**==

==主程序==
```java
/**
 *  @SpringBootApplication : 说明这是一个 SpringBoot 注解
 */
@SpringBootApplication
public class AutoconfigurationApplication {

    public static void main(String[] args) {
        // 将程序启动起来
        SpringApplication.run(AutoconfigurationApplication.class, args);
    }

}
```
==controller==

```java
//@Controller   
//@ResponseBody
@RestController  // 等于@Controller + @ResponseBody
public class HelloSpringBoot {

    @RequestMapping("/hello")
    
    public String helloSpringboot(){
        return "hello spring boot";
    }

}
```
==测试==

```josn
http://localhost:8080/hello
```

## 3. 功能使用
### 3.1 从配置文件获取值以及 .properties 配置文件编码问题
#### 3.1.1 @ConfigurationProperties 方式
1. 实现方式:
    - 引入依赖
    
    ```xml
    <!--可以提供配置文件中的提示功能,然后重新启动一个 springboot 程序就可以使用了-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    ```
    - 在==*要绑定数据的类上*==使用以下注解
        
    ```java
    //将次此类交给Spring容器管理
    @Component 
    //告诉 SpringBoot 将此类中的所有属性和配置文件中的相关配置进行绑定,将配置文件中的每一个属性映射到这个累中
    // prefix 的值是告诉springboot 要和配置文件中的那个标签下的属性进行绑定
    @ConfigurationProperties(prefix="person")      
    ```
    
    - 类中的属性和配置文件中==*一一对应,属性名一定要一样.*==
    
    - 使用 SpringBootTest 测试,注入 javabean,然后编写测试方法,运行
    
1. 代码实现
    ==Person 类,为了测试绑定属性的输出==
    
    ```java
    @Component
    @ConfigurationProperties(prefix = "person") //默认是从全局配置文件加载,所以需要使用@PropertySource 来指配置文件,不指定的话从全局也行,爱咋咋地
    @PropertySource(value = "classpath:person.properties")  //从指定的配置文件加载数据
    public class Person {
    
        private String name;
        private Integer age;
        private boolean isChina;
        private Date birth; 
        Map<String, Object> map;
        private Dog dog;        //person 中的一个属性,用于测试配置文件中对象的输出
    
        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    ", isChina=" + isChina +
                    ", birth=" + birth +
                    ", map=" + map +
                    ", dog=" + dog +
                    '}';
        }
        
        提供 get/set 方法
    }
    ```

    ==Dog==

    ```java
    public class Dog {
    
        private String dogName;
        private Integer age;
    
        @Override
        public String toString() {
            return "Dog{" +
                    "dogName='" + dogName + '\'' +
                    ", age=" + age +
                    '}';
        }
        
        提供 get/set 方法
    }
    ```

    ==yml配置文件==

    ```properties
    person.name=李四
    person.age=12
    person.isChina=true
    person.birth=2019/3/14
    person.map.k1=v1
    person.map.k2=v2
    person.map.k3=v3
    person.dog.dogName=cc
    person.dog.age=12   
    ```
    
    ==单元测试类==

    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest //SpringBoot 的单元测试
    public class AutoconfigurationApplicationTests {
    
        @Autowired
        Person person;
    
        @Test
        public void contextLoads() {
            System.out.println(person);
        }
    
    }
    ```

#### 3.1.2 @Value 方式
1. 实现方式
    - 正常编写 JavaBean和配置文件,但是一定要注意 ==*JavaBean 和配置文件属性一一对应*==
    - 正常测试
    
    ```java
    @Value("${person.name}")
    private String name;
    @Value("#{4+14}")
    private Integer age;
    @Value("${person.isChina}")
    private boolean isChina;
    ```

### 3.2 日志
#### 3.1 springboot 日志使用简单演示    
1. SpringBoot 默认用的是 : SLF4J + logback
2. springboot 在 starter 中已经依赖了这两个日志框架,如果要使用其他矿框架要把这两个框架的依赖移除
3. springboot 默认日志级别是 : info

==*官方实例*==
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    //日志级别,由低到高 trace < debug < info < warn < error
    logger.trace("trace 日志");   //跟踪轨迹日志
    logger.debug("debug 日志");
    logger.info("info 日志");
    logger.warn("警告 日志");
    logger.error("错误 日志");
  }
}
```

```properties
#日志
# springboot->org.springframeword.boot->logging下关于日志的配置文件 base.xml
logging:
  level: info   #日志级别
  # 指定日志名称,如果不指定,就只会在控制台输出,如果指定,则会在项目路径下生成一个指定名称的日志文件
  #file: springboot.log

  # 指定日志文件的路径
  # 如果和 file 同时都指定的话,则会使 file 生效,一般指定 path,不指定 file
  # / : 在当前磁盘的根路径下,默认使用 spring.log 作为日志文件
  path: /usr/local/opt/var/springboot/
  #pattern:
    #console: d{yyyy-MM-dd} [%htread] %-5level %logger{50} - %msg%   #在控制台输出的日志的格式
    #file: d{yyyy-MM-dd} === [%htread] ===  %-5level === %logger{50} - %msg%   #指定文件中日志输出的格式
```
#### 1.2使用自己的配置文件 
1. 在类路径下(resource)放上每个日志框架自己的配置文件即可.然后Springboot 就会不适用默认配置的,文件名称==**要使用下面的名称**==
    - logback : ==*logback-spring.xml*==
    - 因为使用这个名称的话,日志框架就不直接加载日志配置项的,而是由 SpringBoot 加载,就可以使用 springProfile,可以指定某段配置只在某个环境生效,然后在默认配置文件中使用 spring.profile.active=xxx指定配置文件后,这里就可以启动对应的配置
    
        ```xml
        <springProfile name="staging">
            <!-- configuration to be enabled when the "staging" profile is active -->
        </springProfile>
        
        <springProfile name="dev, staging">
            <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
        </springProfile>
        
        <springProfile name="!production">
            <!-- configuration to be enabled when the "production" profile is not active -->
        </springProfile>
    ```


### 3.3 springboot 静态资源规则
1. springboot对静态资源有一些要求,在org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#addResourceHandlers

    ```java
    @Override
    // 指定资源映射 webjars
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
    		if (!this.resourceProperties.isAddMappings()) {
    			logger.debug("Default resource handling disabled");
    			return;
    		}
    		Integer cachePeriod = this.resourceProperties.getCachePeriod();
    		if (!registry.hasMappingForPattern("/webjars/**")) {
    			customizeResourceHandlerRegistration(registry
    					.addResourceHandler("/webjars/**")
    					.addResourceLocations("classpath:/META-INF/resources/webjars/")
    					.setCachePeriod(cachePeriod)); //设置缓存时间
    		}
    		String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    		if (!registry.hasMappingForPattern(staticPathPattern)) {
    			customizeResourceHandlerRegistration(
    					registry.addResourceHandler(staticPathPattern)
    							.addResourceLocations(
    										this.resourceProperties.getStaticLocations())
    								.setCachePeriod(cachePeriod));
    			}
    		}
    		
    
	@Bean
	//设置首页(欢迎页)映射  / /index.html
	public WelcomePageHandlerMapping welcomePageHandlerMapping(
			ResourceProperties resourceProperties) {
		return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
				this.mvcProperties.getStaticPathPattern());
	}	
	
	
	//配置喜欢的图标
	@Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
		public static class FaviconConfiguration {

			private final ResourceProperties resourceProperties;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
				//任何路径下的favicon.ico请求都会映射到静态文件夹
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));
				return mapping;
			}

			@Bean
			public ResourceHttpRequestHandler faviconRequestHandler() {
				ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
				requestHandler
						.setLocations(this.resourceProperties.getFaviconLocations());
				return requestHandler;
			}

		}
	
		
    ```
 
1. webjars 
    1. 由类 中的方法 : addResourceHandlers 指定
    - webjars/** : webjars 是说以静态 jar 包的形式引入资源. 
    - 去这个网站找资源,并可以拿到 mvn 依赖. [webJar](https://www.webjars.org/)
    - 所有 webjars下的所有请求都去 classpath:/META-INF/resources/webjars/ 找资源,就是说通过上边的网址,引入依赖后,去它的jar 包内的这个路径找资源.
    - 如通过上边网站下载的 jQuery后,通过 http://localhost:8080/webjars/jquery/3.3.1/jquery.js 可以访问到 jquery.js
2. 自己的资源怎么引用 (静态资源访问路径)
    - ==**ResourceProperties类**== 设置和静态资源相关的内容,比如缓存时间
    - 还指定了静态资源的访问路径
    - ==**静态资源文件夹**==保存在下边的文件夹中,就可以被访问到
    - 是 ==**spring.resources.static-location**==的配置,可以配置静态文件夹
    
    ```json
    classpath:/META-INF/resources/  : 类路径下的 /META-INF/resource    
    classpath:/resources/ : 类路径下的resources   reousrces/resources
    classpath:/static/  :
    classpath:/public/  : 
    "/" : 当前项目根路径
    && 以上都是静态资源文件夹.如果访问的资源没人处理,那么就去上边列出的路径下去找
    比如 :localhost:8080/abc,就回去上边的几个路径下去找 abc.
    ```

1. 首页映射
    - WebMvcAutoConfiguration类中的方法welcomePageHandlerMapping 设置了欢迎页(首页)的映射
    - 欢迎页 : 静态资源文件夹下的所有所有 index.html页面被 "/**"映射,就是说首页也放在静态资源文件夹下.
    - 比如: localhost:8080/
2. 喜欢的图片
    - 放在静态资源文件夹下,任意 favicon.ico 的URL 请求都会被映射到静态资源文件夹

#### 3.2.3 模板引擎   thymeleaf
##### 3.2.3.1 加入依赖以及文件路径
1. 加入依赖

    ```xml
    <properties>
    	<!--修改 themeleaf springboot 默认的版本-->
    	<thymeleaf.version>3.0.2.RELEASE</thymeleaf.version>
    	<thymeleaf-layout-dialect.version>2.1.1</thymeleaf-layout-dialect.version>
    </properties>
    <!--引入 thymeleaf 依赖-->
    <dependency>
    		<groupId>org.springframework.boot</groupId>
    		<artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    ```
2. 文件路径(通过查看自动配置原则)

    ```java
    //springframework-boot-autoconfigure 包下的 thymeleaf包下的 ThymeleafProperties类
    @ConfigurationProperties(prefix = "spring.thymeleaf")
    public class ThymeleafProperties {
    
        private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");
    
        private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");
        
        //这两行代码,可以看出只要把 HTML 页面放入到 classpath:/templates/ 路径下,thymeleaf 就可以自动渲染
        public static final String DEFAULT_PREFIX = "classpath:/templates/";
        public static final String DEFAULT_SUFFIX = ".html";
    }
    ```
    
1. 简单演示

    ==controller==
    ```java
    // 注意不要使用 @RestController,否则返回的都是文本
    @Controller
    public class ControllerExample {
    
        @RequestMapping("/success")
        public String thymeleafTest(Map<String, Object> map){
            //templates/success.html
            map.put("hello","你好");
            return "success";
        }
    }
    ```
    
    ==HTML页面==
    ```html
    <!DOCTYPE html>
    <!--引入 thymeleaf 命名空间-->
    <html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
        <h1>成功</h1>
        <!--th:text="" : 将 div 里的文本内容设置为 : -->
        <div th:text="${hello}"/>
    </body>
    </html>
    ```

##### 3.2.3.2 thymeleaf 语法  [官方文档](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax)
- 导入 thymeleaf 名称空间

    ```html
    <html lang="en"  xmlns:th="http://www.thymeleaf.org">
    ```
    - thymeleaf 的语法都是以 ==**th: **==开头
- 可以替换 HTML 原生属性的值
    
    ```html
    <!--替换 HTML 原生属性的值-->
    <div id="" class="" th:id="${id}" th:class="${class}" ></div>
    ```
- 其他语法 [参照官方文档第10部分](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#attribute-precedence)
    - Fragment inclusion 片段包含
        - th:insert
        - th:replace
    - Fragment iteration 遍历
        - th:each
    - Conditional evaluation 
        - th:if
        - th:switch
        - th:case
        - th:unless
    - Local variable definition 声明变量
        - th:object
        - th:with
        
    - General attribute modification 任意属性修改,支持前边添加和后边追加
        - th:attr
        - th:attrprepend
        - th:attrappend
    - Specific attribute modification 修改指定属性(支持 HTML 原生标签)
        - th:value
        - th:href
        - th:src
        - ...
    - Text (tag body modification) 修改标签体内容
        - th:text 支持转义特殊字符
        - th:utext 不支持转义特殊字符
    - Fragment specification 声明片段
        - th:fragment
    - Fragment removal 移除
        - th:remove  
- 表达式  [官方使用文档](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#standard-expression-syntax)

```properties
参照官方文档 第4部分
th:each="name: ${list}" :list 是要遍历的后台传过来的对象,name 标识设置的对象的名称
```



### 3.4 嵌入式Servlet 容器
- SpringBoot 默认使用的是嵌入式的 Servlet 容器(Tomcat).


#### 3.4.1 如何定制和修改 Servlet 容器的相关配置
##### 方法1:  修改和 server 相关的配置 (类: ServerProperties )

```properties
//通用的 servlet 容器设置  server.xxx
server.port=8081
server.context-path=/springboot

//tomcat相关的配置
server.tomcat.xxx
```
- EmbeddedServletContainerCustomizer
##### 方法2: ServerProperties实现了 : EmbeddedServletContainerCustomizer,所以两种方法是一样的
- EmbeddedServletContainerCustomizer : 嵌入式的 servlet 容器的定制器,来修改 servlet 容器的配置

```java
@Configuration
public class MyConfig {

    @Bean
    public EmbeddedServletContainerCustomizer addEmbeddedServletContainerCustomizer(){
        return new EmbeddedServletContainerCustomizer() {
            @Override
            public void customize(ConfigurableEmbeddedServletContainer container) {
                container.setPort(8083);
            }
        };
    }
}
```


#### 3.4.2 SpringBoot 能不能支持其他的Servlet容器


### 3.5 注册 Servlet,filter,listener
1. ServletRegistrationBean
2. FilterRegistrationBean
3. ServletListenerRegistrationBean

==servlet==

```java
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println("添加 servlet");
    }
}
```
==filter==

```java
public class MyFilter implements Filter {
    //filter初始化
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    //filter 执行
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("filter 执行了");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    //filter 销毁
    @Override
    public void destroy() {

    }
}
```
==listener==

```java
public class MyListener implements ServletContextListener {
    //当前web 项目启动
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("=======启动了=========");
    }

    //当前 web 项目销毁
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("=======关闭了=========");
    }
}
```

==myServerConfig==

```java
//加载三大组件
@Configuration
public class MyServerConfig {
    
    //加载 servlet
    @Bean
    public ServletRegistrationBean myServlet(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new MyServlet(), "/myServlet");
        return servletRegistrationBean;
    }

    //加载 filter
    @Bean
    public FilterRegistrationBean myFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new MyFilter());
        //要过滤的URL
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/success","/myServlet"));
        return filterRegistrationBean;
    }

    //加载 listener
    @Bean
    public ServletListenerRegistrationBean myListener(){
        ServletListenerRegistrationBean<MyListener> myListenerServletListenerRegistrationBean = new ServletListenerRegistrationBean<>(new MyListener());
        return myListenerServletListenerRegistrationBean;
    }

}
```

-----
## SpringBoot 特性详解
### 1. ==**版本仲裁**==

```xml
<!--
    spring-boot-starter-parent 又依赖了 spring-boot-dependencies ,
    spring-boot-dependencies 真正管理了 spring Boot 应用里边的所有依赖版本,
    所以开发者导入依赖的时候不需要写版本
-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.19.RELEASE</version>
    <relativePath/>
</parent>
```

### 2. ==**spring-boot-starter**==
- spring-boot-starter : 场景启动器
- 作用 : 帮开发者导入相对应模块所需的依赖,比如 spring-boot-starter-web,就到如了 web模块所需的所有依赖,比如 Tomcat,springmvc 的.
- Spring Boot 将所有的功能场景都抽取出来,做成一个个 starter(场景启动器),并且进行版本仲裁,只需要在 pom 内引入这些 starter ,那么相关场景的所有依赖就会被依赖进来.
- [Spring Boot 提供的所有 starter -- target 13.5](https://docs.spring.io/spring-boot/docs/1.5.19.RELEASE/reference/htmlsingle/#using-boot-starter)

### 3. 自动配置
#### 3.1. 了解自动配置原理前需要知道的
-  ==**导入组件**==
    ```java
        // [ 1 ] 主程序
        @SpringBootApplication
        public class AutoconfigurationApplication {}
        
        /**
         * [ 2 ] @SpringBootApplication //主程序   
         */
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @SpringBootConfiguration
        @EnableAutoConfiguration
        @ComponentScan(excludeFilters = {
        		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
        public @interface SpringBootApplication {
        
        }
        
        /**
         * [ 3 ] @EnableAutoConfiguration //开启自动配置
         *     @SpringBootApplication / @EnableAutoConfiguration
         */
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @AutoConfigurationPackage
        @Import(EnableAutoConfigurationImportSelector.class)    //开启导入组件选择器
        public @interface EnableAutoConfiguration { 
        
        }
        
        /**
         * [ 4 ] @AutoConfigurationPackage  //自动配置包,将主程序( @SpringBootApplication标注的类 )所在的包及下面所有子包里面的所有组件扫描到 Spring 容器.
         *      @SpringBootApplication / @EnableAutoConfiguration / @AutoConfigurationPackage
         */
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        // [ 5 ] @AutoConfigurationPackage 自动配置包.通过此注解实现导入组件,导入哪些组件由 AutoConfigurationPackages.Registrar.class 决定
        @Import(AutoConfigurationPackages.Registrar.class)  
        public @interface AutoConfigurationPackage {
        
        }
        
        /**
         *  类 : AutoConfigurationPackages
         *      导入主程序所在包及下面所有子包的组件
         */
        public abstract class AutoConfigurationPackages {
        /**
        	 * {@link ImportBeanDefinitionRegistrar} to store the base package from the importing
        	 * configuration.
        	 */
        	@Order(Ordered.HIGHEST_PRECEDENCE)
        	static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        
        		@Override
        		public void registerBeanDefinitions(AnnotationMetadata metadata,
        				BeanDefinitionRegistry registry) {
        			
        			//在此处打一个点断,然后启动程序,运行到这里后,右键这段代码,选择 [ Evaluate Expression ],然后点击 [ Evaluate ],可以看到导入的包名
        			[断点]register(registry, new PackageImport(metadata).getPackageName());
        		}
        
        		@Override
        		public Set<Object> determineImports(AnnotationMetadata metadata) {
        			return Collections.<Object>singleton(new PackageImport(metadata));
        		}
        	}
        }
        ```
        ==说明==
            - 根据注解 : ==**@SpringBootApplication**== 上的注解 : ==**@EnableAutoConfiguration**== 上的注解 : ==**@AutoConfigurationPackage**== 上的注解 : ==**@Import(AutoCOnfigurationPackages.Registra.class)**== 内的方法 : ==**Registra**== 可以导入主程序所在的包及下面所有子包的组件到 Spring 容器.
        
- ==**导入组件选择器**==
    
        ```java 
        // [ 1 ] @EnableAutoConfiguration
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @AutoConfigurationPackage
        @Import(EnableAutoConfigurationImportSelector.class)    //此注解提供的类 EnableAutoConfigurationImportSelector.提供了导入组件选择器的功能
        public @interface EnableAutoConfiguration {
    
    	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
    
        	/**
        	 * Exclude specific auto-configuration classes such that they will never be applied.
        	 * @return the classes to exclude
        	 */
        	Class<?>[] exclude() default {};
        
        	/**
        	 * Exclude specific auto-configuration class names such that they will never be
        	 * applied.
        	 * @return the class names to exclude
        	 * @since 1.3.0
        	 */
        	String[] excludeName() default {};
    
        }
        
        // [ 2 ] EnableAutoConfigurationImportSelector  //
        @Deprecated
        public class EnableAutoConfigurationImportSelector extends AutoConfigurationImportSelector {
    
            @Override
            protected boolean isEnabled(AnnotationMetadata metadata) {
                if (getClass().equals(EnableAutoConfigurationImportSelector.class)) {
                    return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
                }
                return true;
            }
        }
        
        //AutoConfigurationImportSelector
        public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware, ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    
    	   private static final String[] NO_IMPORTS = {};
    
    	   private static final Log logger = LogFactory.getLog(AutoConfigurationImportSelector.class);
    
    	   private ConfigurableListableBeanFactory beanFactory;
    
    	   private Environment environment;
    
    	   private ClassLoader beanClassLoader;
    
    	   private ResourceLoader resourceLoader;
    
    	   @Override
    	   public String[] selectImports(AnnotationMetadata annotationMetadata) {
    		  if (!isEnabled(annotationMetadata)) {
    			 return NO_IMPORTS;
    		  }
    		  try {
    			 AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
    			 AnnotationAttributes attributes = getAttributes(annotationMetadata);
    			 List<String> configurations = getCandidateConfigurations(annotationMetadata,
    					attributes);
    			configurations = removeDuplicates(configurations);
    			configurations = sort(configurations, autoConfigurationMetadata);
    			Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    			checkExcludedClasses(configurations, exclusions);
    			configurations.removeAll(exclusions);
    			configurations = filter(configurations, autoConfigurationMetadata);
    			fireAutoConfigurationImportEvents(configurations, exclusions);
    			return configurations.toArray(new String[configurations.size()]);
    		}
    		  catch (IOException ex) {
    			 throw new IllegalStateException(ex);
    		  }
    	   }
    	}
        ```
    ==说明==
        - 在注解 : ==*@SpringBootApplication*== 上的注解 : ==*@EnableAutoCnfiguration*== 上的注解 : ==*@Import(EnableAutoConfigurationImportSelector.class)*== ,==*类EnableAutoConfigurationImportSelector*==的父类 ==*AutoConfigurationImportSelector*== 中的 ==*selectImports*== 方法,这个方法会调用==*getCandidateConfigurations方法,*==  getCandidateConfigurations方法中会调用 ==*SpringFactoriesLoader.loadFactoryNames*== , ==*loadFactoryNames是个静态方法,*==方法中会导入 < META-INF/spring.factories > 文件中很多的自动配置类(以 AutoConfiguration结尾),这样就向容器中导入了这个场景所需要的所有组件,并配置好组件
    - 上步骤会将组建添加到容器中,用这些组建来做自动配置.

    
    
#### 3.2 自动配置原理
1. 在3.1中了解怎么导入组件后,下面开始自动配置原理的解析
2. 3.1中从 MEAT-INFO/spring.factoies 文件中导入了很多自动配置类.拿其中一个组件来说明自动配置原理

    - 使用 HttpEncodingAutoConfiguration 这个组建来说明自动配置原理: 
    - HttpEncodingAutoConfiguration,注意其中的注解. 
        
    ```java
    @Configuration  //说明此类是个配置类
    @EnableConfigurationProperties(HttpEncodingProperties.class)    //启动 ConeigurationProperties 功能
    @ConditionalOnWebApplication
    @ConditionalOnClass(CharacterEncodingFilter.class)
    @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
    public class HttpEncodingAutoConfiguration {
        
        private final HttpEncodingProperties properties;
        public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		      this.properties = properties;
		  }

	@Bean
	@ConditionalOnMissingBean(CharacterEncodingFilter.class)
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}

	@Bean
	public LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
		return new LocaleCharsetMappingsCustomizer(this.properties);
	}

	private static class LocaleCharsetMappingsCustomizer
			implements EmbeddedServletContainerCustomizer, Ordered {

		private final HttpEncodingProperties properties;

		LocaleCharsetMappingsCustomizer(HttpEncodingProperties properties) {
			this.properties = properties;
		}

		@Override
		public void customize(ConfigurableEmbeddedServletContainer container) {
			if (this.properties.getMapping() != null) {
				container.setLocaleCharsetMappings(this.properties.getMapping());
			}
		}

		@Override
		public int getOrder() {
			return 0;
		}

	}

    }
    ```
    - 第一个注解 : @Configuration  //说明此类是个配置类
    - 第二个注解 : @EnableConfigurationProperties(HttpEncodingProperties.class) 中的类 : 
        - HttpEncodingProperties.class
            
        ```java
        @ConfigurationProperties(prefix = "spring.http.encoding")   //从配置文件中获取指定的值和属性进行绑定
        public class HttpEncodingProperties {
            public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
        
        	/**
        	 * Charset of HTTP requests and responses. Added to the "Content-Type" header if not
        	 * set explicitly.
        	 */
        	 private Charset charset = DEFAULT_CHARSET;
        
        	/**
        	 * Force the encoding to the configured charset on HTTP requests and responses.
        	 */
        	 private Boolean force;
        
        	/**
        	 * Force the encoding to the configured charset on HTTP requests. Defaults to true
        	 * when "force" has not been specified.
        	 */
        	 private Boolean forceRequest;
        
        	/**
        	 * Force the encoding to the configured charset on HTTP responses.
        	 */
        	 private Boolean forceResponse;
        
        	/**
        	 * Locale to Encoding mapping.
        	 */
        	 private Map<Locale, Charset> mapping;
        }
        ```
        - HttpEncodingProperties.class 上的注解 @ConfigurationProperties(prefix = "spring.http.encoding")会从配置文件中获取指定的值和属性进行绑定
        - 例如在配置文件中可是指定以下配置:
            
            ```properties
            spring:
              http:
                encoding:
                  charset: UTF-8
                  force: false
                  mapping: UTF-8
            ```
        - 可以发现配置文件中可以配置的配置项是和这个HttpEncodingProperties中的属相相对应
        - 可以发现在配置文件中能配置的属性都是在 xxxx.Properties 类中封装着
        - 所以配置文件中可以配置什么配置项就可以参考某个功能对应的这个属性类 
    - 第三个注解: @ConditionalOnWebApplication 
        - spring 底层有个@Condition 注解,根据不同条件,如果满足指定的条件,整个配置类里边的条件就会生效
        - 所以这个注解是判断当前类是否是 web 应用,如果是,当前配置类生效
    - 第四个注解 : @ConditionalOnClass(CharacterEncodingFilter.class)
        - 判断当前项目有没有这个类 : CharacterEncodingFilter,如果有当前配置类生效
    - 第五个注解 :  @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
        - 判断配置文件是是否存在某个配置 : spring.http.encoding.enabled,
        - matchIfMissing=true : 标识如果配置文件中不配置 : spring.http.encoding.enabled ,也是默认有效的(当前配置类生效)
    - 所以这个组件,经过几个判断,配置类生效,也就是上边分析的配置类 : HttpEncodingAutoConfiguration 生效
    - HttpEncodingAutoConfiguration 类生效后,里边有 @Bean 方法 : characterEncodingFilter
        
        ```java
        	@Bean  //给容器中添加一个组件(返回值就是添加的组件)
        	@ConditionalOnMissingBean(CharacterEncodingFilter.class)
        	public CharacterEncodingFilter characterEncodingFilter() {
        		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        		filter.setEncoding(this.properties.getCharset().name());
        		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
        		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
        		return filter;
        	}
        ```
    - characterEncodingFilter 经过ConditionalOnMissingBean的判断后,如果成立,就会生效,向容器中添加一个组件,但是这个组建的某些事是从==*properties*==类中获取的.properties对象是 ==*HttpEncodingProperties*==.
    - HttpEncodingProperties 这个类,已经通过 @EnableConfigurationProperties 注解和配置文件进行了绑定.
    - 总结: ==**根据条件判断这个 xxxAutoConfigurationProperties 配置类是否成立,如果成立的话,就会想容器中添加组件**==
- 所以,SpringBoot 可以配什么配置,最终是来源于这些功能的 xxxxProperties类的属性 
    
------
### springboot 对 SpringMVC 的自动配置
SpringBoot 自动配置了 SpringMVC
下边是对 Springmvcde 默认配置:

- 视图解析器 : 根据方法的返回值得到视图对象,视图对象决定如何渲染
    - ContentNegotiatingViewResolver 和 BeanNameViewResolver
    - ContentNegotiatingViewResolver : 组合所有的视图解析器
- Support for serving static resources, including support for WebJars (see below).
    - 静态资源文件夹路径和 webjars
- Automatic registration of Converter, GenericConverter, Formatter beans.
    - 自动注册了 Converter, GenericConverter, Formatter
    - Converter: 转换器
    - Formatter: 格式化器,
        
    ```java
    @Bean
    @ConditionalOnProperty(prefix = "spring.mvc", name = "date-format") //
    public Formatter<Date> dateFormatter() {
    	return new DateFormatter(this.mvcProperties.getDateFormat());
    }	
    ```
- Support for HttpMessageConverters (see below).
    - HttpMessageConverters: 消息转换器,SpringMVC 中用来转换 HTTP 请求和响应的
    - messageConverters是从容器中确定的.获取所有的messageConverters
    - 自己给容器中添加HttpMessageConverter,只需要将自己的组件注册在容器中,比如 @Bean
- Automatic registration of MessageCodesResolver (see below).
    - MessageCodesResolver : 消息解析
- Static index.html support.
- Custom Favicon support (see below).
- Automatic use of a ConfigurableWebBindingInitializer bean (see below).
    - 我们可以自己配置一个 ConfigurableWebBindingInitializer 
If you want to keep Spring Boot MVC features, and you just want to add additional MVC configuration (interceptors, formatters, view controllers etc.) you can add your own @Configuration class of type WebMvcConfigurerAdapter, but without @EnableWebMvc. If you wish to provide custom instances of RequestMappingHandlerMapping, RequestMappingHandlerAdapter or ExceptionHandlerExceptionResolver you can declare a WebMvcRegistrationsAdapter instance providing such components.

If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc.    


    
-----
## 注解
### SpringBoot 注解
1. **==@SpringBootApplication==**
    - 标注在类上,说明这个类是 SpringBoot 的主配置类. 
    - SpringBoot就会运行这个类的 main 方法来启动 SpringBoot 应用.

1. ==*@ResponseBody*==
    - 标注在类上或者方法上,一般标注在类上,说明这个类中的所有方法返回的数据直接写给浏览器
    - 如果返回的是对象,还能转为 json
2. ==*@Controller*==
    - 标注在类上,说明这个类是 controller
3. ==**@RestController**==
    - @Controller + @ResponseBody = @RestController
4. ==*@ConfigurationProperties*==
    - 标注在类上,就是要和配置文件映射属性的类,比如要给 person 注入配置文件中的值,就在 Person 上标志此注解
    - 默认从全局配置文件中获取值.
    - 告诉 springboot 将类中的所有属性都是配置文件中相关的配置进行绑定,将配置文件中的每一个属性的值映射到这个类中,
    - 要使用这个注解必须要和 ==**@Component**== 一起使用 ,因为容器中的组件才能使用容器提供的功能.
    - 如果专门编写了一个 JavaBean 来和配置文件进行映射,那么就直接使用 @ConfigurationProperties 注解.
    - 属性 : prefix : 配置文件中哪个标签下的属性进行绑定.
        
        ```java
        @Component  //将此类交给容器管理
        @ConfigurationProperties(prefix="person")
        ```
            
1. @Autowired

2. 单元测试
    - SpringBoot 的单元测试,可以在测试期间很方便的使用组件的功能
        
        ```java
        @RunWith(SpringRunner.class)    
        @SpringBootTest //SpringBoot 的单元测试
        ```
3. ==*@Value("${ymlTarGet.property}")*==
    - 作用在类的属性上,为属性注入值
    - @Value("${对象.属性}")
    - Spring 的注解


1. ==*@Validated : JSR 303数据校验*==
    - 标注在指定的要和配置文件映射属性的 JavaBean,比如 person 要映射值,就标注在Person类上.使用 ==*@ConfigurationProperties*== 注入数据时,可以进行数据校验
    - 支持 SPEL 语法
    - 当只是需要获取以下值的时候,可以使用 @Value
    - ==*@Email*== : 作用在类的属性上,配合 @Validated 使用,属性必须时 Email 格式

1. ==*@PropertySource*==
    - 标注在指定的要和配置文件映射属性的 JavaBean,比如 person 要映射值,就标注在Person类上.测试后发现 properties 配置文件可以实现,但是 yml不行,不知道是不是写的代码的问题,妈了个鸡的!
    - 加载指定的配置文件.
    - 比如使用 ==*@ConfigurationProperties*== 获取配置文件值的时候,本来默认从默认配置文件取,现在可以从指定的配置文件中取
    - 属性 : value : 要加载的配置文件的位置 value="classpath:xxxx.yml/properties"

1. @ImportResource
    - 标注在配置类上.可以是主配置类(就是 springboot 自动生成的主程序)上
    - 导入 Spring 的配置文件,让配置文件里边的内容生效.给容器添加组件
    - 属性 : locations : spring配置文件路径  locations:xxxx.xml
    - Springboot 内没有 Spring 的配置文件(xml形式的配置文件),想让这种配置文件加载进来生效,就要使用 @ImportResource 标注在一个配置类上(主程序上)
    - 但是其实这种方式不推荐,SpringBoot 推荐全注解的方式
    
    ```java
    @ImportResource(locations = {"classpath:xxx.xml"})
    @SpringBootApplication
    public class AutoconfigurationApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(AutoconfigurationApplication.class, args);
        }
    }
    ```
  
1. ==*@Configuration*==
    - 作用在类上. 指明当前类是个配置类,替代 Spring 的配置文件,就是不用再写配置文件来添加组件了,可以写在主配置类(就是 springboot 自动生成的主程序)上也可以自己写个配置类
    - 一般和 @Bean 配合使用写完要添加的类之后,在这个配置类里边创建方法,这个方法用 ==@Bean== 标注并将返回值返回给容器,以此来添加组件
    - 和 @ImportResource, SpringBoot 更推荐这种方式来给容器添加组件,用来替代 @ImportResource
        
        ==要添加的组件==
        ```java
        /**
         * 要添加的组件
         */
        public class ComponentAdded {
        }
        ```
        ==配置类,用于向容器添加组件==
        
        ```java
        @Configuration  //告诉 SpringBoot 这是一个配置类,配合@Bean 使用来给容器中添加组件
        public class AddComponent {
        
            @Bean
            public ComponentAdded componentAdded(){
                System.out.println("向 Springboot 添加组件 ComponentAdded 了!!");
                return new ComponentAdded();
            }
        }
        ```
        
        ==单元测试==
        
        ```java
        @RunWith(SpringRunner.class)
        @SpringBootTest //SpringBoot 的单元测试
        public class AutoconfigurationApplicationTests {
        
            @Autowired
            ApplicationContext ioc;
            
            @Test
            public void checkAddComponent(){
                System.out.println(ioc.containsBean("componentAdded"));
            }
        }
        ```
     
1. @Bean 
    - 作用在方法上,相当于 xml 配置文件中的 bean 标签,将方法的返回值添加到容器中,也就是说返回值(类)就是要添加的组件,Bean 的 ID 默认是此方法的==方法名==
    - 就是说要添加哪个类作为组件,就返回哪个类的类型

1. EnableConfigurationProperties(HttpEncodingProperties.class)
    - 启动 ConfigurationProperties 功能.
    - 参数 类 的作用是
2. @ConditionalOnWebApplication
    - 判断是不是 web 应用
3. @ConditionalOnClass(CharacterEncodingFilter.class)
    - 判断有没有这个类(参数内的类)
4. @ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)
    - 判断配置文件中有没有前缀是 spring.http.encoding.enabled 的配置项
    - matchIfMissing是指如果没有,判断也成立
5. @ConditionalOnMissingBean(CharacterEncodingFilter.class)
    - 判断容器中是否没有这个类,没有的话成立


-----  
## 配置文件
### 1. 配置文件加载位置
1. 配置文件的加载顺序
    - 默认使用 application.yml/properties 作为配置文件,但是这些配置文件的位置可以改变
    - 以下==*优先级从高到低*==
    - 当前项目路径下的 config 文件夹下(就是文件结构中直接在项目名上右键创建的 config 下)
    - 当前项目路径下 (直接放在项目下)
    - 类路径的 config 下  (resource 文件夹下的 config 下)
    - 类路径下  (resource 下)
    - ==*以上优先级从高到底,高优先的会覆盖低优先的相同的配置项*==
    - ==*互补配置: 如果高优先和低优先的配置文件有相同的配置项,则会覆盖相同的配置项,如果低优先有一个配置项,而高优先级没有这个配置项,则会使用低优先级的此配置项*==
2. spring.config.location : 

    - 在项目打包好以后,可以使用命令号参数的形式在启动项目的时候,可以使用指定位置的配置文件
    - 指定位置的配置文件会和默认加载的这些配置文件形成互补配置

    ```shell
    java -jar xxx.jar --spring.config.location=配置文件路径(磁盘上的路径)
    ```

#### 1. 外部配置的加载顺序







### 2. 配置文件命名

- ==*配置文件名称是固定的 : application.properties 或者 application.yml*==

### 3. ==yml类型配置文件写法==
#### 1. 基本语法:
- k: v  : 表示一对键值对,==冒号和 v 之间的空格必须有==
- 以空格的缩进来控制层级关系,就是说只要左边对其了,就是同一层级的.    
    ```properties
    server:
        port: 8088
        context-parameters:
        address: 
    ```

- 属性和值大小写敏感
### 2. 值的写法
1. 字面值直接写
2. 字符串默认不加单引号或者双引号
3. 对象和 map  
    - k: v,例如有个对象 : user,user内有name,age,addr,只要注意缩进层级关系就行.
    - map 和对象写法一样
   
    ```properties
    #正常写法
    user:
        name: zhangsan
        age: 20
        addr: beijing
    
    #行内写法
    user: {name: zhangsan,age: 18,add: beijing}
    ```

4. 数组(List,Set)
    - 用 - 值 (-空格值)的方式表示数组中的一个元素

    ```properties
    #正常写法
    arr:
        - 1
        - 2
        - 3

    #行内写法
    arr: [1,2,3]
    ```


### 3. 多环境支持(主配置文件)
#### 2.1 spring.profiles.active 方式
##### 2.1.1 使用 properties 配置文件
- 要使用的配置文件命名 : ==*application-{profille}.yml/properties*== , {profille}用于指定配置名称,如取值为 : dev 或 pro
- 使用方法: 分别编写要是多的多环境配置的配置文件,如: dev 和 prof,然后默认配置文件正常配置,在默认配置文件中使用 : ==*spring.profiles.active={profile}*== 来激活对应的配置
    
    ==*application-dev.properties*==
    ```properties
    server.port=8082
    ```
      
    ==*application-prod.properties*==
    ```properties
    server.port=8083
    ```

   ==*application.properties*==
    ```properties
    // 默认配置,如果下边两个都注释点,或者不指定,则使用默认配置
    server.port=8081
    // 指定启动 application-dev.properties 配置文件的配置
    //spring.profiles.active=dev
    
    // 指定启动application-prod.properties 配置文件的配置
    spring.profiles.active=prod
    ```
    
##### 2.2.2 使用 yml 配置文件
- 和使用 properties 配置文件一样,只不过后缀不同.
- 除了上述方式外,还有一种==*文档块的方式*==
    - 不用编写其他的配置文件,在默认配置文件中使用 ==*spring.profiles指定配置名称*==,然后使用 ==*--- (三个横线)*==隔开各个配置就行,然后在默认的配置使用 ==*spring.profiles.active={配置名称}*==

   ==*application.yml*==
   
    ```properties
    # document 1/3

    #默认配置
    server:
        port: 8088

    #指定要是用的配置的文档块,如果不指定,则使用默认配置
    spring:
        profiles:
        active: dev
        
    #--- 三个横线,用于分割文档块,这里因为编辑的原因,注释掉了,在真正写的时候,需要打开
    # document 2/3

    #指定配置名称
    spring:
        profiles: dev

    #配置内容
    server:
        port: 8085


   #--- 三个横线,用于分割文档块,这里因为编辑的原因,注释掉了,在真正写的时候,需要打开
    # document 3/3

    #指定配置名称
    spring:
        profiles: prod

    #配置内容
    server:
        port: 8086
    
    ``` 
    
#### 2.2 命令行方式

#### 2.3 虚拟机参数


### 4. 配置文件占位符 ${ }
1. 随机数
    
    ```properties
    ${random.uuid}  //随机的 UUID
    ${random.int}   //随机的整数
    .
    .
    .
    ```
1. 使用占位符获取之前配置过的值,如果没有指定的话,可以使用==*冒号*==设置默认值
    

    ```properties
    person.name=李四${random.uuid}    
    person.age=${random.int}
    person.isChina=false
    person.birth=2019/3/14
    person.map.k1=${person.not:12}  //
    person.map.k2=v2
    person.map.k3=v3
    person.dog.dogName=${person.name=李四}cc  //第一行设置过person.name,所以这里可以使用${ } 的形式使用这个值
    person.dog.age=${random.int}    //随机数
    ```

-----
## 相关配置:
[官方配置项](https://docs.spring.io/spring-boot/docs/1.5.19.RELEASE/reference/htmlsingle/#common-application-properties)

```properties
#配置静态文件夹路径
spring.resources.static-locations=classpath:/hello/,classpath:/word/

#设置项目全局的访问路径,就是最前面的 URL ,例如:http://localhost:8080/springboot/success.html
server.context-path=/springboot

#格式化日期
spring.mvc.date-format=yyyy-MM-dd

#禁用缓存
spring.thymeleaf.cache=false
```

### 1. ==**日志**==
[官方关于日志的文档](https://docs.spring.io/spring-boot/docs/1.5.19.RELEASE/reference/htmlsingle/#boot-features-logging)

#### 1.1 简单演示



-----
## 目录结构
- resources
    - 文件夹 : static : 保存所有的静态资源,比如 css, js, image 等
    - 文件夹 : templates : 保存所有模板页面,(SpringBoot 默认 jar 包使用嵌入式 Tomcat,不支持 JSP),可以使用模板引擎(freeMarker, thymeleaf)
    - 文件 : application.properties : springboot 的配置文件

-----
## 使用技巧
1. 使用命令行执行程序
    ```json
    spring-boot-maven-plugin : 可以将程序打成 jar 包,然后可以使用 java -jar xxxxx.jar 执行这个 jar 包
    ```

1. 程序内导入 IOC,然后可以使用 ioc 的方法
    
    ```java
    @AutoWired
    ApplicationContext ioc;
    ```
2. 在命令行启动,并制定要使用的配置文件(多环境加载, ==*spring.profiles.active*== 方式)
        
    ```shell
    
    ```

3. 在项目打包好以后,可以使用命令号参数的形式在启动项目的时候,可以使用指定位置的配置文件
    - 指定位置的配置文件会和默认加载的这些配置文件形成互补配置

    ```shell
    java -jar xxx.jar --spring.config.location=配置文件路径(磁盘上的路径)
    ```


## 如何修改 SpringBoot 的默认配置
1. Springboot 在自动配置的时候,都会先检查容器中是否有此组件,如果没有就自动配置,但是如果有,就用用户自己的.如果有些组件可以有多个,比如视图解析器,Springboot 会将用户配置的和自己默认的组合起来.
2. 在 Springboot 中有很多 xxxCustomizer 帮助我们进行定制配置

    
    
## 精髓
1. SpringBoot启动的时候会加载大量的自动配置类
2. 我们看我们需要的功能有没有在 springboot 中写好的自动配置类
3. 再看这个自动配置类中添加了那些组件(只要有,就可以不用配置了,如果没有,就需要自己写配置类)
4. 容器中的自动配置类里的组件,会从 xxxProperties 类中获取属性,就可以在配置文件中指定这些属性