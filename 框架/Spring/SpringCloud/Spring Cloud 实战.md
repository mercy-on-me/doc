# Spring Cloud (基于 springboot 2.0.0.M3, Springcloud Finchley.M2)
## 1. Eureka 
### 2.1 ==**Eureka Server (注册中心)**==
==**Eureka server 提供服务注册的功能,各个服务启动的时候会将 server 中进行注册,这样 Eureka server 中就有所有服务的信息**==

==**构建过程**==:
    - springboot 项目.引入 cloud discovery --> Eureka server(==*会自动引入 web模块*==)
    - 在主配置类上使用 ==*@EnableEurekaServer*==, 声明这是一个 Eureka 服务器.此时一个简单的 Eureka 服务器就构建完成了.
    - 在配置文件中配置端口为 ==*8761*==(不配置也没问题,只是因为 8761 是约定端口)

#### 2.1.1 构建 Eureka 服务器   
- ==**java 代码**==
    ```java
    /**
     * @EnableEurekaServer : 声明这是一个 Eureka 服务器
     */
    @SpringBootApplication
    @EnableEurekaServer
    public class EurekaServerApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaServerApplication.class, args);
        }
    }
    ```

- ==**application.yml配置文件**==

    ```properties
    server:
      port: 8761
    ```

- ==**登录 localhost:8761 , 查看 Eureka 服务器控制台**==

    ```properties
    http://localhost:8761
    ```

#### 2.1.2 以上构建出现的问题 :
- 用上边的构建方式,虽然也能查看 ==*Eureka 服务器控制台*==, 但是启动服务的时候可以看到报了两个错 : 

    ```properties
    com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused (Connection refused)
    .
    .
    com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
    .
    .
    ```
- 这是因为在服务器启动时,服务器会把自己作为一个客户端,去注册到 Eureka 服务器中,并会去 Eureka 服务器中抓取注册信息,但是它本身是一个 Eureka 服务器,而不是一个服务提供者(客户端)
- 可以在配置文件中使用两个配置解决以上问题 (==*register-with-eureka, fetch-registry*==):

    ```properties
    server:
      port: 8761
    eureka:
      client:
        register-with-eureka: false   # 声明是否将自己的信息注册到 Eureka 服务器中,默认为 true
        fetch-registry: false         # 是否去 Eureka 抓取注册信息
    ```
- 此时在启动项目,就不会报以上的错误信息

#### 2.2 ==**构建服务提供者**==
==**构建过程**==
- 创建 springboot 项目, 引入 cloud discovery -->  Eureka discovery.
- 在主配置类上使用 ==*@EnableEurekaClient*==, 声明这是一个 Eureka 客户端.
- 配置文件中指定 : ==*应用名称, 本服务的实例的主机名称, 要注册到哪个 Eureka 服务器*==
- ==**>>>>>>>注意要引入 starter-web 依赖,否则 Eureka 客户端会启动后马上停止<<<<<<**==

- ==**主配置类**==
    ```java
    /**
     * @EnableEurekaClient 声明这是一个 Eureka 客户端
     */
    @SpringBootApplication
    @EnableEurekaClient
    public class EurekaProviderApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaProviderApplication.class, args);
        }
    }
    ```

- ==**要提供的服务**==

    ```java
    /**
     * @program: springcloud_code
     * @description: Eureka 服务提供者演示
     * @author: cy
     * @create: 2019-04-27 19:42
     **/
    
    @Controller
    public class EurekaProviderController {
    
        @RequestMapping(value = "/person/{personId}")
        public Person getPersonInfo(@PathVariable("personId") Integer personId){
            Person person = new Person(personId, "wangwu", "man");
            return person;
        }
    }
    ```
    
- ==**Person实体类略过,属性有 id,name,sex**==

- ==**配置文件**==

    ```properties
    spring:
      application:
        name: eureka-provider           # 设置应用的名称,在 Eureka 服务器控制台里边展示的注册列表中展示的服务名称
    eureka:
      instance:
        hostname: localhost             # 设置服务实例的主机名
    
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/    # 设置将此服务注册到哪个 Eureka 服务器中.
    ```

- ==**演示配置结果**==
    - 登录 Eureka服务器控制台,可以在 ==*Instances currently registered with Eureka 列表中*== 看到注册的服务.展示的名称为配置文件中配置的 ==*spring.application.name*== 配置的名称
    
```properties
http://localhost:8761
```


