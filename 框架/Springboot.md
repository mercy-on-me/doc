# SpringBoot
## 1. 原理
### 1. 自动配置原理

## 2. MVC
### 1. 集成 MVC
- 加入依赖

```xml
<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 2. 目录结构

1. 静态资源

    ```properties
    classpath:/META-INF/resources/  : 类路径下的 /META-INF/resource    
    classpath:/resources/ : 类路径下的resources   reousrces/resources
    classpath:/static/  :
    classpath:/public/  : 
    "/" : 当前项目根路径
    && 以上都是静态资源文件夹.如果访问的资源没人处理,那么就去上边列出的路径下去找
    比如 :localhost:8080/abc,就回去上边的几个路径下去找 abc.
    ```
1. 模板文件

    ```properties
    classpath:templates : 类路径下的 templates 文件夹存放模板文件
    ```
    
### 3. 配置 MVC 全局特性
- ==*WebMvcConfiguer*== : 此接口用于全局制定 springboot 的 MVC 特性
- 开发者只要实现 WebMvcConfiguer 接口就可以配置全局的 MVC 特性,并标注为配置类

==实现 WebMvcConfiguer==

```java
@Configuration
public class MvcConf implements WebMvcConfigurer {
    //格式化
    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    //添加 处理器拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //增加一个处理器拦截器,拦截 /hello 请求. 此时 /hell 就会被拦截,并走具体的拦截器方法
        registry.addInterceptor(new MvcConfig()).addPathPatterns("/hello");
    }
    
     //跨域访问设置
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("") //允许跨域的请求路径
        .allowedOrigins("") //  允许跨域的的网址        对应的是类 : CorsRegistration 中的方法
        .allowedMethods("POST", "GET")  //允许的跨域的请求的方式
        ;
    }
    
      // URL 到视图的映射,比如对指定的一个 URL 要转到指定的模板,则可用这个方法
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 为所有 index.html 请求设置模板 index.btl
        registry.addViewController("/index.html").setViewName("/index.btl");
        // 将所有以 .do 结尾的请求都重定向到 /index/btl
        registry.addRedirectViewController("/**/*.do", "/index.btl");
    }
}
```
==具体的处理器拦截器实现==

```java
public class MyInterceptor implements HandlerInterceptor {

    // Controller 方法运行前调用此方法,
    // 返回值：
    //      true表示继续流程（如调用下一个拦截器或处理器）；
    //      alse表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应；
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("Controller 方法为开始运行");
        return true;
    }

    // Controller 方法结束后,渲染开始前,调用此方法
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("Controller 方法运行结束,未开始渲染");
    }

    // Controller 方法结束,渲染结束
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("Controller 方法运行完成,渲染完成");
    }
}
```

## 3. 视图技术
- SpringBoot 内置支持如下:
    - FreeMarker
    - Groovy
    - Thymeleaf
    - Mustache

### 3.1 FreeMarker
- 实现方式
    - 引入依赖
    - 模板文件放在 : ==*resources/templates/*== 下
    - java中向 modelAndView 指定模板名称,模板引擎会自动加上 /ftl 后缀去寻找匹配的模板.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

==**Controller**==

```java

@Controller
public class MyController {

