# day12 面试题剖析

## 面试题一——利用case构造新的列

### 0. 建表语句

```sql
drop database if exists demo2;
create database `demo2` default  charset utf8mb4;

use demo2;
drop table if exists tb_result;

create table tb_result
(
 rq date not null,
 sf char(1) not null
)engine= InnoDB;

lock tables tb_result write;

insert into tb_result values
 ('2017-04-09','胜'),
 ('2017-04-09','胜'),
 ('2017-04-09','负'),
 ('2017-04-09','负'),
 ('2017-04-10','胜'),
 ('2017-04-10','负'),
 ('2017-04-10','胜');
    
unlock tables;


drop table if exists tb_course;
create table tb_course
(
course_id int unsigned not null comment '课程编号',
course_name varchar(50) not null comment '课程名称',
primary key (course_id)
)engine=InnoDB;

lock tables tb_course write;

insert into tb_course values
    (1,'Python'),
    (2,'Java'),
    (3,'Database'),
    (4,'JavaScript');
    
unlock tables;

drop table  if exists tb_open_date;
create table tb_open_date
(
month char(6) not null comment'开课日期',
course_id int unsigned not null comment '课程编号'
)engine=InnoDB;

LOCK tables tb_open_date write;

insert into tb_open_date values
 ('202106',1),
    ('202106',3),
    ('202106',4),
    ('202107',4),
    ('202108',2),
    ('202108',4);
    
unlock tables;
```

### 1. 用上面的表查询出如下所示的结果

 | date       | win | los |
 | ---------- | --- | --- |
 | 2017-04-09 |  2  | 2   |
 | 2017-04-10 |  2  | 1   |

- 把长表查成一个宽表
- win 和los 不是原表中有的 需要自己强行加两个列
- 利用case 构造分支 然后取别名

```sql
select 
    rq as date,
    sum(case sf when '胜' then 1 else 0 end) as win,
    sum(case sf when '负' then 1 else 0 end) as los
from tb_result group by rq;
```

### 2. 用上面的表查询出如下所示的结果

| course_name | Jun | JUl | AUG |
| ----------- | --- | --- | --- |
| Python      | O   | x   | x   |
| Java        | x   | x   | O   |
| Database    | O   | x   | x   |
| Python      | O   | O   | O   |

- 构造三个列
- o 还是x 是分支结构 + 存在性判断 + 分支子查询
  
 ```sql
select course_name,
 case when exists
    (
    select 'x' from tb_open_date as t2   -- 可以查course_id 也可以直接查常量
    where t1.course_id = t2.course_id and month = '202106'
    ) then 'o' else 'x' end as Jun,
     case when exists
    (
    select 'x' from tb_open_date as t2  
    where t1.course_id = t2.course_id and month = '202107'
    ) then 'o' else 'x' end as Jul,
     case when exists
    (
    select 'x' from tb_open_date as t2  
    where t1.course_id = t2.course_id and month = '202106'
    ) then 'o' else 'x' end as Aug
from tb_course as t1;
```

## 面试题二——商品购买信息

### 0.建表语句