### 2.3 构建服务调用者
==*服务调用者*== : 指同样注册到 Eureka 的客户端.这个客户端回去调用其他其他客户端发布的服务.简单的说就是 Eureka 的内部调用.

==**构建过程**==
- 引入 ==*discovery*==
- ==**>>>>>>>注意要引入 starter-web 依赖,否则 Eureka 客户端会启动后马上停止<<<<<<**==
- ==*引入 Ribbon*==
- 在主配置类中使用注解 : ==**@EnableDiscoveryClient**==, 这个注解 ==*使得服务有能力去 Eureka 中发现服务*==. (==**@EnableEurekaClient 中包含 @EnableDiscoveryClient 的功能,也就是说一个 Eureka客户端本身就具有发现服务的能力**==).
- 编写controller,并使用@Configuration 说明是个配置类,返回RestTemplate 的 bean ,提供调用其他客户端的功能,需要使用 ==**[RestTemplate](#RestTemplate)**== 类.

- ==**配置类**==

    ```java
    /**
     * @EnableDiscoveryClient : 使服务调用者有去 Eureka 中发现服务的能力
     */
    @SpringBootApplication
    @EnableDiscoveryClient
    public class EurekaInvokerApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaInvokerApplication.class, args);
        }
    }
    ```
- ==**controller,调用服务提供者提供的服务**==

    ```java
    @Configuration
    @RestController
    public class InvokerController {
    
        /**
         * RestTemplate 是 spring-web 模块的类,具有调用 REST 服务的能力, 本身并不具备访问分布式服务的能力.
         *  但是被 @LoanBalanced 修饰后,就具备了访问分布式服务的能力.
         * @return
         */
        @Bean
        @LoadBalanced
        public RestTemplate getRestTemplate(){
            return new RestTemplate();
        }
    
    
        /**
         * 提供一个服务,这个服务可以去调用服务提供者提供的服务
         * @return
         */
        @GetMapping(value = "/router", produces = MediaType.APPLICATION_JSON_VALUE)
        public String router(){
            RestTemplate restTemplate = getRestTemplate();
            /**
             * http://eureka-provider/person/{personId}
             *      eureka-provider : 服务提供者配置文件中配置的服务的名称,也就是 Eureka 服务器控制台展示的注册的服务名称
             *      /person/{personId} : 服务提供者提供的服务.
             *      这里配置的 URL 是调用服务提供者提供的 mapping 为 person,参数为 1
             */
            String json = restTemplate.getForObject("http://eureka-provider/person/1", String.class);
            return json;
        }
    }
    ```
- ==**配置文件**==

    ```properties
    server:
      port: 9000
    
    spring:
      application:
        name: eureka-invoker
    
    eureka:
      instance:
        hostname: localhost
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/
    ```

- ==**调用服务**==

```properties
# 调用服务调用的 router 服务,这个服务里边调用了服务提供者 eureka-provider 提供的服务
http://localhost:9000/router 
```




### 2.4 Eureka 集群搭建
#### 2.4.1 模拟环境搭建
模拟环境 : 根据上边的服务进行模拟, 需要两个 Eureka server 和两个 Eureka 服务提供者
==**两个 Eureka server 模拟方式**== : 
- 同一个配置文件中,将配置拷贝一份,port 设置为 8761 和 8762
- ==*相互注册 !*==, 8761 注册到 8762, 8762 注册到 8761
- 复制 run configurations.一份以 8761 启动,一份以 8762 启动

==**两个服务提供者模拟方式**==
- Person 类添加 message,以输出 URL
- Controller 中输出 URL
- 配置文件中添加端口配置,分别为 8080 和 8081
- 配置 ==*eureka.client.service-url*==,写连个地址,注册到8761 和 8762,以 ==*逗号*== 隔开
- 复制 run configurations ,一份以 8080 启动,一份以 8081 启动

==**服务调用者模拟方式**==
- 配置 ==*eureka.client.service-url*==, 注册地址写两个,分别是 8761 和 8762,并以逗号隔开

==**编写 Rest测试类**==

#### 2.4.2 模拟实现
==**Eureka Server(只需要修改配置文件)**==

```properties
# 演示 Eureka 集群 : server, 此 server 注册到 http://localhost:8762/eureka
#server:
#  port: 8761
#
#spring:
#  application:
#    name: eureka-server-1
#
#
#eureka:
#  client:
#    service-url:
#      defaultZone: http://localhost:8762/eureka
#  instance:
#    hostname: server-1


# 演示 Eureka 集群 : server, 此 server 注册到 http://localhost:8761/eureka
server:
  port: 8762

spring:
  application:
    name: eureka-server-2


eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    hostname: server-2
```

==**服务提供者**==
- ==*controller*==

    ```java
    /**
     * @program: springcloud_code
     * @description: Eureka 服务提供者演示
     * @author: cy
     * @create: 2019-04-27 19:42
     **/
    
    @RestController
    public class EurekaProviderController {
    
        @GetMapping(value = "/person/{personId}", produces = MediaType.APPLICATION_JSON_VALUE)
        public Person getPersonInfo(@PathVariable("personId") Integer personId, HttpServletRequest request){
            Person person = new Person(personId, "wangwu", "man", request.getRequestURL().toString());
            return person;
        }
    }
    ```

- ==*配置文件*==

    ```properties
    spring:
      application:
        name: eureka-provider          # 设置应用的名称,在 Eureka 服务器控制台里边展示的注册列表中展示的服务名称
    eureka:
      instance:
        hostname: localhost             # 设置服务实例的主机名
    
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/,  http://localhost:8762/eureka/   # 设置将此服务注册到哪个 Eureka 服务器中.
    
    
    # 模拟两个服务提供者, 分别为 8080 和 8081
    #server:
    #  port: 8080
    
    # 模拟两个服务提供者, 分别为 8080 和 8081
    server:
      port: 8081
    ```

==**服务调用者**==
- ==*配置文件*==

    ```properties
    server:
      port: 9000
    
    spring:
      application:
        name: eureka-invoker
    
    eureka:
      instance:
        hostname: localhost
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/, http://localhost:8762/eureka/
    ```
  
==**REST 测试类**==

```java
/**
 * 模拟 Eureka 集群 : 调用服务,查看 URL
 */
@SpringBootApplication
public class RestHttpclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(RestHttpclientApplication.class, args);
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet("http://localhost:9000/router");
        for (int i = 0; i < 100; i++){
            try {

                CloseableHttpResponse execute = httpClient.execute(httpGet);
                System.out.println(EntityUtils.toString(execute.getEntity()));
            } catch (Exception e) {
                e.printStackTrace();
            }

        }
    }
}
```  

==**模拟结果**==
- 运行 REST 测试类,结果如下:

```properties
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8080/person/1"}
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8081/person/1"}
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8080/person/1"}
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8081/person/1"}
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8080/person/1"}
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8081/person/1"}
{"id":1,"name":"wangwu","sex":"man","message":"http://localhost:8080/person/1"}
.
.
.
```


#### 2.5 ==**[Eureka 常用配置](#eureka-config)**==

#### 2.6 元数据的使用
- 如果需要自定义元数据,并提供给给其他客户端使用,可在配置文件中进行配置,具体配置见 ==*[Eureka 常用配置](#eureka-config)*==

==**元数据的使用(基于上边已有的代码实现)**==
- ==*服务提供者配置文件中添加元数据配置*==

    ```properties
    eureka:
      instance:
        metadata-map:
          metaValue: custom_meta_data   # 添加元数据, key value 自动以
    ```
    
- ==*服务调用者中获取元数据*==

    ```java
    @Configuration
    @RestController
    public class InvokerController {
    
        @Autowired
        private DiscoveryClient discoveryClient;    //用于获取元数据
    
        @GetMapping(value = "/meta")
        public String getMeta(){
            //获取服务实例(参数为 服务提供者配置的 spring.application.name)
            List<ServiceInstance> instances = discoveryClient.getInstances("eureka-provider");
            for (ServiceInstance ins : instances) {
                //获取元数据
                System.out.println(ins.getMetadata().get("metaValue"));
            }
            return "";
        }
    }
    ```

- ==*结果*==

    ```properties
    # 之所以输出两边,是因为服务提供者有两个,分别是 8080 和 8081
    custom_meta_data
    custom_meta_data
    ```


    
- ==**主配置类**==

    ```java
    @SpringBootApplication
    @EnableEurekaServer
    public class EurekaApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(EurekaApplication.class, args);
        }
    
    }
    ```

- ==**登录页面**==

```properties
http://localhost:8761/  <!--因为配置文件中配置的端口是 8761-->
```

- 以后每次都需要先将 Eureka 启动起来,如果每次都打开 idea 进行启动,太麻烦,可以打个 jar 包,生成的 jar 包在 target 目录下

```shell
<!--打 jar 包-->
mvn clean package
<!--进入 target 目录-->
cd target/
<!--启动 Eureka, 方式一 : 命令行启动,但是一旦使用 control + c ,服务就关闭了-->
java -jar xxxxx.jar
<!--启动 Eureka, 方式二 : 后台启动,这样只要不 kill 进程,服务就一直在 -->
nohup java -jar eureka-0.0.1-SNAPSHOT.jar > /dev/null 2>&1 &
```


#### 2.2 ==**Eureka Client (服务注册)**==

- 代码演示流程
    - 引入 cloud discovery --> Eureka discovery
    - 版本要和 Eureka server 统一
    - 在主配置类上添加注解 : ==**@EnableDiscoveryClient**==
    - 想要注册 ==*注册中心(Eureka server)*== 中去,就需要在配置文件中配置注册中心地址 : eureka.client.server-url : defaultZone: http://localhost:8761/eureka/
    - 修改注册进入的应用的名称 spring.application.name: xxxx
    - 注册进入后,可以看到名称的后边有一个 IP,可以点进入,如果想设置点进入的 IP 跳到另一个 IP,可以使用配置 : eureka.instance.hostname: xxx
    - ==*如果不停的重启 client 应用,页面就会
        - 采用心跳机制,server 会不停的去检查 client 是否存活,会计算在一定时间内 client 上线的比例.当低于一定的比例时,会不知道 client 是否启动,就宁可信其有!就认为 client 是启动的.但是此时可能是没启动的.
        - 所以可以关闭这一机制 : 在 server 端添加配置 : eureka.server.enable-self-preservation: false(默认为 true). 开发环境可以关闭,但是==**生产环境一定要是 true!!!!!**==

- ==*配置文件*==

    ```yml
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/  #注册中心地址
    
      instance:
        hostname: eureka.client_1 #自定义跳转的 IP
    
    spring:
      application:
        name: client_1
    server:
      port: 8761
    ```
    
- ==*配置类*==

    ```java
    @SpringBootApplication
    @EnableDiscoveryClient
    public class EurekaClientApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(EurekaClientApplication.class, args);
        }
    
    }
    ```
    

#### 2.3 Eureka 高可用
- 多个 Eureka server ,一个 server 分别向其他几个 server 中注册
- Eureka Client 分别向两个 Eureka server 中注册,就是注册地址以逗号隔开,分别写两个 server.

- ==*Eureka server*==

    ```yml
    # server_1
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8762/eureka/,http://localhost:8762/eureka/
        register-with-eureka: false   #eureka 就是一个注册中心,所以不需要注册上去,使用此配置,就可以不让它注册上去
      server:
        enable-self-preservation: false # 不注册本身
    spring:
      application:
        name: eureka
    #    
    # server_2
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/,http://localhost:8763/eureka/
        register-with-eureka: false   #eureka 就是一个注册中心,所以不需要注册上去,使用此配置,就可以不让它注册上去
      server:
        enable-self-preservation: false # 不注册本身
    spring:
      application:
        name: eureka
    #   
    # server_3
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
        register-with-eureka: false   #eureka 就是一个注册中心,所以不需要注册上去,使用此配置,就可以不让它注册上去
      server:
        enable-self-preservation: false # 不注册本身
    spring:
      application:
        name: eureka
    ```

- ==*Eureka Client*==

    ```yml
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/,   http://localhost:8762/eureka/,http://localhost:8763/eureka/ #注册中心地址
    
      instance:
        hostname: eureka.client_1 #自定义跳转的 IP
    
    spring:
      application:
        name: client_1
    ```
    
#### 2.4 ==**Eureka 总结**==
1. ==*@EnableEurekaServer*== : 开启 Eureka Server
2. ==*@EnableDiscoveryClient*== : 开启 Eureka Client
3. ==**Eureka server 提供服务注册的功能,各个服务启动的时候会将 server 中进行注册,这样 Eureka server 中就有所有服务的信息**==
4. Eureka在系统运行期间,如果有个服务挂掉了,没有在心跳的时间发送信息,Eureka 就会将挂掉的服务剔除掉. (心跳检测机制, 健康检查, 负载均衡等功能)
5. Eureka 的高可用,生产上需要多台机器,至少两台以上

#### 2.5 ==**分布式下服务注册的地位和原理**==
- 为什么要用注册中心
    - A 和 B 两个服务都有各自的地址,如果 A 需要调用 B,那么 A 中配置 B 的地址就可以找到 B 的服务了
    - 但是如果 B 是分布式服务,每服务都有自己的地址,此时就需要注册中心,启动的时候就会把自己额信息注册到注册中心去,此时如果 A 需要调用 B 的话,就去注册中心取信息就行了.注册中心是 A 和 B 沟通的桥梁.
- A 怎么找到 B?
    - ==*客户端发现*== : 注册中心将其中的信息都告诉 A ,然后 A 根据负载均衡的方式调用其中的一个 B. ==**Eureka**==就是客户端发现
    - ==*服务端发现*== : 会有一个==*代理,*==代理从众多可用的 B 中挑选出一个可用的出来,然后 A 再调用 B
    - 两种方式的优缺点
        - 客户端发现 : 
            - 优点 : 不需要代理的介入, A 知道所有的 B 的实际信息
            - 缺点 : A 服务需要自己实现一套逻辑去挑选 B

        - 服务端发现 :
            - 有点 : 由于代理的作用, B 对 A 是不可见的,A 只需要找代理发一个请求就好了

- ==*[异构]("#yigou")的时候,改怎么通过注册中心调用其他服务?*==
    - 微服务是轻量级的通信,有==*Rest*== 和 ==*RPC*==, Springcloud 选择的是 ==**REST**==
    - Eureka 的服务端提供了较为完善的 REST API,对外提供 API
    - 也支持非 Java 语言实现的服务纳入到自己的服务体系中,只需要其他语言自己实现的 Eureka 客户端 API .

    
    
    
    
    
    
    
## 手册

### RestTemplate <span id="RestTemplate"/>
这个类时 spring-web 模块下的类,主要用来调用 ==*REST*== 服务,本身不具备调用分布式服务的功能,但是被 ==**@LoanBalance**== 注解修饰后,就具备了调用分布式服务的能力.


### 注解

```properties
@EnableEurekaServer : 声明这是一个 Eureka Server
@EnableEurekaClient : 声明这是一个 Eureka Client (包含有 EnableDiscoveryClient 的功能)
@EnableDiscoveryClient : 使服务调用者有去 Eureka 中发现服务的能力
@LoadBalanced : 配合 RestTemplate 使用,使得 RestTemplate 有调用分布式服务的能力.
```

<span id="eureka-config"/>
### Eureka 常用配置

```properties
心跳检测配置
# 配置发送心跳周期, 默认 30秒一次
eureka.instance.leaseRenewalIntervalInseconds : 配置发送心跳的周期

# 服务器端接收心跳请求,如果一定期限内没有收到服务实例的心跳,就会将该实例从注册表中剔除,其他的客户端将无法访问这个实例.这个期限默认是 90 秒.
eureka.instance.leaseExpirationDurationInSeconds : 剔除一定期限内没有发送心跳的服务实例的期限

# 如果一定期限内没有收到服务实例的心跳,清理注册表有一个定时器执行,默认 60 秒一次,如果将剔除没法送心跳的服务的期限设置为小于 60 秒,虽然符合剔除服务的要求,但是还没到 60 秒,那么该服务将依然存在于注册表中,因为还没有执行清理嘛.要等到 60 秒的时候才会清理,这个定时器的执行时间可以通过以下配置项配置 : 
eureka.server.eviction-interval-timer-in-ms : 设置注册表清理定时器的时间

# 自我保护模式,如果启动自我保护模式,服务没有发送心跳也不会被剔除,可通过以下配置开启或者关闭自我保护模式
eureka.server.enable-self-preservation = false : 关闭自我保护模式


注册表抓取时间
# 客户端默认每隔 30 秒去服务器端抓取注册表(可用服务列表),并将服务端的注册表保存到本地缓存中. 可通过以下配置配置抓取时间 : 
eureka.client.registryFecth-IntervalSeonds : 配置客户端去服务端抓取注册表的时间间隔


配置与使用元数据
# 框架自带元数据,包括实例 ID, 主机名称, IP 地址等,如果需要自定义元数据并提供给其他客户端使用,可通过以下配置项进行配置 : 
eureka.instance.metadata-map : 配置自定义元数据,数据可是为 map 形式.key,value 都是自动以
# 元数据都会保存在注册表中, 使用元数据的一方可通过 discoveryClient 的方法获取元数据

```
