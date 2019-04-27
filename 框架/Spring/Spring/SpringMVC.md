# SpringMVC
## 1. 组件以及运行原理
### 1. 组件
```properties
DispatcherServlet : 前端控制器
    作用 : 接收请求.响应结果.请求转发控制中心
HandlerMapping : 处理器映射器
    作用 : 通过请求 URL 查找处理器 Handler(Controller)
Handler : 处理器
    作用 : 处理请求.就是自己编写的 Controller
HandlerAdapter : 处理器适配器
    作用 : 按照 HandlerAdapter 的规则去执行处理器 Handler(Controller)
ViewResolver : 视图解析器
    作用 : 进行视图解析
View : 视图
    作用 : View 是一个接口,适配不同的 View 类型,比如 JSP,Freemarker 等
```
### 2. 运行原理

```properties
客户端向前端控制发送请求
前端控制器请求处理器映射器查找 Handler
处理器映射器向前端控制器返回 handler
前端控制器调用处理器适配器去调用 handler
处理器适配器执行 Handler
Handler 执行完毕后向处理器适配器返回 ModelAndView
处理器适配器向前端控制器返回 ModelAndView
前端控制器调用视图解析器进行视图解析
视图解析器向前端控制返回 view
前端控制器进行视图渲染
前端控制器向用户响应结果.
```


## 注解

```properties
@Controller : 表明这个类是controller
@RequestMapping : 标注在类上或者方法上. URL 映射,请求 URL 中添加参数 : @RequestMapping("/test/{id}")
    参数:
        value : 当只有一个 URL 时可有可没有,value={"/url1", "/url2", "/url3"} : 可以有多个参数
        method : 指定请求类型
            method=RequestMethod.GET : get请求
            method=RequestMethod.POST : POST 请求
            method={}
        consumers : 指定媒体类型.HTTP 头的 Content-Type 的值要和 consumers 指定的一样才匹配.
        produces : 对应 HTTP 请求的 Accept 字段.只有相同才能匹配.通常浏览器都会讲 Accept 设置为 *.*,所以访问浏览器/user/1,浏览器总是会返回 1 对应的数据,
            此属性可设置 response 的 Content-Type,就是编码和格式,比如: produces=MediaType.APPLICATION_JSON_VALUE+";charset=utf-8" : 返回 utf8的 json,中文什么的也不会乱码
    

@PathVariable : 标注在方法参数列表中,用于接收请求 URL 中的参数,和 @RequestMapping 一起使用
    使用方法: 
        url : localhost:8080/test/4
        @RequestMapping("/test/{id}")
        public String run(@PathVariable() Integer id) : 默认将请求URL 中的请求参数 ID 绑定到参数 id 中(名称相同)
        public String run(@PathVariable("id") Integer id1) 默认将请求URL 中的请求参数 ID 绑定到参数 id1 中(名称不相同)

@RequestParam : 标注在方法参数列表中.请求参数绑定,就是请求 URL 中的参数和方法参数列表中的参数进行绑定
    属性 : 
        value : 请求参数名.就是请求 URL 中有参数传入
        required : 这个参数是否必须.false:不必须,true:必须.如果为 true,而请求又没有这个参数,则会报错
        defaultValue : 默认值.标识请求中没有同名参数时的默认值.
         使用方法 : 
            url : localhost:8080/test?id=4
            @RequestMapping("/test")
            public String run(@RequestParam(value="id", required=true, defaultValue="123")){}

@RequestBody : 作用在方法上.用于接收 POST 请求的JSON 数据,转成 java 对象
@ResponseBody : 标注在方法上,返回文本给浏览器
```