```SQL
drop database if exists demo1;

create database demo1 charset utf8mb4;

use demo1;
drop table if exists tb_order;

create table tb_order(
    id int not null,
    order_no varchar(20) not null comment '订单号',
    user_id varchar(50) not null comment '用户号',
    order_dat date not null comment '下单日期',
    store varchar(50 ) not null comment '店铺号',
    product varchar(50) not null comment '商品号',
    quantity int not null comment '购买数量',
    primary key (id)
) engine=innodb comment='订单表';

lock tables tb_order write;

insert into tb_order  values
    ( 1, 'd001', 'customera' , '2018-01-01', 'storea', 'proda',1),
    ( 2, 'd001', 'customera' , '2018-01-01', 'storea','prodb', 1),
    ( 3, 'd001', 'customera' , '2018-01-01', 'storea','prodc', 1),
    ( 4, 'd002', 'customerb' , '2018-01-12', 'storeb','prodb', 1),
    ( 5, 'd002', 'customerb' , '2018-01-12', 'storeb', 'prodd',1),
    ( 6, 'd003', 'customerc' , '2018-01-12', 'storec', 'prodb',1),
    ( 7, 'd003', 'customerc' , '2018-01-12', 'storec','prodc', 1),
    ( 8, 'd003', 'customerc' , '2018-01-12', 'storec','prodd', 1),
    ( 9, 'd004', 'customera' , '2018-01-01', 'stored','prodd', 2),
    ( 10,'d005', 'customerb' , '2018-01-23', 'storeb','proda', 1);

unlock tables;

drop table if exists tb_product;

create table tb_product(
    prod_id varchar(50) not null comment '商品号',
    category varchar(50) not null comment '种类',
    color varchar(10) not null comment '颜色',
    weight decimal(10,2) not null comment '重量',
    price int not null comment '价格',
    primary key (prod_id)
) engine=innodb comment='产品表';

lock tables tb_product write;
insert into tb_product values
    ( 'proda', 'catea','yellow' ,5.60,100 ),
    ( 'prodb', 'cateb', 'red' ,3.70,200),
    ( 'prodc', 'catec', 'blue' ,10.30,300),
    ( 'prodd ' , 'cated ' , 'black' ,7.80,400);

unlock tables;

drop table if exists tb_store;

create table tb_store(
    store_id varchar(50) not null comment '店铺号',
    city varchar(20) not null comment '城市',
    primary key (store_id)
) engine=innodb comment='店铺表';

lock tables tb_store write;

insert into tb_store values
    ( 'storea' , 'citya' ),
    ( 'storeb' , 'citya' ),
    ( 'storec' , 'cityb' ),
    ( 'stored' , 'cityc' ),
    ( 'storee' , 'cityd' ),
    ( 'storef' , 'cityb' );

unlock tables;
```

### 1.查询出购买金额不低于800的用户的总购买金额，总订单数和总购买商品数

- 订单号相同的算作一单

```SQL
select 
 user_id,
    sum(price * quantity) as '总购买金额',
    count(distinct order_no) as '总订单数',
    sum(quantity) as '总购买商品数'
from tb_order inner join tb_product on product = prod_id
group by user_id having '总购买金额' >= 800;
```

### 2. 查询所有城市（包含无购买记录的城市）的总店铺数，总购买人数和总购买金额

- 三表联查 店铺表为右外连接

```SQL
select
 city,
    count(distinct store_id) as 总店铺数,
    count(distinct user_id) as 总购买人数,
    ifnull(sum(quantity * price),0) as 购买总金额 -- sum忽略空值 
from tb_order inner join tb_product on prod_id = product
right join tb_store on store_id = store -- 店铺表需要查完整
group by city; 
```

### 3. 查询购买过 catea 产品的用户和他们的平均订单金额

- 订单号相同的算做一单

- 子查询---> 查到购买a产品的用户集合 ---> 根据用户分组 ---> 聚合函数
- 平均订单金额 不是avg 是总价除以订单数

```SQL
select 
    user_id,
    sum(price * quantity) / count(distinct order_no) as 平均订单金额
    -- 不能用avg 因为订单号相同算作一单
from tb_order inner join tb_product
on product = prod_id where user_id in
(
 select user_id from tb_order
    inner join tb_product on prod_id = product
    where category = 'catea'
) 
group by user_id;
```

## 面试题三——求众数和中位数

### 0.建表语句

```sql
create database demo3 default charset utf8mb4;
use demo3;
drop table if exists tb_graduate;
create table tb_graduate
(
g_name varchar( 50) not null comment '名字',
g_income int not null comment '收入'
) engine=innodb ;

lock tables tb_graduate write;
insert into tb_graduate values
('桑普森',400000 ),
( '迈克',30000),
( '怀特',20000),
('阿诺德',20000),
('史密斯',20000),
('劳伦斯',15000),
('哈德逊',15000),
('肯特',10000),
('贝克',10000),
('斯科特',10000);

unlock tables;
```

### 1.用上面的表查出毕业生收入的众数

