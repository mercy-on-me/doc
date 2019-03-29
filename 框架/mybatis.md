# Mybatis
## 原理
- 通过 xml 配置文件将要执行的各种 statement 配置起来,并通过 java 对象和 statement 中的SQL 进行映射生成最终的 SQL 语句,最后由 mybatis 框架执行 SQL 并将结果映射成 java 对象并返回


## mybatis 架构



## 急速入门
- 编写核心配置文件
- 编写 POJO,Dao,Mapper 文件


## mapper 文件编写

### resultMap

```xml
<resultMap id="userResultMap" type="pojo.User">
    <id column="id" property="id"/>
    <result column="username" property="username"/>
    <result column="sex" property="sex"/>
    <result column="birthday" property="birthday"/>
    <result column="address" property="address_1"/>
</resultMap>
```

### 动态 SQL
- ==**if**==
    
    ```xml
    <if test="xx != nlll and xx != ''">
     and xxxx
    </if>
    ```

- ==**where**==
    - 使用 if 标签时,需要使用 where 1 = 1 的形式,太麻烦,where 解决此问题
    
    ```xml
    <where>
        <if test="xx != nlll and xx != ''">
            and xxxx
        </if>
    </where>
    
    等价于 
    
    where 1 = 1
    <if test="xx != nlll and xx != ''">
        and xxxx
    </if>
    ```

- ==**sql**==
    - 将重复的 SQL 提取出来,比如字段
    
```xml
<!--定义-->
<sql id="userFeilds">
    id,
    username,
    sex,
    birthday,
    address
</sql>

<!--使用-->
<include refid="userFeilds"/>
```

### 1 对 1(比如订单信息内包含客户信息)
- 除了关联查询之外还可以使用 resultMap

```xml
<resultMap id="userResultMap" type="pojo.User">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <result column="birthday" property="birthday"/>
        <result column="address" property="address_1"/>
        <!--
            association : 设置一对一关系,比如 order 内有 user 属性
            property : order 内的 User 对象的属性名
            javaType : 属性类型
        -->
        <association property="cat" javaType="pojo.Cat">
            <!--
                id ; 声明主键,标识唯一的关联关系
            -->
            <id property="id" column="cat_id"/>
            <result property="name" column="catname"/>
        </association>
 </resultMap>
```

## 配置文件解析
- 主要对象:

```properties
sqlsession : mybatis 的核心对象.封装了框架对数据库的所有操作,通过 SQLSessionFactory 构建.
SQLSessionFactory : 用于构建 SQLSession 对象.此对象通过SQLSessionFactory的 build 方法构建
SqlSessionFactoryBuilder : 用于构建 SQLSessionFactory
XMlConfigBuilder : 用于解析配置文件,生成 Configuration 对象.是 mybatis 的基础,包含了mybatis 框架的所有配置
Configuration
```
- 构建 SQLSession
    - mybatis 的核心对象是sqlsession
    - SQLSession 对象通过 SqlSessionFactory对象构建,SqlSessionFactory对象通过 SqlSessionFactoryBuilder 对象的 buid 方法构建.
    - buid 方法内通过 XMLConfigBuilder 对象解析配置文件,生成 Mybatis 的配置文件 configuration.

    
```java
// 加载配置文件
InputStream resourceAsStream = SqlSessionFactory.class.getClassLoader().getResourceAsStream("");
// 构建 SQLSessionFactory 对象
SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsStream);
//构建 Sqllsession对象
SqlSession sqlSession = build.openSession();

// build 方法
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```

### 1. 解析 properties 配置

```java
// 解析properties配置
private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
        Properties defaults = context.getChildrenAsProperties();
        String resource = context.getStringAttribute("resource");
        String url = context.getStringAttribute("url");
        if (resource != null && url != null) { // resource和url只能同时配置一个
            throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
        }
        if (resource != null) {
            defaults.putAll(Resources.getResourceAsProperties(resource));
        } else if (url != null) {
            defaults.putAll(Resources.getUrlAsProperties(url));
        }
        Properties vars = configuration.getVariables();
        if (vars != null) {
            defaults.putAll(vars);
        }
        configuration.setVariables(defaults);        // 将属性设置到Configuration对象
        parser.setVariables(defaults);
    }
}
```
- properties 标签不能同时配置 resource 和 URL,并且 properties 下的 property 标签会覆盖 resource 和 URL.

### 2. 解析 typeAliases配置 (别名)
