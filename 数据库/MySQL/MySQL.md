# MySQL
## 表

```sql
<!--查看所有数据库-->
show databases ;
<!--创建数据库-->
use springboot;
<!--查看所有表-->
show tables;
<!--查看建表语句-->
show create table customer;
<!--如果表不存在就-->
create table if not exists product_category(
	category_id int primary key auto_increment comment '类目ID',
	category_name varchar(64) not null comment '类目名称',
	category_type int not null comment '类目编号' unique ,
	create_time timestamp not null default current_timestamp comment '创建时间',	 -- 设置时间,默认当前时间
	update_time timestamp not null default current_timestamp on update current_timestamp comment '更新时间' -- 默认当前时间,更新时会自动更新
) comment '类目表';作用在数据上
```
作用的数据上,

约束
- unique 约束
    - 作用在数据上,说明此数据在表中是唯一的。
    - UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。
    - PRIMARY KEY 拥有自动定义的 UNIQUE 约束。
    - 请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

创建表时给字段默认值

```sql
<!--默认值 0-->
order_status tinyint(3) not null default '0' comment '订单状态, 默认0: 新下单',  
<!--默认时间为当前时间-->
create_time timestamp not null default current_timestamp comment '创建时间'
```

## 索引
==**语法**==

```sql
CREATE TABLE 表名(
    字段名 数据类型 [完整性约束条件],
                  [UNIQUE | FULLTEXT | SPATIAL] INDEX | KEY
                  [索引名](字段名1 [(长度)] [ASC | DESC])
);
UNIQUE：可选。表示索引为唯一性索引。
FULLTEXT；可选。表示索引为全文索引。
SPATIAL：可选。表示索引为空间索引。
INDEX和KEY：用于指定字段为索引，两者选择其中之一就可以了，作用是一样的。
索引名：可选。给创建的索引取一个新名称。
字段名1：指定索引对应的字段的名称，该字段必须是前面定义好的字段。
长度：可选。指索引的长度，必须是字符串类型才可以使用。
ASC：可选。表示升序排列。
DESC：可选。表示降序排列。
```


## 数据库

```sql
<!--创建库-->
create database 库名;
show databases;  查看所有库
````