- 众数不唯一
- 众数出现的次数要大于等于 其他数出现的次数  ---> 众数
- `all` 全程量词 所有的都要满足条件
- `exists` 存在量词 存在就行

```sql
select g_income,count(*) as cnt
from tb_graduate
group by g_income having cnt >= all(
  select count(*) from tb_graduate group by g_income
);

-- 法二
select g_income,count(*) as cnt
from tb_graduate
group by g_income having cnt =(
 select max(cnt) from (
    select count(*) as cnt from tb_graduate group by g_income
    ) as temp 
);-- 数量等于最大值就是众数
-- 不能直接使用 max(count(*)) 不符合聚合函数的用法
```

### 2. 用上面的表查出毕业生收入的中位数

- 先找出收入比半数以上要高
- 在找出收入比半数以上要低的
- 两个集合交集---> 可能有两个中位数--->再求平均就是 中位数

```SQL
-- 关联子查询
select avg(distinct g_income) as 中位数
from (
 select t1.g_income 
    from tb_graduate as t1 , tb_graduate as t2
    group by t1.g_income
    having
  sum(case when t2.g_income >= t1.g_income then 1 else 0 end)>=count(*)/2
  and
  sum(case when t2.g_income <= t1.g_income then 1 else 0 end)>=count(*)/2
) as temp;
```

## 面试题四——用户留存数据

### 0.建表语句

```SQL
drop database if exists demo4;
create database demo4 default charset utf8mb4;

use demo4;
create table tb_user_login
(
user_id varchar(300) comment '用户ID',
login_date date comment '登录日期'
) engine=innodb;

insert into tb_user_login (user_id, login_date) values
('A', '2019/9/2'),
('A', '2019/9/3'),
('A', '2019/9/4'),
('B', '2019/9/2'),
('B', '2019/9/4'),
('C', '2019/1/1'),
('C', '2019/1/2'),
('C', '2019/9/3'),
('C', '2019/9/4'),
('C', '2019/9/5');
```

### 1. 查询连续三天登录的用户ID (常见)

- 时间升序 ---> 针对每个用户的登录信息 建立行号 日期建立编号
- -> 连续三个日期减去编号都等于 第一个日期 --->就是连续三天登录
- subdate()写成date_sub也行，
- 但是date_sub 必须写上时间单位,例如 date_sub(time1,time2 day)
  
```SQL
select 
 user_id
from(
 select 
    user_id, login_date,
    row_number() over (partition by user_id order by login_date) as rn
    from tb_user_login
) as temp
group by user_id
having count(subdate(login_date,rn)) >= 3;
```

### 2. 查询每天新增用户数以及他们的次日留存率和三十日留存率 (重要！)

- 402010法则
  - -> 第2天 留存40% 第7天 留存20%的用户 一个月过后留存10% --->产品比较健康
- **`GROUP BY`和`PARTITION BY`**
  - group by 会让数据变少 比如分3个组---> 第一个里面就只有2条数据了
  - partition by 不会导致数据变少

```SQL
select
    first_login,
    count(distinct t1.user_id) as 新增用户数,
    count(distinct t2.user_id) / (count(distinct t1.user_id)) as 次日留存率,
    count(distinct t3.user_id) / (count(distinct t1.user_id)) as 三十日留存率
from(
    select user_id,login_date,
    min(login_date) over (partition by user_id order by login_date) 
        as first_login  -- 找到每个用户的首次登录时间
    from tb_user_login
) as t1
left join tb_user_login as t2  
-- t2 满足条件的才会留下来，即t2中为次日留存的数据
on t1.user_id = t2.user_id and datediff(first_login,t2.login_date) = -1
-- 后面的日期比第一条登录的日期刚好大一天 ---> 次日留存用户
left join tb_user_login as t3  
-- t1要查完整 t3是满足条件的数据
on t1.user_id = t3.user_id and datediff(first_login,t3.login_date) = -29
group by first_login;
```

## 问题五——用户活跃数据

### 0. 建表语句

