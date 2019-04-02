#
- shiro 应用程序安全四大基石
    - Authentication : 身份验证
    - Authorization : 授权
    - Session Management : 会话管理
    - cryptography : 密码学
- 组件 : [==*Subject*==](#subject), [==**securityManager(Shiro 的核心)**==](#securityManager), [==*Realms*==](#Realms)

## 组件说明
### ==**Subject**==<span id="subject"/>
- Subject : 主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者；
    - 一定要绑定到一个 SecurityManager.进行交互时,这些交互转换成与SecurityManager 的一些交互


### ==**SecurityManager**==<span id="securityManager"/>
- SecurityManager : ==*Shiro 的核心*==,安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互，如果学习过SpringMVC，你可以把它看成DispatcherServlet前端控制器；

### ==**Realms**==<span id="Realms"/>
- realm : 域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。
- 是Shiro和应用程序的安全数据之间的“桥梁”或“连接器”。当需要与与安全相关的数据(如用户帐户)进行实际交互以执行身份验证(登录)和授权(访问控制)时，Shiro会从一个或多个为应用程序配置的领域中查找这些内容。

从这个意义上说，领域本质上是一个特定于安全性的DAO:它封装了数据源的连接细节，并根据需要将关联的数据提供给Shiro。在配置Shiro时，必须指定至少一个用于身份验证和/或授权的领域。SecurityManager可以配置多个领域，但至少需要一个。

Shiro提供了开箱即用的领域，可以连接到许多安全数据源(即目录)，如LDAP、关系数据库(JDBC)、文本配置源(如INI)和属性文件，等等。如果缺省领域不满足您的需求，您可以插入您自己的领域实现来表示自定义数据源。

与其他内部组件一样，Shiro SecurityManager管理如何使用领域获取安全数据和标识数据，并将其表示为Subject实例。