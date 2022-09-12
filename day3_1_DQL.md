# day3——DQL 数据查询语言

## 1. 知识集合

- 投影和别名
  - 指定要查的列 这个操作叫投影
  - as 列的别名

**Coalesce()函数**

 `coalesce (expression_1, expression_2, ...,expression_n)`

- 依次参考各参数表达式，遇到非null值即停止并返回该值。
- `select coalesce(success_cnt, 1) from tableA`
  - 当success_cnt 为null值的时候，将返回1，否则将返回success_cnt的真实值。
- `select coalesce(success_cnt,period,1) from tableA`
  - 当success_cnt不为null，那么无论period是否为null，都将返回success_cnt的真实值（因为success_cnt是第一个参数），当success_cnt为null，而period不为null的时候，返回period的真实值。只有当success_cnt和period均为null的时候，将返回1。

**分支结构 case when then end 构造分支结构**

```SQL
计算25岁以上和以下的用户数量
select 
case when age < 25 or age is null then '25岁以下'
        when age >= 25 then '25岁及以上'
        end age_cut,count(*) number
from user_profile
group by age_cut

```

**字符串操作**

- `char_length` 计算字符串大小
- `UPPER()` 转化为大写
- `left()` 返回字符串左边的字符
- `substr` 
- `substring(a,n,m)` 截取字符串a第n-m个字符
- `conact()`拼接字符串
- `group_concat`用法：
  - `group_concat([去重：distinct] '字符串' [排序：group by 该字符串 asc/desc] 分隔符：separator ',')`

  ```sql
  select sell_date, 
    count(DISTINCT product) as num_sold,
    GROUP_CONCAT(DISTINCT(product) order by product SEPARATOR ",") as products
    -- 每个日期的销售产品名称应按词典序排列

from Activities
group by sell_date
order by sell_date

  ```

