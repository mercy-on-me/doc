# Spring
## 1. 简介
spring 基础包: core, context, beans, expression

### 1.1 相关类介绍 
- 工厂类

    ```properties
    BeanFactory : (旧版本的工厂类)最顶层的工厂,创建对象的方式用的是延迟加载,就是加载配置文件的时候,没有创建对象,在调用getBean(“”)方法的时候,才创建对象.
    ApplicationContext : (新版本的工厂类)也是工厂,创建对象的方式:只要加载到配置文件,立即就把配置的单例模式的类给实例化了. 有连个实现类 :  
        ClassPathXmlApplicationContext : 加载类路径下的配置文件
        FileSystemXmlApplicationContext : 加载文件系统下的配置文件(比如说 C 盘下, D 盘下,这种路径)
    ```

- ==**BeanPostProcessor**==

    ```properties
    BeanPostProcessor : 可以在生成类的过程中,对类生成代理,并进行增强.
    ```
    
## 2. IOC 控制反转()
### 2.1 IOC 概述
- IOC : (==*Bean 管理*==)将对象创建的权利反转给 Spring框架,由 Spring 框架控制对象的创建.也就是 : 将类交给 Spring 管理.

### 2.2 IOC 底层实现原理
- 最开始 : 直接 new 一个对象,此时接口和实现类有耦合
- 工厂 : 通过工厂类创建对象,此时接口个工厂类有耦合
- 最后 : 使用 ==*工厂+反射+配置文件*==
- 通过配置文件装载这些需要使用的 bean,然后工厂读取配置文件,通过 ID 获取到 bean,最后通过反射创建 bean 的实例

- IOC 的实现
    - 工厂 + 配置文件 + 反射
    - 配置文件 : 指定需要交给 Spring 管理的类
    - 工厂 : 通过读取配置文件,获取到类的 class
    - 反射 : 工厂获取到 class 后,通过反射创建对象

### 2.3 演示 : 
#### 2.3.1 配置文件方式
- 导包 : ==*基础包*==
 
- ==*配置文件*==   
- spring 默认配置文件名称为: ==**applicationContext.xml**==, 可修改

    ```xml
    <!--
        id: 当前 Bean 的唯一标识,通过 ID 获取 Bean
        class: 当前 Bean 的全路径
        scope: 对象的作用范围
            singleton : 单例,作用范围是整个应用.应用加载,创建容器的时候对象被创建,只要对象存在,就一直存活,应用被卸载的时候,对象被销毁
            prototype : 多例,每次访问对象都会创建.  使用对象的时候就创建,只要对象在使用中,就一直存活,长时间不用此对象,被 java垃圾回收机制回收
        init-method: 初始化方法.对象创建出来后,马上执行初始化方法
        destroy-method: 销毁方法
    -->
    <bean id="userService" class="com.cy.service.impl.UserServiceImpl" scope=""  init-method="initBean()" destroy-method=""/>
    ```
- ==接口和实现类正常编写==
- ==测试类==

