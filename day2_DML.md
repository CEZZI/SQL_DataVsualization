# day 2 DML(数据操作语言)

## 1. ER图概述

    -ER图 ---> ENtity Relationship Diagram
      实体：矩形框 ---> 表
      属性：椭圆框 ---> 列（字段、属性、特征）
      关系: 菱形框 
      重数: 一对一(1:1)、一对多(1:n)、多对多(n:n)

    - EER图 ---> EXtended ER图
  
    正向工程: 先设计EER图，然后根据EER图生成数据库和表
    反向工程: 用设计好的数据课的表生成EER图
![EER图](https://camo.githubusercontent.com/574a0428459c6b7b437417542b39a57ac61b2c8f123f359f4ad8e5977bc8daa3/68747470733a2f2f67697465652e636f6d2f6a61636b66727565642f6d797069632f7261772f6d61737465722f32303231313130363036333933392e706e67)

## 2. 使用Workbench绘制EER图

## 3. 数据库的正向工程和反向工程

### 3.1 正向工程

---> database ---> Forward Engineer

``- skip creation of FOREIGN`` 不需要生成外键约束

外键约束会影响数据的传输效率 很多互联网公司要求 可以有多对多的关系 但是不要外键约束

### 3.2 反向工程

---> database ---> reverse Engineer
把代码变成工程

```SQL
-- 如果存在名为school的数据库就删除它
drop database if exists `school`;
-- 创建名为school的数据库并设置默认的字符集和排序方式
create database `school` default charset utf8mb4;
-- 切换到schooL数据库上下文环境
use `school`;

-- 创建学院表

create table `tb_college`
(
`col_id` int unsigned auto_increment comment'编号',
`col_name` varchar(50) not null comment '名称',
`col_intro` varchar(5080) default '' comment'介绍',
primary key (`col_id`)
) engine=innodb comment'学院表';


-- 创建学生表

create table `tb_student`(
`stu_id` int unsigned not null comment '学号',
`stu_name` Varchar(20) not null comment'姓名',
`stu_sex` boolean default 1 comment '性别',
`stu_birth` date not null comment'出生日期',
`stu_add` varchar(255) default '' comment'籍贯',
`col_id` int unsigned not null comment'所属学院',
primary key (`stu_id`),
foreign key (`col_id`) references `tb_college` (`col_id`)
)engine=innodb comment'学生表';


-- 创建教师表
create table `tb_teacher`
(
`tea_id` int unsigned not null comment '工号',
`tea_name` varchar(20) not null comment '姓名',
`tea_title` varchar(10) default '助教'comment'职称',
`col_id` int unsigned not null comment '所属学院',
primary key (`tea_id`),
foreign key (`col_id`) references `tb_college`(`col_id`)
) engine=innodb comment '老师表';



-- 创建课程表
create table `tb_course`
(
`cou_id` int unsigned not null comment '编号',
`cou_name` varchar(50) not null comment '名称',
`cou_credit` int unsigned not null comment'学分',
`tea_id` int unsigned not null comment'授课老师',
primary key (`cou_id`),
foreign key (`tea_id`) references `tb_teacher` (`tea_id`)
)engine=innodb comment '课程表';


--  创建选课记录表
create table `tb_record`
(
`rec_id` bigint unsigned auto_increment comment '选课记录号',
`sid` int unsigned not null comment'学号',
`cid` int unsigned not null comment '课程编号',
`sel_date` date not null comment'选课日期',
`score` decimal(4,1) comment '考试成绩',
primary key (`rec_id`),
foreign key (`sid`) references `tb_student` (`stu_id`),
foreign key  (`cid`) references `tb_course` (`cou_id`),
unique (`sid`,`cid`)
) engine=innodb comment'选课记录表';
```

## 4. DML(数据操作语言)

### 4.1 插入数据

`insert into 表 values();`

- 普通插入（全字段）：
  
  `INSERT INTO table_name VALUES -(value1, value2, ...)`
- 普通插入（限定字段）：

 `INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...)`

- 多条一次性插入：

```sql
INSERT INTO table_name (column1, column2, ...) VALUES 
(value1_1, value1_2, ...), 
(value2_1, value2_2, ...), ...
```

- 从另一个表导入：
  
  `INSERT INTO table_name SELECT * FROM table_name2 [WHERE key=value]`
  
- 插入主键已经出现的数据:`REPLACE INTO`

replace into 跟 insert into功能类似，不同点在于：replace into 首先尝试插入数据到表中，如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据；否则，直接插入新数据。

要注意的是：插入数据的表必须有主键或者是唯一索引！
否则的话，replace into 会直接插入数据，这将导致表中出现重复的数据。

```sql
REPLACE INTO examination_info
VALUES(NULL,9003,'SQL','hard',90,'2021-01-01 00:00:00');
```
  
```SQL
-- 我们已经创建了一张新表exam_record_before_2021用来备份2021年之前的试题作答记录，结构和exam_record表一致，请将2021年之前的已完成了的试题作答纪录导入到该表。
INSERT INTO exam_record_before_2021(uid, exam_id, start_time, submit_time, score)
SELECT uid, exam_id, start_time, submit_time, score
FROM exam_record
WHERE YEAR(submit_time) < '2021';

```

```sql
use school;

-- 如果不指定给哪些列赋值，就得给按照表的顺序全部赋值
insert into tb_college values(default,'数学学院','学习很难的该死的数学');
-- 三元组 编号默认，学院名称，简介

-- 给指定的列赋值 （没有赋值的列要么允许为空，要么有默认值）
insert into tb_college(col_name,col_intro) values('金研院','中泰金融');
-- 不给学院的编号 编号为默认值
-- Error Code: 1136. Column count doesn't match value count at row 1 

-- 批量插入——同时加入三条数据
insert into tb_college(col_name,col_intro) values
    ('经管学院','fagdgfeee'),
    ('化学学院','ffadfa'),
    ('物理学院','sddfad');
    
    
-- 批量添加学生
insert into tb_student(stu_id,stu_name,stu_sex,col_id) values
    (1,'caozi','0',1),
    (2,'caoziwen','0',2),
    (3,'caozi1','0',3),
    (4,'caoen','0',4),
    (5,'caoyuanming','1',1);
    
    
-- Error Code: 1062. Duplicate entry '1' for key 'tb_student.PRIMARY‘
-- 重复的条目 学号要求不相同 ---> 学号为主键

insert into tb_student(stu_id,stu_name,stu_sex,col_id) values
 (7,'caozi','0',15);
-- Error Code: 1064. You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '' at line 2
```

### 4.2 删除数据

`delete from 表 where 条件` `truancate table 表名`

- insert / delete / updata
- 删除一定要带条件！！
- 如果无条件删除，就把整张表都删除了  `delete from tb_college;`
- 删除语句中也可以写 order by / limit
- delete与truancate

truncate table 在功能上，与不带where字句的delete语句相同；二者均删除表中的全部行，但truncate table 比delete速度更快，且使用的系统和事务日志资源少。 truncate 删除表中的所有行，但表的结构及其列，约束，索引等保持不变。新行标识所用的技术值重置为该列的种子。如果想保留标识计数值，轻盖拥delete 。如果要删除表定义及其数据，请使用drop table 语句。

- 根据条件删除：DELETE FROM tb_name [WHERE options] [ [ ORDER BY fields ] LIMIT n ]
- 全部删除（表清空，包含自增计数器重置）：TRUNCATE tb_name

##### 例题

```SQL
请删除exam_record表中未完成作答或作答时间小于5分钟整的记录中，开始作答时间最早的3条记录。
delete from exam_record
where 
    submit_time is null or timestampdiff(minute,start_time,submit_time) < 5
order by start_time
limit 3;
```

```SQL
use school;
delete from tb_college where col_id = 4;
-- in 成员运算 删掉 5 6 行

delete from tb_college where col_id=1; 
--不让删除 这个学院有学生 违反了外键约束
-- Error Code: 1451. Cannot delete or update a parent row: a foreign key constraint fails (`school`.`tb_student`, CONSTRAINT `tb_student_ibfk_1` FOREIGN KEY (`col_id`) REFERENCES `tb_college` (`col_id`)) 

-- 删除表  truncate 删除的数据 不能恢复
truncate table tb_college;
```

```sql
-- 删除重复的邮箱
-- 建立两个表！
delete  p1 from Person p1, Person p2 
where p1.email = p2.email and p1.id > p2.id 
```

### 4.3 更新数据

`update 表 set 更新的内容 where 条件；`

- 在SQL语句中,等号`=`代表相等判断。如果要赋值的话，需要先写上`set`
- `set` 后面的等号 赋值
- 没有写条件的话 就是 全表更新
- between 1 and 5 ☞在1和5之间

```SQL
use school;

update tb_student set stu_add='湖南湘潭'where stu_id in (1,3);

update tb_student set stu_birth='2000-1-9',stu_add='湖南湘潭'
where stu_id between 6 and 7;
```

```SQL
-- 男女性别转换
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;

作者：LeetCode
链接：https://leetcode.cn/problems/swap-salary/solution/jiao-huan-gong-zi-by-leetcode/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
#