    @RequestMapping("/show.html")
    public ModelAndView showUser(Model model){
        ModelAndView view = new ModelAndView();
        User user = new User("zhangsan", 12);
        // 添加对象
        view.addObject("user", user);
        // 设置视图名称,freemarker 会自动加上后缀 .ftl 来匹配视图
        view.setViewName("test");
        return view;
    }
}
```

==**ftl 视图**==

```xml
<body>
<!--获取对象属性-->
${user.name}
<br>
${user.age}
</body>
```



### 3.2 Thymeleaf
- 引入依赖

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



-----
## 4. 持久化操作
### 1. 使用 MySQL
- 引入依赖




-----
## 注解

```properties
@Controller : 标注在类上.说明此类事 controller
@ResponseBody : 标注在方法上,说明此方法返回的数据直接写到浏览器,还可以返回 json,如果是对象也会被转为 json,如果不加则返回视图.
@RestController : @Controller + @ResponseBody
@RequestMapping : 标注在类上或者方法上. URL 映射
    参数:
        value : 当只有一个 URL 时可有可没有,value={"/url1", "/url2", "/url3"} : 可以有多个参数
        method : 指定请求类型
            method=RequestMethod.GET : get请求
            method=RequestMethod.POST : POST 请求
            method={}
    请求 URL 中添加参数 : @RequestMapping("/test/{id}")
    请求 URL 可以使用通配符 : * 可以匹配任意字符. ** 可以匹配任意路径. ? 可以用来匹配单个字符
        /user/*.html : 匹配/user/1.html /user/2.html 等
        /**/1.html : 匹配/1.html  /user/1.html /user/add/1.html
        /user/?.html : 匹配 /user/1.html ,不匹配/user/11.html
    如果一个请求有多个@RequestMapping 可以匹配.有通配符的优先级低于没有通配符的优先级. 比如: /user/?.html 优先级低于 /user/add.html

@PathVariable : 标注在方法参数列表中,用于接收请求 URL 中的参数,和 @RequestMapping 一起使用
    使用方法: 
        public String run(@PathVariable() Integer id) : 默认将请求URL 中的请求参数 ID 绑定到参数 id 中(名称相同)
        public String run(@PathVariable("id") Integer id1) 默认将请求URL 中的请求参数 ID 绑定到参数 id1 中(名称不相同)

@RequestParam : 标注在方法参数列表中.请求参数绑定,就是请求 URL 中的参数和方法参数列表中的参数进行绑定
    属性 : 
        value : 请求参数名.就是请求 URL 中有参数传入
        required : 这个参数是否必须.false:不必须,true:必须.如果为 true,而请求又没有这个参数,则会报错
        defaultValue : 默认值.标识请求中没有同名参数时的默认值.
    使用方法 : public String run(@RequestParam(value="id", required=true, defaultValue="123")){}

@ModelAttribute : 
    标注在 Controller 内的某个方法上 : 不需要指定 URL,在这个 Controller 的其他方法被调用前,都会先调用被这个注解标注的方法.并可以使用注解 @PathVariable 获取 URL 上的参数.可以在这个方法中向 model 中添加一个参数,比如关于客户的操作,一般都要获取客户信息,这里个方法就可以获取客户信息,然后添加到 model 中,如果只添加一个对象.则可以直接 return userService.getUser(id),不用 model.addAttribute.
    标注在 Controller 内的某个方法的参数列表中 : 运用在参数上，会将客户端传递过来的参数按名称注入到指定对象中，并且会将这个对象自动加入ModelMap中，便于View层使用；

@Valid : 标注在方法参数上,可以进行校验

@Validated : 标注在Controller方法上,只要URL 被调用就触发一次检验
```

----
## 5. 日志
- 日志框架的原则 : 




-----
## 配置


```properties
#使用指定配置
spring.profiles.active=xxx : 
#配置静态文件夹路径
spring.resources.static-locations=classpath:/hello/,classpath:/word/

#设置项目全局的访问路径,就是最前面的 URL ,例如:http://localhost:8080/springboot/success.html
server.context-path=/springboot

#格式化日期
spring.mvc.date-format=yyyy-MM-dd

#禁用缓存
spring.thymeleaf.cache=false
```
-----
## 视图操作
### 取值
==**map**==

```xml
<!--freemarker-->
${map["key"]}
```





----
## 方法

```java
model.addAttribute("hello",str);
System.out.println(model.containsAttribute("hello"));   //判断 model 内是否存在变量
model.addAllAttributes(map);    //添加多个变量,如已经存在,则覆盖
model.mergeAttributes(map);     //添加多个变量,如已经存在,则忽略
```

-----

## 开发技巧
1. URL : 如果期望返回视图,URL 以 .html结尾.如果期望返回 JSON,URL 就以 .json 结尾