```java
@Slf4j
public class UserServiceImplTest {

    ApplicationContext contecxt = null;

    @Before
    public void getContext(){
        contecxt = new ClassPathXmlApplicationContext("applicationContext.xml");
    }


    @Test
    public void test1() {
        UserService userService = (UserService)contecxt.getBean("userService");
        userService.test();
    }
}
```
#### 2.3.2 注解方式
- 实现方式: 
    - 开启包扫描
    - 将类交给 Spring 管理.@Service 等注解
    - 如果使用配置类的话.参考 [注解](#zhujie)


### 2.4. DI 依赖注入
- 依赖注入
    - 依赖注入是 IOC 的实现方式,就是在 spring 创建这个对象的过程中,将这个对象所依赖的属性注入进入.比如 UserserviceImpl 中有个属性为 name,那么 DI 就是在创建 UserserviceImpl 的过程中,将 name 属性注入进 UserServiceImpl对象中.(==*必须是先有 IOC*==)
- DI 的两种方式:
    - 构造方法注入
    - set 方法注入

#### 2.4.1 构造方法注入
- 实现方式:  
    - Bean 创建构造方法,需要注入哪些属性就在构造方法中添加哪些属性
    - 配置文件中这个 Bean 的是,添加 Bean 的子标签 : constructor-arg,指定属性的名称
    - 测试中正常通过context 获取
- 演示

    ```xml
    <bean id="位置标识" class="类全路径">
        <constructor-arg name="类的属性" value="属性的值"/>
        ...
    </bean>
    ```

#### 2.4.2 set 方法注入
- 实现方式
    - 为属性提供 set 方法
    - 配置文件中为 Bean 添加子标签 property,此标签有属性 namem,value(还有 ref)
- 演示

    ```xml
    <bean id="位置标识" class="类全路径">
        <property name="类的属性" value="属性的值"/>
        <property name="类中有属性是对象,这个对象也叫给了 Spring 管理" ref="配置文件中已经配置的对应 Bean 的 ID"/>
        ...
    </bean>
    ```

### 2.5. Spring Bean 管理
- Spring 实例 Bean 的三种方式 : 
    - 是否类构造器实例化(==*默认使用无参构造器*==)
    - 使用静态工厂方法实例化(==*简单工厂模式*==)
    - 使用实例工厂方法实例化(==*工厂方法模式*==)
  
### 2.5.1 类构造器实例化(==*默认使用无参构造器*==)
- 配置文件配置 bean
- 通过工厂获取bean实例

### 2.5.2 静态工厂方法实例化(==*简单工厂模式*==)
- 编写工厂类,此类中 new 要使用的 bean
- 配置文件中配置工厂类,并指定 factory-method
- 加载配置文件,获取 工厂类 实例
- ==**静态工厂类**==

    ```java
    //静态工厂类
    public class Bean2Factory {
        public static Bean2 createBean2(){
            System.out.println("通过静态工厂创建bean2");
            return new Bean2();
        }
    }
    ```

- ==**配置文件**==

    ```xml
   <!--  静态工厂方式  
        class : 静态工厂类
        factory-method : 静态工厂类方法
    -->
    <bean id="bean2" class="com.cy.ioc.demo2.Bean2Factory" factory-method="createBean2"/>    ```

- ==**测试**==

    ```java
    @Before
        public void getContext(){
            context = new ClassPathXmlApplicationContext("applicationContext.xml");
        }
        
    @Test
        public void bean2(){
            Bean2 bean2 = (Bean2)context.getBean("bean2Factory");
        }
    ```

#### 2.5.3 实例工厂方法实例化(==*工厂方法模式*==)
- 和静态工厂方式的区别是 : create 方法不是静态的
- 所以需要现有工厂实例

- ==**静态工厂类**==

    ```java
    public class Bean3Factory {

        public Bean3 createBean3(){
            System.out.println("实例工厂方式创建 bean");
            return new Bean3();
        }
    }
    ```

- ==**配置文件**==

    ```xml
  <!--  实例工厂方式
      先配置工厂类的 bean
    -->
    <bean id="bean3Factory" class="com.cy.ioc.demo2.Bean3Factory"/>
    <!--  实例工厂方式
        配置要使用的 bean
        factory-bean : 配置使用的工厂实例
        factory-method : 工厂类的方法
    -->
    <bean id="bean3" factory-bean="bean3Factory" factory-method="createBean3"/>
    ```

- ==**测试**==

    ```java
    @Before
        public void getContext(){
            context = new ClassPathXmlApplicationContext("applicationContext.xml");
        }
        
     @Test
    public void bean3(){
        Bean3 bean3 = (Bean3)context.getBean("bean3");
    }
    ```

### 2.6 Bean 配置项

```properties
id : bean 在Spring 当中唯一的标识
name : 也是 bean 在 Spring 中的标识,虽然没有要求唯一,但是使用时还是要设置为唯一的,当 bean 的名称中含有特殊字符时,就需要用 name 属性
scope: 对象的作用范围
    singleton : (默认)单例,作用范围是整个应用.应用加载,创建容器的时候对象被创建,只要对象存在,就一直存活,应用被卸载的时候,对象被销毁
    prototype : 多例,每次访问对象都会创建.  使用对象的时候就创建,只要对象在使用中,就一直存活,长时间不用此对象,被 java垃圾回收机制回收
    request : 每次 HTTP 请求都会创建一个新的 Bean,将创建出来的对象,除了放在容器一份之外,还会放到request域中一份.仅仅适用于 WebApplicationContext 环境
    session : 同一个HTTP session 共享一个 bean,不同的HTTP Session 使用不同的 bean. 放在session域中一份. 仅仅适用于 WebApplicationContext 环境中
init-method: 初始化方法.对象创建出来后,马上执行初始化方法
destroy-method: 销毁方法
```

### 2.7 Bean 的生命周期
- init-method : 初始化方法
- destroy-method : 当是==*单例*==的时候,可以调用这个方法
- Spring 容器中 bean 的完整的生命周期 : ==*一共十一步*==
    
    ```properties
    第一步 : 实例化对象
    第二步 : 设置属性
    第三步 : 如果 Bean 实现了 BeanNameAware, 则要指定 setBeanName.设置 bean 的名称
    第四步 : 如果 Bean 实现了 BeanFactoryAware 或者 ApplicationContextAware,则需要执行 setBeanFactory 或者 setApplicationContext.让 bean 了解工厂信息.
    第五步 : 除了这个 Bean 之外,如果有其他的一个类实现了 BeanPostProcessor,则需要执行 postProcessBeforeInitialization,是初始化之前要执行的方法
    第六步 : 如果 Bean 实现了 InitializingBean,则需要执行 afterPropertiesSet, 是设置属性后执行的方法
    第七步 : 执行 init-method 方法
    第八步 : 除了这个 Bean 之外,如果有其他的一个类实现了 BeanPostProcessor,则需要执行 postProcessAfterInitialization, 是初始化之后执行的方法.
    第九步 : 执行业务方法
    第十步 : 如果 Bean 实现了 DisposableBean ,则会执行 spring 的销毁方法 destroy
    第十一步 : 执行 destroy-method 方法
    其中第五步和第八步可以实现对此 Bean 的增强
    ```

- ==**代码演示**==
- ==**Bean**==
    ```java
    public class Bean implements BeanNameAware, BeanFactoryAware, InitializingBean, DisposableBean {
    
        private String name;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            System.out.println("第二步 : 设置属性");
            this.name = name;
        }
    
        public Bean() {
            System.out.println("第一步 : 初始化 bean");
        }
    
        @Override
        public void setBeanName(String name) {
            System.out.println("第三步 : 设置 bean 的名称");
        }
    
        @Override
        public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
            System.out.println("第四步 : 了解工厂信息");
        }
    
        @Override
        public void afterPropertiesSet() throws Exception {
            System.out.println("第六步 : 设置属性后执行的方法");
        }
    
        public void setup(){
            System.out.println("第七步 : 执行开发人员自己指定的 init-method 方法");
        }
    
        public void run(){
            System.out.println("第九步 : 执行实际的业务方法");
        }
    
    
        @Override
        public void destroy() throws Exception {
            System.out.println("第十步 : 执行 spring 的销毁方法");
        }
    
        public void destoryMethod(){
            System.out.println("第十一步 : 执行开发人员自己指定的 destroy-method 方法");
        }
    }
    ```

- ==**实现 BeanPostProcessor 的类**==

    ```java
    public class MyBeanPostProcessor implements BeanPostProcessor {
        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("第五步 : 初始化前执行的方法");
            return null;
        }
    
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            System.out.println("第八步 : 初始化后要执行的方法");
            return null;
        }
    }
    ```
    
- ==**配置文件**==

```xml
<!--配置 Bean-->
<bean id="bean" class="com.cy.lifecycle.Bean" init-method="setup" destroy-method="destoryMethod">
    <property name="name" value="生命周期演示"/>
</bean>

<!--配置 BeanPostrocessor-->
<bean class="com.cy.lifecycle.MyBeanPostProcessor"/>
```

- ==**测试**==
    
    ```java
    public class SpringIOC {
    
        private ClassPathXmlApplicationContext context = null;
    
        @Before
        public void getContext(){
            context = new ClassPathXmlApplicationContext("applicationContext.xml");
        }
    
        @Test
        public void cycleTest(){
            Bean bean = (Bean)context.getBean("bean");
            bean.run();
            context.close();
        }
    }
    ```

    
## 3. AOP
### 1. AOP 概述
- 通过动态代理.将横向的功能抽取出来,然后在使用的地方插入这些功能.
- 动态代理实现,两种方式: Proxy ,CGLib

```java
//Proxy 方式,只能对接口使用
//生成代理对象,并调用方法时,增强方法就执行了
public class MyProxy {
	 //动态代理实现, us要被增强的对象.必须要 final 修饰
    public static UserService getProxy(final UserService us){
        //生成代理
        Object proxyInstance = Proxy.newProxyInstance(us.getClass().getClassLoader(), us.getClass().getInterfaces(), new InvocationHandler() {
            //proxy : 代理对象,不要使用,否则就递归了
            //method : 标识被代理对象的方法.
            //args : 参数
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("增强了...前");
                //这里指在代理对象调用方法是,会自动执行被代理对象的方法.
                Object invoke = method.invoke(us, args);
                System.out.println("增强了...后");
                return invoke;
            }
        });
        return (UserService)proxyInstance;
    }


    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        UserService proxy = MyProxy.getProxy(userService);
        proxy.save();
        System.out.println("=================");
        proxy.update();
    }
}



//CGLib方式

```

### 2. AOP 术语

```properties
joinpoint : 连接点,
pointCut : 切入点.
Advice : 通知.具体的增强
    5种通知类型: 
        Before : 前置通知,目标方法调用前执行
        After : 后置通知,目标方法调用后执行
        After-retuning : 返回通知,目标方法执行成功后执行
        After-throwing : 异常通知,目标方法抛出异常后执行
        Around : 环绕通知,合并了 Before 和 After
introduction : 引介
target : 代理的目标.就是要增强的对象
Proxy : 代理.被 AOP 织入后,产生的接口代理对象
weaving : 织入.一个动态的过程,把增强应用到目标对象来创建新的代理对象的过程
aspect : 切面.切入点+通知
```

### 3. AOP实现
#### 3.1 配置文件方式

```xml
<!--开启aop 配置-->
<aop:config>
    <!--配置切面-->
    <aop:aspect ref="">
        <!--配置通知和切入点-->
        <aop:before method="" pointcut=""/>
    </aop:aspect>
</aop:config>
```

#### 3.2 注解方式
- 开启包扫描
- 开启Spring 对AOP 注解的支持
- 声明切面类,并交给 Spring 管理,编写通知方法

==配置文件==
```xml
<!--开启宝扫描-->
<context:component-scan base-package=""/>
<!--
    开启 Spring 对 AOP 注解的支持
    有属性: proxy-target-class : 配置代理方式
        true: 默认为 jdk
        false : cglib
-->
<aop:aspectj-autoproxy/>
```

==切面类==

```java
@Component
@Aspect
public class MyAspect{
    
    //通知方法
    //@Before : 前置通知,属性 value 指定切入点
    @Before(value="execution(* com.cy.*.save())")
    public void beforeAdvice(){
        ...增强代码
    }
}
```


#### 3.2 切入点表达式

```properties
* execution:匹配方法的执行
    * 语法
        * execution({修饰符} 返回值类型 包名.类名.方法名(参数))
    * 写法说明:
        * 修饰符:如果修饰符是:public,则可省略
        * 返回值类型:方法的返回值类型,如果和要增强的方法的返回值不同,那么会因为找不到匹配的方法而使这个配置不起作用,但是不会报错
            * 返回值类型可用星号(*),表示任意返回类型
        * 包名:也可以用星号(*)表示,表示任意包
            * 但是有几级包就要用几个星号
            * com.cy.demo.Demo:*.*.*.Demo

```
-----
### 事务管理
#### 1. 概述
- 使用 Spring 管理事务,要用到的核心接口:

```properties
PlatFromTransactionManager : Spring 平台事务管理器
    提供了两个实现类 :
        DataSourceTransactionManager:使用jdbc或者iBati运行持久化数据库时使用,在配置文件中配置bean
        HibernateTransactionManager:使用hibernate进行持久化时使用,在配置文件中配置bean

TransactionDefinition : 事务的定义信息对象
    getName:获取事务对象名称
    getIsolationLevel:获取事务隔离级别
    getPropagationBehavior:获取事务传播行为
    getTimeout:获取事务超时时间,默认值是-1,标识没有超时限制,如果有,以秒为单位进行设置
        如果运行时间超过设置的时间,这个方法没有执行完, 并且后边还涉及到对数据库的增删改查时,事务会回滚,如果时间超过设置时间,方法没有执行完,但是后边没有涉及到对数据库的增删改查,那么事务不会回滚
    isReadOnly:获取事务是否只读,一般查询时设置为只读

```

#### 2. 实现
#### 2.1 注解方式
- 开启扫描包
- 配置平台事务管理器的 Bean
- 开启对事务注解的支持
- 配置 jdbctemplate bean
- 在类或方法上使用注解: @Transactional,说明此类或者方法进行事务管理

==实现==

```xml
<!--开启扫描包-->
<context:component-scan base-package=""/>
<!--开启对事务注解的支持-->
<tx:annotation-driven transaction-manager="平台事务管理器 BeanID"/>
<!--使用 Spring 默认方式加载 JDBC 配置文件-->
<context:property-placeholder location="classpath:jdbc 配置文件"/>
<bean 配置数据源>
<bean id="平台事务管理器 BeanID" class="平台事务管理器类全路径">
    <property name="数据源 ID" ref="数据源 ID"/>
</bean>
```
==service层实现类==

```java
@Service
@Transactional(isolation=Isolation.DEFAULT) //也可以加在类上
public class UserServiceImpl implements UserService{
    ....业务代码
}
```

-----
## 注解
<span id="zhujie"/>

```properties
@Configuration : 标注在类上.说明这个类是个配置类
@PropertySource : 标注在类上.用于指定配置文件路径,加载配置.通常和@Configuration 一起使用.
@Import : 标注在类上.用于引入其他配置类.通常和@Configuration一起使用
@ComponentScan : 标注在类上.指定扫描包,属性 basePackage:指定包路径.通常和 @Configuration 一起使用,用于纯注解方式.

@Component : 标注在类上.将类交给 Spring 管理,属性 : value,指定 Bean 的 ID,不指定的话默认 ID 为类名,首字母小写
@Controller : 标注在类上.也是将类交给 Spring 管理,用于表现层.属性 : value,指定 Bean 的 ID,不指定的话默认 ID 为类名,首字母小写
@Service :  标注在实现类上.也是将类交给 Spring 管理,用于业务层.属性 : value,指定 Bean 的 ID,不指定的话默认 ID 为类名,首字母小写
@Repository : 标注在实现类上.也是将类交给 Spring 管理,用于持久层.属性 : value,指定 Bean 的 ID,不指定的话默认 ID 为类名,首字母小写
@Scope : 标注在类上.表明这个 Bean 的作用范围(多例,单例..)
@Value : 标注在属性上. 用于注入基本类型和 String 类型数据

@AutoWired : 自动按照类型注入.当接收只有一个实现时,会自动将实现注入,和 Bean ID 没有关系,当有多个时,需要指定 BeanID
@Qualifier : 在@AutoWired 的基础上,按照bean ID 注入
@Resource : 注解按照 Bean ID注入,属性:name,bean 的 ID 

@Bean : 标注在方法上. 将此方法的返回值交给 Spring 管理,属性 name : 指定 BeanID.默认的 BeanID 为方法名.

@ContextConfiguration : 标注在类上,用于加载配置文件.有属性 locations="classpath:xxxxx.xml"

@Transactional(isolation=Isolation.DEFAULT) : 标注在类上或者方法上,说明此类或方法开启事务管理

@Aspect : 标注在类上,说明此类是个切面类
    @Before : 标注在方法上,说明此方法为前置通知
    @After : 标注在方法上,说明此方法为后置通知
    @After-retruning : 标注在方法上,说明此方法是在目标方法执行成功后通知
    @After-throwing : 标注在方法上,说明此方法在目标方法抛出异常后通知
    @Around : 标注在方法上,说明此方法为环绕通知.@Around 标注的通知方法需要有参数 final ProceedingJoinPoint p; p.proceed标识执行切入点方法
    都有属性: value="切入点表达式"
    
    
@within(注解类型) : 一般作用在方法上,匹配具有指定类型注解的方法.比如切面: @Before("@within(org.springframework.stereotype.Controller)"):标识所有具有@Controller 的方法被执行时都会执行通知方法
```

## 配置

```xml
<!--开启扫描包-->
<context:component-scan base-package=""/>
<!--开启对事务注解的支持-->
<tx:annotation-driven transaction-manager="平台事务管理器 BeanID"/>
<!--使用 Spring 默认方式加载 JDBC 配置文件-->
<context:property-placeholder location="classpath:jdbc 配置文件"/>
<!--主配置文件可通过下边的方法引入其他配置文件-->
<import resource="配置文件路径">
```

## 其他知识点
### 使用到的设计模式
- 工厂模式
    - 在控制反转中使用了工厂模式

    
- 代理模式
    - AOP 切面编程.使用了动态代理