- `if`函数 mysql方言

   (因为其他数据中可能没有if函数，Oracle中做同样事情的函数叫做Decode
- 不知道准确的名字 就不能写成等号 此时写like

```sql
--复旦大学的每个用户在8月份练习的总题目数和回答正确的题目数情况
select 
    up.device_id,university,
    ifnull(count(question_id),0) as question_cnt,
    sum(if(result='right',1,0))
from  user_profile up
left join question_practice_detail  qpd   
on qpd.device_id = up.device_id and month(qpd.date) = 8
where university = '复旦大学' 
group by device_id;
```

**正则表达式**

- SQL中使用通配符 % ---> 匹配0个or任意多个字符
  
  - 提示：前面带%的模糊查询性能基本上非常糟糕
- 正则表达式模糊查询 `regexp` ---> regular expression
  - 杨. ---> 匹配杨和以后的任意字符  
  - ^杨.{1,2}$ ---> 任意字符出现1次或2次

**字符串分割**

- 字符串分割 `substring_index(str,delim,count)`
  - str：字符；delim：分隔符; count：计数
  - str = www.wikibt.com ---> substring_index(str,'.',2) ---> www wikibt
  - 如果是需要中间的某一个字符串，则需要嵌套使用
- 字符串分割`substring(s,index,len)`
  - 返回从字符串s的index位置其长度为len个字符

**排序**

- `Oder by` 排序 ---> asc - 升序 （从小到大），decs 降序 （从大到小）
  - `group by XX` 根据XX分组
- `order by FIELD(difficult_level,'easy','medium','hard');`
  - 根据指定的排序方式对某一列进行排序
- 四舍五入 round(X,2) 保留两位小数后的四舍五入
  
**时间计算**

- `datediff()`计算日期的时间差
- `day()` 提取时间中的日期
- `TIMESTAMPDIFF(interval, time_start, time_end)`
- `last_day()`返回参数日期的最后一天
  
   可计算time_start-time_end的时间差，单位以指定的interval为准，常用可选：
  - SECOND 秒
  - MINUTE 分钟（返回秒数差除以60的整数部分）
  - HOUR 小时（返回秒数差除以3600的整数部分）
  - DAY 天数（返回秒数差除以3600*24的整数部分）
  - MONTH 月数
  - YEAR 年数

```sql
-- 计算用户留存率 
--不关心同一用户（设备）在这天答了什么题、答题结果如何，只关心他是否答题
-- 次日留存率 = 去重后符合次日留存的数据 / 去重的总数据
select 
    count(q2.device_id) / count(q1.device_id) as avg_ret
from
    (select distinct device_id, date from question_practice_detail)  q1
    left join
    (select distinct device_id, date from question_practice_detail) q2
    on q1.device_id = q2.device_id and  q2.date = DATE_ADD(q1.date, interval 1 day)
```

**分组以前的数据筛选使用where子句，分组以后的数据筛选使用having子句**

## 2. 统计学常识

- 描述性统计：能拿到全量数据
  - 集中趋势：均值、中位数、众数
  - 离散趋势：极差、方差、标准差
  - 相关性：协方差、相关性(Spearman、Pearson、Kindall)
- 推断性统计：用样本推断总体
  - t检验和F检验：样本的均值和方差能不能代表总体的均值和方差
  - 方差分析：检查数据的改变是否是入籍波动造成的，是否具体显著性

**SQL中获取数据的描述性统计信息的函数:**

    sum / avg / min / max / count / stddev /var

## 3. SQL语句书写顺序

**一定要记下来!!**

**分组以前的数据筛选使用where子句，分组以后的数据筛选使用having子句**

```sql
select ..., ..., ...
    from ..., ...
    where ... 
        and ... or ...
    group by ..., ...
    having ...
    order by ... asc, ...desc
    limit ... offset ...
```

## 4.    窗口函数的语法

```sql
    row_num() over (partition by ... order by ...) ---> OLAP查询 ---> 离线查询
```

## 5. 代码合集

```sql
-- 数据写入
insert into `tb_student`
(`stu_id`,`stu_name`,`stu_sex`,`stu_birth`,`stu_add`, `col_id` )
values
(1001,'杨过过',1,'1990-3-4', '湖南长沙',1),
(1002,'任我行',1, ' 1992-2-2','湖南长沙',1),
(1033,'王语嫣',0,'989-12-3','四川成都',1),
(1572,'岳不群',1,'1993-7-19','shan西咸阳',1),
(1378,'纪嫣然',0,'1995-8-12','四川绵阳',1),
(1954,'林平之',1,'1994-9-20','福建莆田',1),
(2035,'东方败',1,'1988-6-30',null, 2),
(3011,'周震南',1,'1985-12-12','福建莆田',3),
(3755,'项少龙',1,'1993-1-25', null, 3),
(3923,'杨不悔',0,'1985-4-17','四川成都',3);
```

```SQL
use school;


-- 1.查询所有学生的所有信息
-- 不建议写* 效率低下 
select * from tb_student;

-- 好的写法 把所有的课程列写出来 
select stu_id,stu_name,stu_sex,stu_birth,stu_add,col_id 
from tb_student;
-- 查询所有课程名称及学分(投影和别名) 
-- 指定要查的列 这个操作叫投影
-- as 列的别名
select cou_name as 课程名称,cou_credit as 学分 from tb_course;


-- 2.查询所有女学生的姓名和出生日期(筛选)
select stu_name,stu_birth from tb_student  where stu_sex = 0;


-- 3.查询所有80后学生的姓名、性别和出生日期(筛选)
select stu_name,stu_sex,stu_birth from tb_student 
where stu_birth >= '1980-1-1' and stu_birth <= '1989-12-31'; 
-- 法二
select stu_name,stu_sex,stu_birth from tb_student 
where stu_birth between '1980-1-1' and '1989-12-31'; 

-- 3.查询所有80后女学生的姓名、性别和出生日期(筛选)
select stu_name,stu_sex,stu_birth from tb_student 
where stu_birth between '1980-1-1' and '1989-12-31' and stu_sex = 0; 

-- 分支结构 case when then end 构造分支结构
select 
 stu_name as 姓名,
 case stu_sex when 1 then '男' else '女' end as 性别,
    stu_birth as 生日 from tb_student 
where stu_birth between '1980-1-1' and '1989-12-31';

-- mysql方言 (因为其他数据中可能没有if函数)
-- Oracle中做同样事情的函数叫做Decode
select
 stu_name as 姓名,
 if(stu_sex,'男','女') as 性别,
    stu_birth as 生日 
from tb_student 
where stu_birth between '1980-1-1' and '1989-12-31';


-- 4.查询姓”杨“的学生姓名和性别(模糊)
-- 不知道准确的名字 就不能写成等号 此时写like
-- SQL中使用通配符 % ---> 匹配0个or任意多个字符
select stu_name,stu_sex from tb_student where stu_name like '杨%';


-- 5.查询姓”杨“名字两个字的学生姓名和性别(模糊)
-- 使用下划线_
select stu_name,stu_sex from tb_student where stu_name like '杨_';
-- 查询姓”杨“名字三个字的学生姓名和性别(模糊)
select stu_name,stu_sex from tb_student where stu_name like '杨__';


-- 6.查询名字中有”不“字或“嫣”字的学生的姓名(模糊)
-- 提示：前面带%的模糊查询性能基本上非常糟糕
-- or 后面直接写'%嫣%'是不对的！or后面应该要是一个条件
select stu_name,stu_sex from tb_student 
where stu_name like '%不%'or stu_name like '%嫣%';
-- 法二 union并集运算
select stu_name,stu_sex from tb_student where stu_name like '%不%'
union
select stu_name,stu_sex from tb_student where stu_name like '%嫣%';

update stu_name set stu_name='岳不嫣' where stu_id = 1572;

-- 正则表达式模糊查询 regexp ---> regular expression
-- 杨. 匹配杨和以后的任意字符  
-- ^杨.{1,2}$ 任意字符出现1次或2次
select stu_name,stu_sex from tb_student where stu_name regexp '杨.{2}';


-- 7.查询没有录入家庭住址的学生姓名(空值)
-- 空值做任何运算结果也是空值，null相当于条件不成立
select stu_name from tb_student where stu_add is null;


-- 8.查询录入了家庭住址的学生姓名(空值) 最好不要写等号 和不等号
-- SQL中不等号的写法<> 不支持！=  等号为<=>
select stu_name,stu_add from tb_student where stu_add is not null;
select stu_name,stu_add from tb_student where stu_add is null;


-- 9.查询家庭住址（去重） distinct
select distinct stu_add from tb_student where stu_add is not null;


-- 10.查询学生选课的所有日期(去重)
select distinct sel_date from tb_record;


-- 11.查询男学生的姓名和生日按年龄从大到小排列(排序)
-- Oder by 排序
-- asc - 升序 （从小到大），decs - 降序 （从大到小）
select stu_name,stu_birth from tb_student
where stu_sex=1 order by stu_birth asc;


-- 12.查询年龄最大的学生的出生日期(聚合函数) ---> 找出最小的生日
select now(); -- 查询现在的时间和日期
select curdate(); -- 查询现在的日期
datediff() --计算时间差
-- floor 向下取整

select
 min(stu_birth) as 生日,
    floor(datediff( curdate(),min(stu_birth))/365) as 年龄
from tb_student;

-- 13.查询年龄最小的学生的出生日期(聚合函数)
select
 min(stu_birth) as 生日,
    floor(datediff(curdate(),max(stu_birth))/365) as 年龄
from tb_student;


-- 14.查询所有考试的平均成绩
-- 聚合函数遇到null值会做忽略处理
select avg(score) from tb_record;

 -- 考虑空值 使用count(*)
  select sum(score) / count(*) from tb_record;


-- 15.查询课程编号为1111的课程的平均成绩(筛选和聚合函数)
select avg(score) from tb_record where cid=1111;


-- 16.查询学号为1001的学生所有课程的平均分(筛选和聚合函数)
select avg(score) from tb_record where sid=1111;


-- 17.查询男女学生的人数(分组和聚合函数)
-- group by 根据上面分组
-- 分完组以后在进行聚合函数
-- SAC(Split - Aggregate - Combine)
select 
 if(stu_sex,'男','女') as 性别,
 count(*) as 人数
from tb_student group by stu_sex;


-- 18.查询每个学生的学号和平均成绩(分组和聚合函数)
select sid,round(avg(score),2) from tb_record group by sid;


-- 19.查询平均成绩大于等于90分的学生的学号和平均成绩
-- 分组以前的数据筛选使用where子句，分组以后的数据筛选使用having子句
select
 sid,
    round((score),2) as 平均分 
from tb_record 
group by sid having 平均分 >=90;




```
