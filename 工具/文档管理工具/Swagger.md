# Swagger
- Swagger作用 : 
    - 是一个文档管理工具(API)
    - 代替 postman 进行接口测试.
    - 根据接口文档,生成对应语言的代码
    - 根据代码.生成文档
- 使用场景 : 
    - 前后端分离.
    - 开发的代码需要被第三方使用.

    
    
##  2. swagger 优化
- 选择性显示接口

```java
select().apis(RequestHandlerSelectors.basePackage(XXXController.class.getPackage().getNmae()    //只扫描XXXController.下的接口,(包路径)
path(PathSelectors.ant("/xxx/*"));  //根据请求的 URL 选择要显示的接口.
```

- 分组
    - 使用 @Bean 来进行分组

- 详细注释说明
    - 使用 ==*@ApiOperation*== 进行注解
    
```java
// 方法注释
@ApiOperation(value ="相当于要展示的方法名", notes = "具体的方法描述")

// 属性注释
@ApiModelProperty(value = "简单说明",  )


```
- 中文显示
    - 重写 swagger-ui.html
    - 引入

    
    
## 3. Swagger demo
==**使用到的注解**==

```properties
作用在类上 :
    @EnableSwagger2 : 开启 Swagger2 功能.一般作用在 Swagger 的配置类上. springboot 项目则作用在主配置类上.
    @Api : 作用在类上,说明此类的作用.
    
作用在方法上 : 
    @ApiOperation : 对方法进行说明.
    @ApiImplicicParams : 对方法的参数列表进行说明.一般内部使用 @ApiImplIcitParam 来分别说明没一个参数.
    @ApiImplicitParam : 针对没一个参数进行说明.有一下几个属性 :
         paramType : 参数放在哪个位置. 属性值:
            query : 请求参数放在请求地址中. xx/xx?参数=1. 使用@RequestParam获取参数
            path : 请求参数放在 URL 中. xx/xx/{参数}. 使用 @PathVariable 获取参数
            header : 请求参数放置于 Request Header,使用 @RequestHeader 获取参数
            body,form 两个不常用
        name : 参数名
        value : 对参数进行说明,说明参数的意思
        required : 参数是否必输
        dataType : 参数的类型
        defaultValue : 参数的默认值  
        
作用在 POJO 上 :
    @ApiModel : POJO 模型
    @ApiModelProperty : 作用在 POJO 属性上.
        value : 属性描述
        required : 是否必输       
```

==**演示 ( 基于 Spring boot )**==
==*配置类*==

```java
@EnableSwagger2 //启用 Swagger2
@Configuration
public class SwaggerConfiguration {

    /**
     * 创建 API 应用
     * apiInfo() : 增加 API 相关信息
     * @return
     */
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SPRING_WEB)
                .apiInfo(apiInfo())
                .select()   // 通过select()函数返回一个ApiSelectorBuilder实例,用来控制哪些接口暴露给Swagger来展现，
                .apis(RequestHandlerSelectors.basePackage("com.example.swagger.controller"))//本例采用指定扫描的包路径来定义指定要建立API的目录。
                .paths(PathSelectors.any())
                .build();

    }

    //创建 API 基本信息,会展示在文档页面
    访问地址：http://项目实际地址/swagger-ui.html
    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("Spring Boot中使用Swagger2构建RESTful APIs")
                .description("演示 Swagger 的使用")

                .version("V1.0")
                .build();
    }
}
```

==*API文档内容*==

```java
@Controller
@RequestMapping("/say")
@Api(value = "SwaggerController | 一个用来测试 Swagger 注解的 controll≤er")
public class SwaggerController {

    @RequestMapping(value = "/getName", method = RequestMethod.GET)
    @ApiOperation(value = "根据 ID 获取用户名称", notes = "仅有 1 和 2 正确返回")
    @ApiImplicitParam(paramType = "query", name = "id", value = "用户 ID", required = true, dataType = "Integer")
    public @ResponseBody String getName(@RequestParam Integer id){
        if(id == 1){
            return "id = 1";
        }else if (id == 2){
            return "id = 2";
        }else {
            return "未知";
        }
    }

    @RequestMapping(value = "/modify", method = RequestMethod.GET)
    @ApiOperation(value = "修改用户密码", notes = "根据 ID, userName, currPassword 修改用户密码")
    @ApiImplicitParams({
            @ApiImplicitParam(paramType = "query", name = "id", value = "用户 ID", required = true, dataType = "Integer"),
            @ApiImplicitParam(paramType = "query", name = "userName", value = "用户名称", required = true, dataType = "String"),
            @ApiImplicitParam(paramType = "query", name = "currPassword", value = "用户当前密码", required = true, dataType = "String"),
            @ApiImplicitParam(paramType = "query", name = "newPassword", value = "用户新密码", required = true, dataType = "String")
    })
    public @ResponseBody String modifyPassword(@RequestParam Integer id,
                                               @RequestParam String userName,
                                               @RequestParam String currPassword,
                                               @RequestParam String newPassword){

        if(id < 0 || id > 2){
            return "未知用户";
        }
        if(StringUtils.isEmpty(newPassword) || StringUtils.isEmpty(currPassword)){
            return "密码不能为空";
        }
        if(newPassword.equals(currPassword)){
            return "新旧密码不能相同";
        }
        return "密码修改成功";
    }
}
```

==*使用下面的 URL 可以查看文档信息*==

```properties
http://localhost:8080/swagger-ui.html
```