```sql
create table tb_order
(
order_id bigint unsigned comment'订单ID',
user_id varchar(300) comment'用户ID',
order_pay int comment '订单金额',
order_time datetime comment '下单时间'
) engine=innodb;

insert into tb_order values
(1,'A',100,'2021-11-10' ),
(1,'A',200,'2021-11-10' ),
(2,'B',300,'2021-11-10' ),
(3,'C',400,'2021-11-11' ),
(3,'C',500,'2021-11-11' ),
(3,'C',600,'2021-11-11' ),
(4,'A',700,'2021-11-12' ),
(4,'A',800,'1021-11-12' ),
(5,'B',700,'2021-11-12' ),
(6,'B',600,'2021-11-12' ),
(7,'A',500,'2021-11-13' ),
(7,'A',400,'2021-11-13' ),
(8,'D',300,'2021-11-12' ),
(9,'E',200,'2021-11-12' );

create table tb_act_apply
(
act_id int unsigned comment '活动编号',
user_id varchar(30) comment '用户ID',
act_time datetime comment '报名时间'
) engine=innodb;

-- 用户参与活动的报名表
insert into tb_act_apply values
(1,'A','2021-11-11'),
(2,'B','2021-11-09'),
(3,'C','2021-11-11'),
(1,'D','2021-11-12');

-- 用户行为日志
create table tb_tracking_log
(
user_id varchar(300) comment '用户ID',
oper_id int unsigned comment '操作ID',
log_time  datetime comment '操作时间'
)engine=innodb;

insert into tb_tracking_log values
 ( 'A',1,'2021-11-11 09:20:00'),
 ( 'A',2,'2021-11-11 09:21:00'),
 ( 'A',5,'2021-11-11 09:22:00'),
 ( 'B',5,'2021-11-11 09:20:00'),
 ( 'B',1,'2021-11-11 09:21:00'),
 ( 'C',4,'2021-11-12 09:20:00'),
 ( 'D',1,'2021-11-12 09:20:00'),
 ( 'D',3,'2021-11-12 09:21:00'),
 ( 'D',2,'2021-11-12 09:22:00'),
 ( 'E',1,'2021-11-11 09:20:00'),
 ( 'E',2,'2021-11-12 09:21:00');
```

### 1. 查询每个活动从开始后 到2021年11月13日前平均每天产生的订单数

- 活动开始时间定义为最早有用户报名的时间
- 每个互动的订单总数除以活动天数（活动最早用户报名时间2021-11-13）
- 订单数可能会重复
  
```sql
select
 act_id,
    count(distinct order_id) / datediff('2021-11-13',begin_time) as 每天平均订单数
from(
 select
  act_id, user_id, act_time,
        min(act_time) over (partition by act_id) as begin_time
        from tb_act_apply
) as  t1 inner join tb_order t2
on t1.user_id = t2.user_id
where order_time >= act_time and order_time < '2021-11-13'
group by act_id;
```

### 2. 查询每天的访客数和平均操作次数

- 访客数需要去重 不计算反复访客

```sql
select 
    date(log_time) as date, -- log_time中包含时分秒 利用date() 提取日期
    count(distinct user_id) as 访客数,
    count(*) / count( distinct user_id) as 平均操作次数
from tb_tracking_log group by date;
```

### 3. 查询每天中符合以下条件的用户ID

要求：操作1之后中操作2 ，操作1和2 必须相邻 ---> 窗口函数 lag lead

**窗口函数介绍**

- `lag(n,m,0)` 获取n(当前记录)前面的前m条的数据，如果有就取出来，没有就取0
- `lead(n,m,0)` 获取n后面的m条数据，如果有就取出来，没有就取0、

```SQL
select user_id
from
(
 select user_id, oper_id, log_time,
    lag(oper_id,1) over (partition by date(log_time),user_id order by log_time) as prev_oper_id
    from tb_tracking_log
) as temp where oper_id =2 and prev_oper_id =1;
-- 当前操作为2 lag提取的上一个操作是1

-- 利用lead()
select user_id
from(

    select User_id,oper_id,log_time
    lead(oper_id,1) over (parition by date(log_time),user_id order by log_time ) as later_oper_id
    from tb_tracking_log
) as temp where oper_id = 1 and later_oper_id =2; 
```
