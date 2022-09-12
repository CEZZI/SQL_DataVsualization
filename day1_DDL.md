# day1 SQL语句基础

## 1. SQL账号

    - 老师的： -mysql -u guest -h 47.104.31.138 -p 密码： Guest.618
    - 我的： -mysql -u root -p 密码：123456
    - 云端数据库： 47.104.31.138 用户名：guest 密码：Guest.618   
    - 登录： mysql -u root -p
    - 链接本机: mysql -u root -h localhost -p

## 2. SQL语言

#### SQL ---> 结构化查询语言

    ~ DDL(数据定义语言) ---> 创建删除各种对象 ---> create / drop / alter
    ~ DML(数据操作语言) ---> 插入、删除、修改数据 ---> insert / delete / update
    ~ DQL(数据查询语言) ---> 检索（查询）数据 ---> select  
    ~ DCL(数据控制语言) ---> 授予或者召回用户权限 ---> grant / revole

- mysql 连接服务器 mysql -u root -p
- SQL 是不区分大小写大的编程语言 ---> create / CREATE

## 2.1 SQL语言基础

  1. 查看所有数据库 `show`

      show databases;

  2. 创建数据 `create`

      `create database XX default charset utf8mb4;`

      指定默认字符集为 utf8mb4

  3. 删除数据库 `drop XX;`

      `drop database if exists school;`

  4. 查看数据库的过程 `show create`

      `show create database school;`

  5. 切换到指定的数据库 `use`

      `use school;`

      每次新建SQL脚本都需要指定数据库

  6. 显示数据中所有的表`show tables;`

  7. 创建二维表

     **数据类型：**

     - 整数： int / bigint / smallint / tinyint ---> unsigned（无符号整数）
     - 小数：float / double / decimal
     - 时间类型：time / data / datetime / timestamp
     - 字符串: char(10) / varchar(20)
     - 大对象：longtext(字符) / longblob(二进制) --->单列可以方4G

```SQL
CREATE TABLE
[IF NOT EXISTS] tb_name -- 不存在才创建，存在就跳过
(column_name1 data_type1 -- 列名和类型必选
  [ PRIMARY KEY -- 可选的约束，主键
   | FOREIGN KEY -- 外键，引用其他表的键值
   | AUTO_INCREMENT -- 自增ID
   | COMMENT comment -- 列注释（评论）
   | DEFAULT default_value -- 默认值
   | UNIQUE -- 唯一性约束，不允许两条记录该列值相同
   | NOT NULL -- 该列非空
  ], ...
) [CHARACTER SET charset] -- 字符集编码
[COLLATE collate_value] -- 列排序和比较时的规则（是否区分大小写等）
```
#### 例子1

创建一张表，如果已经被创建了正常返回原表

```sql
create table if not exists user_info_vip(
    id int(11) primary key auto_increment comment '自增ID',
    uid int(11) not null unique comment '用户ID',
    nick_name varchar(64) comment '昵称',
    achievement int(11) default 0 comment '成就值',
    level int(11) comment'用户等级',
    job varchar(32) comment'职业方向',
    register_time datetime default current_timestamp comment'注册时间'
 )default charset=utf8;
```

#### 例子2

   ```sql
   create table tbl_student
    (
        stu_id integer not null comment '学号',
        stu_name varchar(20) not null comment '姓名',
        stu_sex boolean default 1 comment '性别',
        stu_birth date comment '出生日期',
        primary key(stu_id)    
    )engine=innodb comment '学生表';

    # 非空约束： not null 
    # default 默认值为1
    # 主键列（primary key）: 能够唯一确定的一条记录的列  
    #  主键可以不加not null 要求 非空且不重复
   ```



  8. 在已有的表中添加、修改或删除列`alter table`
   
   - 多次修改需要写多条`ALTER TABLE`语句，不能合并在一条语句里
  
  ```sql  
alter table user_info add school varchar(15) after level;
-- 增加列在某列之后
alter table 增加的表格 add 增加列的名称 数据类型 位置(after level 在level 之后)

alter table user_info change job profession varchar(10);
-- 更换列的名称及数据类型
alter table user_info change 原列名 修改列名 修改数据类型

alter table user_info modify achievement int(11) default 0;
-- 更改数据类型
alter table 表名 modify 修改列名称 数据类型 默认值等

--例题
alter table tb_student add column cod_id int unsigned not null --加入一列
alter table tb_student constraint fk_student_col_id foreign key (col_id) references tb_college(col_id); 
  ```   

#### 例子3

```sql
alter table user_info add column school varchar(15) after level;
-- 字段level的后面 after 增加一列最多可保存15个汉字的字段school
alter table user_info change job profession varchar(10);
-- 表中job列名改为profession 同时varchar字段长度变为10
alter table user_info modify achievement int(11) default 0;
-- achievement的默认值设置为0
```

## 3. 表之间的关系

- 学生表<-----从属----->学院表

   （多）————————（一）

## 4.附录_代码

```sql
use school;

drop table if exists tb_student;

CREATE TABLE tbl_student (
    stu_id INTEGER NOT NULL COMMENT '学号',
    stu_name VARCHAR(20) NOT NULL COMMENT '姓名',
    stu_sex BOOLEAN DEFAULT 1 COMMENT '性别',
    stu_birth DATE COMMENT '出生日期',
    PRIMARY KEY (stu_id)
)  ENGINE=INNODB COMMENT '学生表';

-- 修改表
-- 添加一个列


alter table tbl_student add column stu_addr varchar(20) default '' comment '籍贯';

-- 删除一个列
alter table tbl_student drop column stu_addr;

-- 修改一个列（修改列的名字） change
alter table tbl_student change column stu_sex stu_gender boolean default 1 comment '性别';

-- 修改一个列（修改列的数据类型）modify
alter table tbl_student modify column stu_gender char(1) default '男';

-- 创建学院表tb_college
create table `tb_college`
(
 `col_id` int unsigned auto_increment comment '编号',
    `col_name` varchar(50) not null comment '名称',
    `col_intro` varchar(50) default '' comment '介绍',
    primary key(col_id)
)engine=InnoDB comment '学院表';

-- 修改学生表添加一个列来保护学生对学院的多对一关系
-- 多对一关系都是在多的一方添加一个列来维护

alter table tbl_student add column col_id int unsigned not null comment '学院编号';

-- 修改学生表添加一个外键约束，限制学生表中的学院编号必须参照学院表的学院编号
-- add constraint 约束 / foreign key 外键 / reference 参考
alter table tbl_student add constraint fk_student_col_id foreign key(col_id) references tb_college(col_id);


-- 老师表 老师属于学院
create table `tb_teachers`
(
 `teach_id` int unsigned auto_increment comment '编号',
    `teach_name` varchar(50) not null comment '名称',
    `teach_course_name` varchar(50) default '' comment '授课名称',
    primary key(teach_id)
)engine=InnoDB comment '老师表';

-- 课程表 老师可以开多个课程
create table `tb_course`
(
 `col_id` int unsigned auto_increment comment '编号',
    `col_name` varchar(50) not null comment '名称',
    `col_intro` varchar(50) default '' comment '介绍',
    primary key(col_id)
)engine=InnoDB comment '学院表';
-- 选课记录表 多对多关系
```
