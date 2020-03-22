#### MySQL服务器的连接和退出

- 连接：

  - 完整格式：mysql -h主机地址 -P端口号 -u用户名 -p用户密码
  - 常用简写：mysql -h主机地址 -u用户名 -p（密码使用暗文形式）

- 退出

  - ```mysql
    exit;
    ```

  - ```mysql
    quit
    ```

  - ```mysql
    \q
    ```

#### 查看数据库的编码

- 查看数据库全局默认的编码

  - ```mysql
    show variables like 'character_set_%';
    ```

- 查看某个数据库的编码

  - show create database 数据库名;

#### 数据库的增删查改

- 显示数据库

  - 查看指定数据库

    - show create database 数据库名称;

  - 查看所有数据库

    - ```mysql
      show databases;
      ```

  - 默认有4个数据库

    1. information_schema：保存着关于MySQL服务器所维护的所有其他数据库的信息，如数据库名，数据库的表，表栏的数据类型与访问权限等
    2. mysql：MySQL系统数据库，保存了登录用户名，密码，以及每个用户的权限等
    3. performance_schema：用来保存数据库服务器性能的参数
    4. sys：这个库是通过视图的形式把information_schema和performance_schema结合起来，查询出更加令人容易理解的数据

- 创建数据库

  - create database if not exists 数据库名称 charset=utf8;

- 删除数据库

  - drop database if exists 数据库名称;

- 修改数据库

  - 只能修改数据库的字符集
    - alter database 数据库名称 charset=字符集;

#### 表的增删查改

- 注意点：在对表进行操作的时候，必须指定某一个数据库

  - use 数据库名称;

- 查看表

  - 查看数据库中有哪些表

    - ```mysql
      show tables;
      ```

  - 查看指定表的结构

    - desc 表的名称;

- 创建表

  - ```mysql
    create table if not exists stu(
    	id int, #表的名称 表中数据的数据类型,
    	name text #表的名称 表中数据的数据类型
    )
    ```

- 删除表

  - drop table if exists 表名;

- 修改表

  - 修改表名
    - rename table 旧名称 to 新名称;
  - 添加字段
    - alter table 表名 add 新增字段名称 新增字段数据类型 [位置]
    - 默认新增的字段会放到原有字段的后面
    - alter table stu add age int;
    - 放到最前面：alter table stu add age int first;
    - 放到某个字段后面：alter table stu add age int after name;
  - 删除字段
    - alter table 表名 drop 字段名称;
  - 修改字段
    - 修改字段数据类型
      - alter table 表名 modify 需要修改的字段名称 新的数据类型;
    - 修改字段的名称和数据类型
      - alter table 表名 change 原始字段名称 新的字段名称 新的数据类型;

#### 表的存储引擎

- MySQL中有三种存储引擎

  1. MyISAM：安全性低，不支持事务和外键，适合频繁插入和查询的应用
  2. InnoDB（默认）：安全性高，支持事务和外键，适合对安全性，数据完整性要求较高的应用
  3. Memory：数据存储到内存中，访问速度极快，但不会永久存储数据，适合对读写速度要求极高的应用

- 创建表的时候指定存储引擎

  - ```mysql
    create table stu(
    	id int,
    	name text
    )engine=引擎名称;
    ```

- 修改表的存储引擎

  - alter table 表名 engine=引擎名称;

#### 数据的增删改查

- 插入数据

  - insert into 表名 (字段名称1, 字段名称2) values (值1, 值2);

  - ```mysql
    insert into stu (id, name) values (1, 'qq');
    ```

  - 注意点：

    1. 插入数据的时候，指定字段名称顺序可以和表中的字段顺序不一样
    2. 插入数据的时候，指定取值的顺序必须和指定字段名称顺序一样
    3. 插入数据的时候，如果指定取值的顺序和表中的字段顺序一样，那么可以不指定字段名称
    4. 可以同时插入多条数据，一条数据一个括号，中间逗号隔开

- 查询数据

  - 查询某个表中所有字段的数据
    - select * from stu;
  - 查询特定字段的数据
    - select 字段名称1 from 表名;
  - 查询某个表中满足条件的数据
    - select 字段名称1, 字段名称2 from 表名 [where 条件];

- 更新数据

  - update 表名 set 字段名称=值 [where 条件];

  - 注意点：

    - 不指定条件时，整个表的数据都会被修改
    - 可以指定多个条件，支持AND OR = != 等 [支持的条件](https://www.runoob.com/mysql/mysql-where-clause.html)

  - ```mysql
    update stu set score = 77 where name = 'qq' AND id = 1;
    ```

- 删除数据
  - delete from 表名 [where 条件];
  - delete from 表名;

#### MySQL数据类型

- MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型。
- [菜鸟文档](https://www.runoob.com/mysql/mysql-data-types.html)
- [详细分类视频](https://www.it666.com/course/118/task/7971/show)