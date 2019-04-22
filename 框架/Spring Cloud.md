# Spring Cloud (基于 springboot 2.0.0.M3, Springcloud Finchley.M2)
- 本文档需用用到 springboot, Docker 等技术
- 基于慕课网 < Springcloud 微信点餐> 
    - 单体架构
    - 基于 Ajax 的前后端分离
    - 分布式 ( 水平扩展 & 服务拆分 )
- Springboot 基于 spring,实现简单构建项目
- Spring cloud 基于 springboot, 实现==*简化构建分布式项目*==
- 微服务是一种==*架构风格*==
    - 由一系列微小的服务共同组成
    - 分别有自己独立的进程
    - 每个服务为独立的业务开发
    - 独立部署
    - 分布式管理
    
- 微服务是由一系列微小的服务功能组成的
    - 服务间通信需要 服务发现与注册
    - 与外部通信需要 服务网管(Service Gateway),所有的服务都要通过网关.
        - 屏蔽后台的一些细节,比如后台升级
        - 有路由的功能
        - 做限流和容错
        - 监控和日志
        - 微服务的安全性(验证,权限等)
    - 后端通用服务 (也称中间层服务 Middle Tier Service)
        - 启动时将地址
    - 前端服务 (也称边缘服务 Edge Service)
        - 对后端服务进行必要的聚合和裁剪后,暴露给外部

<span id="yigou"/>
- 微服务的特点 : ==**异构**==
    - ==**异构**== : 不同的语言,不同的数据库.
    - 越大的微服务项目,就越有可能出现一个项目中不同的开发语言的情况,不同的语言又会根据情况选择不同的数据库.

- 微服务的调用方式 : ==**轻量级通信: REST 或者 RPC**==
    

## 1. Spring cloud 简介
- Spring cloud 是一个==*开发工具集*==,包含多个子项目
- 利用 Springboot 的开发便利性,实现了很多功能
- 主要是基于对 Netflix(美国的一家视频网站,分布式做的最牛批的) 开源组件的进一步封装


## spring cloud 组件
### 1. Spring cloud 组件
1. Eureka : 负责 ==*服务发现*==
    - Eureka Server
    - Eureka Client
    - Eureka 高可用
    - 服务发现机制
2. Config : 负责 ==*统一配置中心*==
    - Config Server
    - Config Client
    - Spring cloud Bus (结合 RabbitMQ实现配置自动刷新)
3. Ribbon : 负责==*服务通信*==
    - RestTemplate
    - Feign
    - 分析 Ribbon 源码
4. Zuul : 
5. Hystrix : 熔断机制
    
### 2. Eureka (服务注册与发现)

- Eureka 基于 Netflix Eureka 做了二次封装
- Eureka 由两部分组成 :
    - ==*Eureka Server*== : 注册中心
    - ==*Eureka Client*== : 服务注册
    - 原理 : 
        - 服务端供服务注册的服务器.
        - 客户端用来简化与服务器的交互,作为轮询负载均衡器,并提供服务的故障切换
        - server : 系统中的其他服务通过客户端连接到 server,并维持心跳连接.(每隔一段时间就会去注册)
        - 这样就能监控系统中服务是否正常运行

#### 2.1 ==**Eureka Server (注册中心)**==
- ==**Eureka server 提供服务注册的功能,各个服务启动的时候会将 server 中进行注册,这样 Eureka server 中就有所有服务的信息**==
- 代码演示
    - 引入 cloud discovery --> Eureka server
    - 加入注解 : ==*@EnableEurekaServer*== (此时可以访问页面,但是会报错,原因是以为没有这个应用既是server又是client)
    - 配置 URL : eureka.client.service-url = http://localhost:8080/eureka,但是此时还是不行,点进去 service-url 会发现,他的值是也 map,key为defaultZone,所以需要设置为下面的配置形式
    - 配置服务名称 : spring.application.name
    - 这个服务本来就是个注册中心,所有不需要展示在注册的应用列表内,使用 eureka.client.rregister-with-eureka: false, 这样就不会注册上去了
    - 以后的应用会很多,所以 URL 的端口最好改一下,改为自己的端口,项目 port 最好也改一下
    - 使用 localhost:8080 即可登录查看

- ==**配置文件**==    
    ```yml
    eureka:
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/    # defaultZone 是 service-url 的值的 key
        register-with-eureka: false   #eureka 就是一个注册中心,所以不需要注册上去,使用此配置,就可以不让它注册上去,不加此配置,可以在页面中 "Instances currently registered with Eureka" 中看到注册的服务
    spring:
      application:
        name: eureka
    
    server:
      port: 8761
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