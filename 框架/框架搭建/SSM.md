# SSM 框架搭建
==**applicationContext.xml**==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context
	http://www.springframework.org/schema/context/spring-context.xsd
	http://www.springframework.org/schema/aop
	http://www.springframework.org/schema/aop/spring-aop.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx.xsd
	http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc.xsd">


    <!--开启注解扫描,指定包结构,默认扫描该包下所有的类,哪个类上有注解,就将此类交给IOC管理
            需要导入约束:xmlns:context="http://www.springframework.org/schema/context"
            component-scan:组件扫描
                base-package:要扫描的包,扫描此包下所有的类,哪个类上有注解,就将此类交给IOC
         -->
    <context:component-scan base-package="com.cy"></context:component-scan>

    <!--开启Spring对AOP注解的支持
        proxy-target-class:配置生成代理方式
            true:cglib方式
            false:jdk方式,默认方式
    -->
    <aop:aspectj-autoproxy />

    <!-- 加载配置JDBC文件 -->
    <context:property-placeholder location="classpath:properties/db.properties" />
    <!-- 数据源 -->
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="maxActive" value="10" />
        <property name="maxIdle" value="5" />
    </bean>

    <!--事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <!-- 使用全注释事务 -->
    <tx:annotation-driven transaction-manager="transactionManager" />


<!--  mybatis 配置 start -->
    <!-- 在使用mybatis时 spring使用sqlsessionFactoryBean 来管理mybatis的sqlsessionFactory -->
    <bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean" scope="singleton">
        <!--指定mybatis的全局配置文件-->
        <property name="configLocation" value=""></property>
        <!-- 实体类映射文件路径,这里只有一个就写死了，多个可以使用mybatis/*.xml来替代 -->
        <property name="mapperLocations" value="classpath:mapper/*.xml"></property>
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--动态代理实现 不用写dao的实现 ,配置扫描器 将mybatis接口实现加入到ioc容器中-->
    <bean id="MapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 这里的basePackage 指定了dao层接口路劲，这里的dao接口不用自己实现 -->
        <property name="basePackage" value="com.cy.dao" />
        <!-- 如果只有一个数据源的话可以不用指定，但是如果有多个数据源的话必须要指定 -->
        <property name="sqlSessionFactoryBeanName" value="sessionFactory" />
        <!--直接制定了sqlsessionTemplate名称，这个和上面的其实是一样的 -->
        <!-- <property name="sqlSessionTemplateBeanName" value="sqlSession" /> -->
    </bean>
<!--  mybatis 配置 end -->

</beans>
```