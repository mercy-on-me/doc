# Javaweb 相关
## servlet 简介
- 运行在服务器端运行的一个 java 小程序,接收和响应磁
- servlet

## Tomcat 等级
Tomcat container->

## Servlet 开发流程
- 继承 HttpServlet
- 重写 doget dopost方法
- 在 web.xml中注册servlet



## Servlet 面试相关
1. Servlet 声明周期
    - 初始化 : 容器加载 servlet,调用 init 方法初始化 servlet
    - 处理请求 : 根据不用的请求调用 doGet 或者 doPost 方法
    - 销毁 : 服务结束后,

    
    
## cookie 和 session
### 1. cookie
- cookie :
    - 用户访问某一网站后在本地内存中以 k:v形式存储的网站的信息,并随每次请求发送到同一服务器.
    - 内容包括 : 名字,值,过期时间,路径和域
    - 相当于一个通行证,每个用户在访问服务器的时候都需要带上自己的通信证,然后服务器根据通行证确定用户的身份.客户端请求服务器,如果需要记录用户状态,就通过 response 向客户端办法一个cookie,然后客户端浏览器会把 coolie 保存起来,然后用户再次请求服务器的时候,就吧请求的网址连同这个 cookie 一起发送给服务器,然后服务器来验证客户身份,


### 2. session
- session : 
    - 就是会话,当有用户第一次访问某个服务器的时候,就会在服务器上产生一个 session,服务器会将用户的信息临时保存在这个 session 上,并返回 sessionid,客户端再次访问你服务器的时候,只需要从 session 中查找该客户的状态就可以了
    - 相当与客户明细表
    - 只有访问 jsp,servlet等程序的时候才会创建 session
    - 当用户离开网站后,session 就会被销毁.
    - 默认超时时间是 20 分钟
    - 但是如果为了负载均衡,那么下一个请求到达另一台服务器的时候,session 会丢失

    
### 3.相关问题
- 禁用 cookie 后怎么办?
    - URL 重写
        - 针对客户端不支持 cookie 的解决方案
        - httpServletPesponse 提供了URL 重写方法 : encodURL(url);
        - URL 重写的原理是 : 将该客户的session ID信息重写到 URL 地址中,服务器能够解析重写后的 URL 获取 sessionid,这样哪怕进制了 cookie,也可以通过 session ID 来记录客户状态.
    - Tomcat 判断客户端是否支持 cookie 的依据是请求中是否含有 cookie,尽管客户端支持 cookie,但是第一次的访问的时候也是没有 cookie 的,所有第一次访问的 URL 中有 jsesessionid,但是第二次访问的时候服务器已经在浏览器中写入了 cookie,所以后边的访问不会带有 jsesessionid.

