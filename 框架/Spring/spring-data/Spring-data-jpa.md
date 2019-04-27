# Spring-data-jpa
## 结构

## 框架搭建
### SpringBoot
- ==**引入 POM**==
    
    ```xml
    基于 springboot : 
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <!--通过 MySQL 操作数据库-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    ```
- ==**配置**==

    ```properties
    // 基于 MySQL
    spring.jpa.hibernate.ddl-auto=update
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?characterEncoding=utf-8
    spring.datasource.username=root
    spring.datasource.password=root
    ```

### 1. 使用 ==*CrudRepository*==
- 提供了通用的 CRUD 方法
- 以及==**各种查询各种 by 的功能,除了 ById 之外,其他都需要在自己写的接口内编写方法,但是不用写实现**==
- 使用方法 : 
    - 编写实体类
    - 编写接口继承 ==*CrudRepository*==

- ==**实体类**==

    ```java
    @Entity 
    @Table(name = "customer")
    public class Customer {
    
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO )
        private int id;
        private String username;// varchar(64) NOT NULL COMMENT '用户姓名',
        private String sex;// varchar(32) NOT NULL COMMENT '性别',
        private String birthday;// timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '生日',
        private String address;// varchar(256) NOT NULL COMMENT '用户地址',
        private String cat_id;// int(11) DEFAULT NULL COMMENT '汽车 ID',
        // 省略构造方法以及 get/set 方法
    }
    ```
    
- ==**编写接口,继承 CurdRepository**==

    ```java
    // 不用使用注解, Spring 会自动管理这个接口
    // 不用使用注解, Spring 会自动管理这个接口
// 第一个参数是要操作的表对应的实体类. 第二个参数是 ID
    public interface CustomerRepository extends CrudRepository<Customer, Integer> {
    
        // CrudRepository 内虽然没有这个方法,但是会自动去按照 name 查所有,不需要写实现
        public List<Customer> findAllByUsername(String username);
    
    }
    ```
    
- ==**编写Controller**==

    ```java
    @Slf4j
    @Controller
    @RequestMapping("/example")
    public class MyController {
    
        @Autowired
        private CustomerRepository customerRepository;
    
        @GetMapping("/add")
        public @ResponseBody Object add(@RequestParam String username, @RequestParam String sex){
            Customer customer = new Customer();
            customer.setUsername(username);
            customer.setSex(sex);
            customer.setAddress("beijing");
            customer.setBirthday("2018/1/1");
            customer.setCat_id("1");
            log.info("=============================>进行保存");
            customerRepository.save(customer);
            return customer;
        }
    
        @GetMapping("find")
        public @ResponseBody Object find(@RequestParam Integer id){
            log.info("url 参数为 : id = " + id);
            Optional<Customer> customer = customerRepository.findById(id);
            log.info("query result : " + customer);
    
            return customer;
        }
    
        @GetMapping("/findAll")
        public @ResponseBody Object findAll(){
            Iterable<Customer> all = customerRepository.findAll();
            return all;
        }
    
        @GetMapping("/count")
        public @ResponseBody Object count(){
            long count = customerRepository.count();
            return "there is " + count + " pieces of data";
        }
    
        @GetMapping("/findByName")
        public @ResponseBody Object findByName(@RequestParam String username){
            List<Customer> customerList = customerRepository.findAllByUsername(username);
            return customerList;
        }
    }
    ```
    
    
### 2. ==**PagingAndSortingRepository**==
- 提供 ==**分页查询,排序查询,分页+排序查询等**==
- 实现方式 : 
    - 编写实体类
    - 编写接口实现 ==*PagingAndSortingRepository*==
    - 需要 ==*Sort(实现排序)*== 以及 ==*PageRequest(实现分页)*==

- ==*接口*==

    ```java
    // 不用使用注解,Spring 会自动去管理这个类
    // 不用使用注解, Spring 会自动管理这个接口
// 第一个参数是要操作的表对应的实体类. 第二个参数是 ID
    public interface CustomerPageRepository extends PagingAndSortingRepository<Customer, Integer> {
        // pageable 为分页参数
        public Page<Customer> findCustomerByUsername(String username, Pageable pageable);
    }
    ```

- ==*controller*==

```java
@Controller
@RequestMapping("/page")
public class PageController {

    @Autowired
    private CustomerPageRepository customerRepository;

    @GetMapping("/find")
    public @ResponseBody Object findOne(@RequestParam("page") int page, @RequestParam("size") int size){
        // sort : 排序参数, 这里是按照 id 降序排列
        Sort sort = new Sort(Sort.Direction.DESC, "id");
        // pageRequest : 设置分页信息, 页码, 每页数据条数, 排序参数. 页码从 0 开始
        PageRequest pageRequest = new PageRequest(page, size, sort);
        //进行查询
        Page<Customer> customers = customerRepository.findAll(pageRequest);
        return customers;
    }

    @GetMapping("/findByName")
    public @ResponseBody Object findByName(@RequestParam("page") int page, @RequestParam("size") int size, @RequestParam String username){
        // 按照名称查找, 以及 ID 降序 排列, page 从 0 开始
        Page<Customer> customer = customerRepository.findCustomerByUsername(username, new PageRequest(page, size, new Sort(Sort.Direction.DESC, "id")));
        return customer;
    }
}
```


