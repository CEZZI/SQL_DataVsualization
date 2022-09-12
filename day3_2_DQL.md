# day3.2 DQL语句与窗口函数

## 1. 知识点合计
  
### 1.多表链接

- 内连接：`inner join 表 on 条件`
- 内连接:（inner） join 只有满足的条件才会查出来 ---> 无法查询到未选课的同学
- 外连接：左外连接left（outer）join、右外连接、全外连接（full join）（MySQL不支持）
  - 左表：写join前的表
  - 右表： 写在join后面的表
  - 左外连接left (outer) join：把不满足连表条件的左表记录也能完成的查询出来，不满足条件地方填充null


![join](https://camo.githubusercontent.com/a9a4db5e02205f8f8ac3f7018018e820bbd0c8a1489ffd96f897abf1dd3bd7ab/68747470733a2f2f67697465652e636f6d2f6a61636b66727565642f6d797069632f7261772f6d61737465722f32303231313132313133353131372e706e67)

- `join on 条件` 常见的的是 列名相等 t1.id = t2.id --> using(id)
  - 要记住 后面是接的条件
  
    ```sql
    select w1.id as Id
    from weather w1 join weather w2 on  datediff(w1.recordDate,w2.recordDate) = 1
    and w1.Temperature > w2.Temperature;
    ```

- 数据格式转换`cast(字段名 AS 数据类型)`
- 嵌套查询
- 成员运算 in
- ifnull(a,b) 如果a为null则填充为b
- limit 和 offset
  - limit 3 限制前三条记录
  - limit 3 offset 6 跳过1-6条拿后面三条的记录 即789条件
  - limit 5,3 = limit 3 offset 5 ---> 跳过前5个拿3个调剂

**五种常用的窗口函数**

- `rank() over()` 1 2 2 4 4 6  (计数排名，跳过相同的几个，eg.没有3没有5)\
- `row_number() over()` 1 2 3 4 5 6 (赋予唯一排名)
- `dense_rank() over()` 1 2 2 3 3 4 (不跳过排名，可以理解为对类别进行计数)
- `percent_rank() over()` 按照数字所在的位置进行百分位分段
- `ntile(n) over()` 将数字按照大小平均分成n段
- `lead(字段名，n) over()`把字段数据向前移n个单元格
- `lag(字段名，n) over()`把字段数据向后移n个单元格

```SQL
-- 1.查询每个学生的学号和平均成绩(分组和聚合函数)
select sid,round(avg(score),2) from tb_record group by sid;


-- 2.查询平均成绩大于等于90分的学生的学号和平均成绩
-- 分组以前的数据筛选使用where子句，分组以后的数据筛选使用having子句
select
 sid,
    round((score),2) as 平均分 
from tb_record 
group by sid having 平均分 >=90;

-- 3.查询年龄最大(条件)的学生的姓名(子查询 / 嵌套查询)
-- 使用两次查询 
-- 嵌套查询：一个查询的结果作为另一个查询的条件
select stu_name 
from tb_student 
where stu_birth =
(
 select min(stu_birth) from tb_student
);

-- 4.查询年龄最大(条件)的学生姓名和年龄(子查询+运算)
select 
    stu_name, 
    floor(datediff(curdate(),stu_birth)/365) as 年龄
from tb_student 
where stu_birth=
(
 select min(stu_birth) from tb_student
    );
    
-- 5.查询选了两门以上的课程的学生姓名(子查询/分组条件/集合运算)
-- 不能用等号 要用成员运算 in
select stu_name from tb_student where stu_id in(
 select sid from tb_record group by sid having count(*) >2
);



-- 6.查询课程的名称、学分（课程表）和授课老师（老师表）的名字
-- 直接查询两张表 得到的是两个集合的笛卡尔积
select cou_name,cou_credit,tea_name
from tb_course,tb_teacher
where tb_course.tea_id = tb_teacher.tea_id; -- 课程表的老师编号和老师表的老师编号能对上才行 
-- 法二 inner join 内连接
-- 给表别名不建议写as
select cou_name,cou_credit,tea_name from tb_course t1
inner join tb_teacher t2 on t1.tea_id=t2.tea_id;


-- 7.查询学生姓名(学生表)、课程名称（课程表）以及成绩(连接查询)
-- 越小的表 越靠右写 小表靠右（驱动表） 大表靠左
-- 学号和sid连上 课程号和cid连上
select stu_name,cou_name,score
from tb_student,tb_course,tb_record
where stu_id= sid and cou_id= cid and score is not null;
-- 法二 inner joint 表 on 条件
select stu_name,cou_name,score from tb_student
inner join tb_record on stu_id=sid
inner join tb_course on cou_id=cid
where score is not null;

-- 8.查询选课学生的姓名和平均成绩(子查询和连接查询)
-- 分组聚合建立了一个临时表 一定要给临时表别名
SELECT 
    stu_name, avg_score
FROM
    tb_student,
    (SELECT 
        sid, ROUND(AVG(score), 1) AS avg_score
    FROM
        tb_record
    GROUP BY sid) tb_temp
WHERE
    stu_id = sid;

-- 9.查询 每个 学生的姓名和选课数量(左外连接和子查询)
-- 没有选课的学生也要调出来
-- 内连接:（inner） join 只有满足的条件才会查出来 ---> 无法查询到未选课的同学
-- 外连接：左外连接left（outer）join、右外连接、全外连接（MySQL不支持）
-- 左表：写join前的表称 ---> tb_student/ 右表： 写在join后面的表  ---> tb_temp
    -- 左外连接：把不满足连表条件的记录也能完成的查询出来，不满足条件地方填充null
-- ifnull(a,b) 如果a为null则填充为b

select
stu_name as 姓名,
ifnull(total,0) as 选课数量 
from tb_student left outer join(
select sid, count(*) as total from tb_record group by sid) tb_temp
on stu_id=sid
order by 选课数量 desc limit 3 offset 6; 
```

## 2. MySQL练习

### 2.1 知识点合集

- any 和all
- `in` `not in` 成员运算 性能不是很好
- `distinct` 效率很低
- 尽量少用`in / not in / distinct`操作
- 可以考虑利用存在性判断`exists / not exists` 替代集合元素和去重操作
- `select 'x' (from dual)` 查找常量   x写成y也行，写成啥都行
  - from dual ---> 伪表可省略

- 要查很后面的列
  - 先把主键eno拿出来 主键的速度很快 然后用主键做条件

```SQL
select ..., ... , from ... where eno=  
(select eno from ..., order by ...limit 1 offset 1000000);
```

数据量很大的话就要分表分库
cobar ---> MyCat ---> 数据库中间件

    - 连接池和链接监控
    - 分表分库支持
    - 负载均衡
  
## 2.2 练习

1. 查询月薪最高的员工姓名和月薪(子查询)

```SQL
use hrs;

SELECT 
    ename, sal
FROM
    tb_emp
WHERE
    sal = (SELECT 
            MAX(sal)
        FROM
            tb_emp);
-- 法二 all()集合  不让使用orderby 和 聚合函数的情况
SELECT 
    ename, sal
FROM
    tb_emp
WHERE
    sal >= ALL (SELECT 
            sal
        FROM
            tb_emp);

```

2. 查询员工的姓名和年薪((月薪+补贴)*13)

注意：必须对null进行处理，由于null计算后也为空值，故年薪也会变成null

```sql
SELECT 
    ename, (sal + IFNULL(comm, 0)) * 13 AS ann_sal
FROM
    tb_emp
ORDER BY ann_sal DESC;
```

3. 查询有员工的部门的编号和人数

4. 查询所有部门的名称和人数

```sql
-- 查询有员工的部门的编号和人数
select dno,count(*) as 人数 from tb_emp group by dno;

-- 查询所有部门的名称和人数

SELECT 
    dname, IFNULL(count1, 0)
FROM
    tb_dept
        LEFT JOIN
    (SELECT 
        dno, COUNT(*) AS count1
    FROM
        tb_emp
    GROUP BY dno) tb_temp ON tb_temp.dno = tb_dept.dno;
```

5. 查询月薪最高的员工(Boss除外)的姓名和月薪

```sql
SELECT 
    ename, sal
FROM
    tb_emp
ORDER BY sal DESC
LIMIT 1 OFFSET 1;
-- 法二
SELECT 
    ename, sal
FROM
    tb_emp
WHERE
    sal = (SELECT 
            MAX(sal)
        FROM
            tb_emp
        WHERE
            mgr IS NOT NULL);
```

6. 查询月薪超过平均月薪的员工的姓名和月薪

7. 查询月薪超过其所在部门平均月薪的员工的姓名、部门编号和月薪

```sql
-- 查询月薪超过其所在部门平均月薪的员工的姓名、部门编号和月薪
SELECT 
    ename, dno, sal
FROM
    tb_emp,
    (SELECT 
        AVG(sal) AS avgsal
    FROM
        tb_emp
    GROUP BY dno) tb_temp
WHERE
    sal >= avgsal;

-- 查询部门中月薪最高的人姓名、月薪和所在部门名称（三表联查）

-- 正确结果：
select ename, sal, dname
from tb_emp t1,tb_dept t2,(
 select dno, max(sal) as max_sal from tb_emp group by dno) t3
    where t1.dno= t2.dno and t1.dno = t3.dno and sal= max_sal;
```

8. 查询主管的姓名和职位

```SQL
-- 查询主管的姓名和职位
select ename,job
from tb_emp
where job like '%主管%';
-- 不仅是职位中带有主管的就是主管
-- 只要员工编号出现在别人的主管编号中就是主管 
select ename,job from tb_emp where eno= any(
select distinct mgr from tb_emp where mgr is not null
);

-- 法二
-- select 'x' (from dual) 查找常量   x写成y也行，写成啥都行
 -- from dual ---> 伪表可省略 
select ename, job from tb_emp t1 where exists(
 select 'x' from tb_emp t2 where t1.eno=t2.mgr
    );
```

## 3. 窗口函数

MySQL8 有窗口函数：`row_num() / rank() /dense_rank()` 效率也不高
窗口数据库不适合业务数据库，只适合离线数据分析
做业务的时候 不能用 用户体验很糟糕

- MySQL8 有窗口函数：`row_num() / rank() /dense_rank()`
  
  - `row_num()` 是独一无二的
  - `dense_rank()` 允许排序重复 不跳过数值 1 22 3
  - `rank()` 允许排名重复  跳过数值 1 22 4
  - `lag(n,m,0)` 获取n(当前记录)前面的前m条的数据，如果有就取出来，没有就取0
  - `lead(n,m,0)` 获取n后面的m条数据，如果有就取出来，没有就取0
    - `lead(4,5,0)` ---> 在数字4 后面，往后拿5条数据

### 3.1 使用窗口函数

9. 查询月薪排名4~6名的员工排名、姓名和月薪 (关键:排名)

```sql
-- 三种函数结果：
select 
 ename, sal, 
    row_number() over (order by sal desc) as row_num,
    rank() over(order by sal desc) as ranking,
    dense_rank() over (order by sal desc) as den_ranking
from tb_emp limit 3,3;
```

用`row_num()`是最好的，保证了唯一性也便于后续利用`limit&offset`取值。如果要求使用`dense_rank()`和`rank()`函数，则需要建立一个临时表再次排序。

```sql
-- 要求必须使用dense_rank
select 
 ename, sal, 
    dense_rank() over (order by sal desc) as den_ranking
from tb_emp limit 3,3; -- 这样得到的结果为 3 4 5
-- 取得时 第四条 第五条  第六条 但不代表是 第四五六名alter
-- 把结果做成临时表
select 
 ename, sal, ranking from(
    select ename, sal, dense_rank() over (order by sal desc) as den_ranking from tb_emp
    ) tb_temp where ranking between 4 and 6;
```

#### 例题：找出每个学校GPA最低的同学

现在运营想要找到每个学校gpa最低的同学来做调研，请你取出每个学校的最低gpa。

直接利用min() 和 group by 不能找出最低gpa对应的id

```sql
select device_id,university,gpa
from
( select *,
    row_number() over (partition by university order by gpa asc) as row_num
    from user_profile) t1
where row_num = 1
order by university
```

### 3.2 不允许使用窗口函数

如果不允许使用窗口函数，则利用`select @a:= @a+1` 这种方式来人工添加行号，其中@a为用户自定义的变量a, `:=`称为海象运算符起到赋值的作用，同`set a =0` 含义相同。

- `@a` 用户自定义的变量a
- `:=` 海象运算符 赋值 或者使用 `set a =0`
- `select @a:= @a+1` 利用这种方式 添加行号

```sql
SELECT 
    row_name, ename, sal
FROM
    (SELECT 
        @a:=@a + 1 AS row_num, ename, sal
    FROM
        tb_emp, (SELECT @a:=0) t1
    ORDER BY sal DESC) t2
WHERE
    row_num BETWEEN 4 AND 6;
```

### 3.3 窗口函数语法

`rank() over (partition by dno order by sal desc)`

`dense_rank() over (order by sal desc)`

## 4. TopN问题

### 4.1 有窗口函数MySQL8.0

窗口函数主要用于解决TopN查询问题

- 例题：查询每个部门月薪排前2名的员工姓名和月薪

#### partition by 和group by

```sql
- 不能使用 group by，使用group by分组后记录只有3条
- partition by 不会使记录变少，还会在每个组里面排名
  
select ename,sal,dno from(
 select ename,sal,dno,rank() over (partition by dno order by sal desc) as ranking
from tb_emp) tb_temp where ranking<=2;
```

获取并返回 Employee 表中第二高的薪水。如果不存在第二高的薪水，查询应该返回 null 。

```SQL
-- 然而，如果没有这样的第二最高工资，这个解决方案将被判断为 “错误答案”，因为本表可能只有一项记录。为了克服这个问题，我们可以将其作为临时表

SELECT
    IFNULL(
      (SELECT DISTINCT Salary
       FROM Employee
       ORDER BY Salary DESC
        LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary
```

### 4.2 没有窗口函数 MySQL5.X

思路：同一个部门中，工资比我高的人不超过2个，我就是工资的前两名

```sql

select ename,sal,dno from tb_emp t1
where (select count(*) from tb_emp t2 where t1.dno=t2.dno and t2.sal>t1.sal) <2
order by dno asc, sal desc;

```

###### 附录——导入数据

```sql
drop database if exists hrs;
create database hrs default charset utf8mb4;

use hrs;

create table tb_dept
(
dno int not null comment'编号',
dname varchar(10)not null comment '名称',
dloc varchar(20) not null comment '所在地',
primary key (dno)
);

insert into tb_dept values
 (10,'会计部','北京'),
 (20,'研发部','成都'),
 (30,'销售部','重庆'),
 (40,'运维部','深圳');

create table tb_emp
(
eno int not null comment '员工编号',
ename varchar(20) not null comment '员工姓名',
job varchar(20) not null comment '员工职位',
mgr int comment '主管编号',
sal int not null comment '员工月薪',
comm int comment '每月补贴',
dno int comment '所在部门编号',
primary key (eno),
foreign key (dno) references tb_dept(dno) ,
foreign key (mgr) references tb_emp(eno)
);

insert into tb_emp values
(7800,'张三丰', '总裁',null, 9000, 1200, 20),
(2056,'乔峰', '分析师', 7800,5000, 1500, 20),
(3088,'李莫愁','设计师', 2056, 3500, 800, 20),
(3211,'张无忌', '程序员', 2056, 3200,null, 20) ,
(3233,'丘处机','程序员',2056,3400, null, 20),
(3251,'张翠山', '程序员',2056,4000,null, 20),
(5566,'宋远桥', '会计师', 7800,4000,1000, 10),
(5234,'郭靖', '出纳', 5566, 2000, null, 10),
(3344,'黄蓉', '销售主管', 7800,3000,800, 30),
(1359,'胡一刀','销售员',3344,1800, 200, 30),
(4466,'苗人凤', '销售员', 3344,2500, null, 30),
(3244,'欧阳锋','程序员', 3088, 3200, null, 20) ,
(3577,'杨过','会计', 5566, 2200, null, 10),
(3588,'朱九真', '会计', 5566, 2500,null, 10);


```