### 3. ==**JpaRepository**==
- 以上的 CrudRepository 和 PagingAndSoringRepository 是spring data 为了兼容 NoSQL 进行的一些抽象封装.
- 从 JPARepository 往下是对关系型数据库进行的抽象封装
- JpaRepository 继承 PagingAndSortingRepository,也就继承了它的所有方法.

- ==*接口*==

    ```java
    /**
     * @program: example
     * @description: JpaRepository 继承了 PagingAndSortingRepository, 有它的所有方法,所以可以实现分页,排序等功能
     * @author: cy
     * @create: 2019-04-11 17:57
     **/
    public interface CustomerJpaRepository extends JpaRepository<Customer, Integer> {
    
        public List<Customer> findCustomerByUsername(String username, Pageable pageable);
    }
    ```

- ==*controller*==

    ```java
    @Controller
    @RequestMapping("/jpsR")
    public class CusJpaController {
    
        @Autowired
        private CustomerJpaRepository customerJpaRepository;
    
        @GetMapping("/findByUsername")
        public @ResponseBody Object findByUsername(@RequestParam String username,
                                                   @RequestParam int page,
                                                   @RequestParam int size){
    
            // 根据名称查询,用按照 ID 降序, 按照分页输出
            List<Customer> id = customerJpaRepository.findCustomerByUsername(username, 
                    new PageRequest(page, size, 
                            new Sort(Sort.Direction.DESC, "id")));
            return id;
        }
    }
    ```
    
## 2. 定义查询方法
- Spring JPA Repository 的实现原理是采用动态代理的机制
- 内部基础架构中有个根据方法名的查询生成器机制，对于在存储库的实体上构建约束查询很有用，该机制方法的前缀 find…By、read…By、query…By、count…By 和 get…By 从所述方法和开始分析它的其余部分（实体里面的字段）。
- ==*org.springframework.data.repository.query.parser.PartTree*== 中配置了规则
- Spring JPA 里面是将下划线视为保留字符，所以如果

### 返回实体类中指定的属性
- 比如 Customer 中,有 5 个属性,但是只想返回其中的 2 个属性.
- 创建一个接口 A ,此接口中编写对应属性的 get 方法
- 自定义的 Repository 中创建查询方法, 返回值类型是 接口 A 类型

- ==*实体类*==

    ```java
    @Entity
    @Table(name = "customer")
    public class Customer {
    
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO )
        private int id;
        private String username;// varchar(64) NOT NULL COMMENT '用户姓名',
        private String sex;// varchar(32) NOT NULL COMMENT '性别',
        private String birthday;// timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '生日',
        private String address;// varchar(256) NOT NULL COMMENT '用户地址',
        private String cat_id;// int(11) DEFAULT NULL COMMENT '汽车 ID',
        //为了篇幅,省略 set/get 方法以及构造函数
    }
    ```

- ==*编写接口,用于指定要返回的指定的Customer属性*==

    ```java
    /**
     * @program: example
     * @description: 只返回 Customer 中的 username 属性,编写指定属性的 get 方法
     * @author: cy
     * @create: 2019-04-11 19:40
     **/
    public interface UsernameOnly {
        String getUsername();
        String getSex();
    }
    ```
    
- ==*CustomerPageRepository*==

    ```java
    // 不用使用注解,Spring 会自动去管理这个类
    public interface CustomerPageRepository extends PagingAndSortingRepository<Customer, Integer> {
        /**
         * 返回值类型为 UsernameOnly, 以此来返回 Customer 中指定的属性
         * @param id
         * @return
         */
        public List<UsernameOnly> findCustomerById(int id);
    }
    ```

- ==*Controller*==

    ```java
    @Controller
    @RequestMapping("/page")
    public class PageController {
    
        @Autowired
        private CustomerPageRepository customerRepository;
    
        //返回Customer指定的属性
        @GetMapping("/only")
        public @ResponseBody Object findOnlyUsername(@RequestParam int id){
            List<UsernameOnly> customerById = customerRepository.findCustomerById(id);
            return customerById;
        }
    }
    ```

## 注解方式
- @Query

### 1. @Query
==*源码及语法*==

```java
public @interface Query {
   /**
    * 指定JPQL的查询语句。（nativeQuery=true的时候，是原生的Sql语句）
    */
   String value() default "";
   /**
    * 指定count的JPQL语句，如果不指定将根据query自动生成。
    * （如果当nativeQuery=true的时候，指的是原生的Sql语句）
    */
   String countQuery() default "";
   /**
    * 根据哪个字段来count，一般默认即可。
    */
   String countProjection() default "";
   /**
    * 默认是false，表示value里面是不是原生的sql语句
    */
   boolean nativeQuery() default false;
   /**
    * 可以指定一个query的名字，必须唯一的。
    * 如果不指定，默认的生成规则是：
    * {$domainClass}.${queryMethodName}
    */
   String name() default "";
   /*
    * 可以指定一个count的query的名字，必须唯一的。
    * 如果不指定，默认的生成规则是：
    * {$domainClass}.${queryMethodName}.count
    */
   String countName() default "";
}
```

- ==*

