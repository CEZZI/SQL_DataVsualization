# day_14 SQL进阶篇

牛客网刷题

## 03 分组聚合查询

### 分组查询

#### SQL128.未完成试卷数大于1的有效用户

请统计2021年每个未完成试卷作答数大于1的有效用户的数据（有效用户指完成试卷作答数至少为1且未完成数小于5），

输出用户ID、未完成试卷作答数、完成试卷作答数、作答过的试卷tag集合，按未完成试卷数量由多到少排序。

- 未完成试卷作答数大于1 ---> 有效用户(子查询)
- 垃圾牛客网聚合函数后面必须有group by,如果count放在查询有效客户的子查询中 会引发错误，所以 count 和group by 都放在最后一个查询中
- **统计作答过的tag集合：**
  - 对于每条作答tag，用:连接日期和tag：`concat_ws(':', date(start_time), tag)`
    - 连接效果如下 时间:tag
  - 对于一个人（组内）的多条作答，用连接去重后的作答记录：`group_concat(distinct concat_ws(':', date(start_time), tag) SEPARATOR ';')`
- `concat`和`concat_ws`
  - concat 可以把int型转化为str型进行拼接，concat_ws不行
  - 是否拼接NULL：oncat拼接时，只要参数中有null(有一个null即可)，不管有多少不为空的参数，结果都为null；concat_ws遇到参数有null时，则会忽略，不会返回null。


```sql
select uid,
    count(incomplete) as incomplete_cnt,
    count(complete) as complete_cnt,
    group_concat(distinct concat_ws(':', date(start_time), tag) SEPARATOR ';') as detail
from(
    select uid,tag,start_time,
        (if(submit_time is null, 1, null)) as incomplete,
        (if(submit_time is null, null, 1)) as complete
    from exam_record left join examination_info using(exam_id)
    where year(start_time) = 2021

    )  as t1
    group by uid
    having incomplete_cnt between 2 and 4 and complete_cnt>= 1
    order by incomplete_cnt desc
```
## 04 多表查询

### 4.1 嵌套子查询

#### SQL129 月均完成试卷数不小于3的用户爱作答的类别

当月均完成试卷数”不小于3的用户们爱作答的类别及作答次数，按次数降序输出

- 月均完成试卷数的计算`count(exam_Id)/count(date_format(start_Time,'%Y%m'))` 
- 不是像以前一样把所有符合结果的数据都提取出来 利用`where uid in{}`来实现数据筛选
  
```sql
select tag,count(tag) as tag_cnt
from exam_record
join examination_info using(exam_id)
where uid in 
    -- 计算月均完成试卷数不小于3的用户
   (
       select uid 
       from exam_record
       where submit_time is not null -- 提交时间非空才能计算
       group by uid
       having count(exam_id) / count(distinct date_format(start_time,'%Y%m')) >=3
       )
    group by tag
    order by tag_cnt desc

```

#### SQL130 试卷发布当天作答人数和平均分

请计算每张SQL类别试卷发布后，当天5级以上的用户作答的人数uv和平均分avg_score，按人数降序，相同人数的按平均分升序，示例数据结果输出如下

```SQL
SELECT
    exam_id,
    count( DISTINCT uid ) AS uv,
    ROUND(avg( score ), 1) AS avg_score
FROM exam_record 
WHERE (exam_id, DATE(start_time)) IN (
    SELECT exam_id, DATE(release_time)
    FROM examination_info WHERE tag = "SQL"
) AND uid IN ( SELECT uid FROM user_info WHERE `level` > 5 )
GROUP BY exam_id
ORDER BY uv DESC, avg_score ASC;
```

#### SQL131 作答试卷得分大于过80的人的用户等级分布

统计作答SQL类别的试卷得分大于过80的人的用户等级分布，按数量降序排序（保证数量都不同）

```SQL
select `level`, count(distinct uid) as level_cnt
from user_info left join exam_record using(uid) join examination_info using(exam_id) 
where score > 80 and tag='SQL'
group by `level`
order by  count(distinct uid) desc,level des
```


### 4.2 合并查询

#### SQL132

请统计每个题目和每份试卷被作答的人数和次数，分别按照"试卷"和"题目"的uv & pv降序显示

```sql
-- 如何把题目和试卷合并到一个查询结果里 union
-- 一定要要对表进行重新命名，否则报错
select * from
(select exam_id as tid,count(distinct uid) as uv, -- 被作答的人数
    count(exam_id) as pv -- 被作答的次数
from exam_record
group by exam_id
order by uv desc,pv desc) t1

union all
select * from(
select question_id as tid,count(distinct uid) as uv, -- 被作答的人数
    count(question_id) as pv -- 被作答的次数
from practice_record
group by question_id 
order by uv desc,pv desc
) t2
```

#### SQL133-分别满足两个活动的人

请写出一个SQL实现：输出2021年里，所有每次试卷得分都能到85分的人以及至少有一次用了一半时间就完成高难度试卷且分数大于80的人的id和活动号，按用户ID排序输出。

```sql
-- 没必要使用case when 条件 then XX end 来构造活动列 多此一举 
-- 直接使用条件筛选即可

-- 输出2021年里，所有每次试卷得分都能到85分的人 --> 只需调用exam_record表
select * from(
select distinct uid, 'activity1' as activity
from exam_record
where year(submit_time) =2021 -- 直接在此处加上分数大于最低分85的条件 会出现聚合函数使用错误
group by uid
having min(score) >= 85
) t1
union all

select * from( -- 至少有一次用了一半时间就完成高难度试卷且 分数大于80的人的id和活动号
    select distinct uid, 'activity2' as activity
   from exam_record left join examination_info using(exam_id)
   where year(submit_time) =2021 and difficulty = 'hard'and score > 80
        and timestampdiff(minute, start_time, submit_time) * 2 < duration
) t2
order by uid

```


### 4.3 连接查询

select uid,
    count(uid) as exam_cnt, count(uid) as question_cnt
from 
    user_info left join exam_record using(uid) join practice_record using(uid)

where uid in (
    select uid from user_info left join exam_record using(uid) join examination_info using(exam_id)
    where avg(score) > 80 and tag='SQL' and difficulty='hard' and level =
)

#### SQL134 满足条件的用户的试卷完成数和题目练习数

```sql
-- 试卷的完成数和题目的完成数 需要利用两个select查询  union
 
select uid, exam_cnt,
    if(question_cnt is null, 0, question_cnt) -- 没有考试次数时 要记为0
from(
    select uid, count(submit_time) as exam_cnt
    from exam_record
    where YEAR(submit_time) = 2021
    group by uid
) t
 
left join(
    select uid, count(submit_time) as question_cnt
    from practice_record
    where YEAR(submit_time) = 2021
    group by uid
) t2 using(uid)

where uid in  -- 高难度SQL试卷得分平均值大于80并且是7级的红名大佬
(
    select uid from user_info left join exam_record using(uid) join examination_info using(exam_id)
    where tag='SQL' and difficulty='hard' and level = 7
    group by uid
    having avg(score) > 80 -- 分组之后查询使用having语句
)
order by exam_cnt asc, question_cnt desc

```

#### SQL135 每个6/7级用户活跃情况

**hard** ---> 连接查询 <--- 查询的表通过union all 来实现 试卷和练习结果的链接


```SQL
-- 请统计每个6/7级用户总活跃月份数 ---> 需要把试卷和联系的日期加起来
-- 2021年活跃天数 ---> 不管有没有提交都算活跃 
    -- 2021年活跃天数 != 2021年试卷作答活跃天数+2021年答题活跃天数，
    -- 如果同一天完成试卷跟答题依然是一天
-- 2021年试卷作答活跃天数、
-- 2021年答题活跃天数，
-- 按照总活跃月份数、2021年活跃天数降序排序。

-- 需要把不同的日期变成列输出 不能使用多个 查询再union
-- 涉及到多个表 需要对表进行重命名
select u_i.uid as uid,
       count(distinct act_month) as act_month_total,
       count(distinct case 
             when year(act_time) = 2021  -- 查询表中把答题和试卷合并在一起后 需要case when 进行区分
             then act_day 
             end) as act_days_2021,
       count(distinct case 
             when year(act_time) = 2021 
             and tag = 'exam' 
             then act_day 
             end) as act_days_2021_exam,
        count(distinct case
             when year(act_time) = 2021
             and tag = 'question'
             then act_day
             end) as act_days_2021_question
from user_info u_i
left join (select uid,
             start_time as act_time,
             date_format(start_time, '%Y%m') as act_month,
             date_format(start_time, '%Y%m%d') as act_day,
             'exam' as tag
      from exam_record
      union all 
      select uid,
             submit_time as act_time,
             date_format(submit_time, '%Y%m') as act_month,
             date_format(submit_time, '%Y%m%d') as act_day,
             'question' as tag
      from  practice_record
      ) exam_and_practice
on exam_and_practice.uid = u_i.uid
where u_i.level >= 6
group by uid
order by act_month_total desc, act_days_2021 desc
```

## 05 窗口函数

### 5.1 专用窗口函数

#### SQL136-每类试卷得分的前三名

```sql
--  两人最大分数相同，选择最小分数大者 ---> 还需计算最小分数
-- 建立子查询在提取数据

select tag,uid,ranking
from(
    select tag,uid,
    row_number() over (partition by tag order by max(score) desc,min(score) desc,uid desc) as ranking
    from examination_info left join exam_record using(exam_id)
group by tag,uid
) t1
where ranking <=3
```

#### SQL137 第二快/慢用时之差大于试卷时长一半的试卷

```sql
-- 找到第二快和第二慢用时之差大于试卷时长的一半的试卷信息，
-- 按试卷ID降序排序。

-- 找出同一张试卷第二快和第二慢的作答记录 ---> 快和慢 分别是升序和降序 建立两种排序方法
select distinct exam_id, duration ,release_time
from (
    select exam_id,duration,release_time,
    sum(case when rank1 =2 then costtime when rank2 = 2 then -costtime else 0 end) as sub
    from( -- 建立排序规则
        select exam_id,duration,release_time,
            timestampdiff(minute,start_time,submit_time) as costtime,
            row_number() over (partition by exam_id order by timestampdiff(minute, start_time, submit_time) desc) rank1,
            -- 按照作答时间的降序建立排序方式
            row_number() over (partition by exam_id order by timestampdiff(minute, start_time, submit_time) asc) rank2
            -- 按照作答时间的升序 排序
        from exam_record join examination_info using(exam_id)
        where submit_time is not null
        ) t1
    group by exam_id
    ) t2
where sub*2 >= duration -- 用时之差大于试卷时长的一半
order by exam_id desc

```

#### SQL138

请计算在2021年至少有两天作答过试卷的人中，计算该年连续两次作答试卷的最大时间窗days_window，那么根据该年的历史规律他在days_window天里平均会做多少套试卷，按最大时间窗和平均做答试卷套数倒序排序。

- lead(字段名，n) over () :取值向后偏移n行（空间的理解就是直接将一列数据往前推n个位置，后面的位置就空出来了，具体配合图片理解）;
- lag(字段名，n) over () :取值向前偏移n行（空间的理解就是直接将一列数据往前后n个位置，前面的位置就空出来了，具体配合图片理解）;
- lag(字段名，n, x) over () :取值向前偏移n行，并将空值填充为数字x（空间的理解就是直接将一列数据往前后n个位置，前面的空出来的位置用X填充上，具体配合图片理解） 。

```sql
SELECT id,score,Lead(score,2) over(order by id) lead_score,-- score数列向前推动2位，后面就腾空了2个位置
      Lag(score,2) over(order by id) lag_score, -- score数列向后推2位，腾空2个位置
      lag(score,2,666) over(order by id) lag_score_3 -- score数列向后推动2位，空值被填充为666
FROM exam_record;

```
```sql
-- 2021年至少有两天作答过试卷的人 
    -- where year(submit_time) = 2021 and count(submit_time) >= 2 错误 做过两次并不一定是在两天做的
-- 计算改年连续两次作答试卷的最大时间窗 （时间间隔？！）
    -- 连续两次作答最大时间窗为6天（1号到6号）”，时间窗的计算方法为：6号-1号+1。
    -- 相邻时间的时间差 对时间排序 row_number() over (parition by uid order by submit_time)
-- 在时间窗内天内 平均做多少套试卷 （total / diff_time）* days_window  命名为 avg_exam_cnt


with t2 as (
select uid,
	count(start_time) as total, -- 2021年总作答次数
	datediff(max(start_time),min(start_time)) +1 as diff_time, -- 头尾作答时间窗
    max(datediff(next_time,start_time)) +1  as day_window -- 最大间隔天数
from(
	select uid,start_time, -- 第二次作答时间
    lead(start_time,1) over(partition by uid order by start_time) as next_time
    from exam_record
    where year(start_time)= 2021    
	) t1
group by uid -- 计算出总作答次数 收尾作答时间窗 最大间隔天数
)
select uid,day_window,round(total * day_window/diff_time,2) as avg_exam_cnt -- 第二次作答时间for
from t2
where diff_time >1 -- 有一个时间窗的话 说明至少作答了两次and
order by day_window desc, avg_exam_cnt desc;
```



#### 

- 月份降序我们用分组连续排名。
  
  知识点：`dense_rank() over()`、`date_format()` 对每个用户ID内进行排名，因为一个月可能出现多次，所以要采用连续排名，月份大的在前面月份小的在后，符合离现在最近的月份在前。
  `dense_rank() over(partition by uid order by date_format(start_time, '%Y%m') desc) as recent_months`

```SQL
-- 找到每个人近三个有试卷作答记录的月份中没有试卷是未完成状态的用户的试卷作答完成数，
-- 按试卷完成数和用户ID降序排名。

select uid,
	count(ifnull(score,0)) as exam_complete_cnt
from (
	SELECT  uid, score, start_time,
    dense_rank() over(partition by uid order by date_format(start_time,'%Y%m') desc) as recent_month
    from exam_record    
) recent_table
where recent_month <= 3 -- 最近作答的3个月 不能使用limit 只调出3条记录
group by uid
having count(score) = count(uid) -- 作答数等于记录数 说明成绩为空的情况
order by exam_complete_cnt desc, uid desc;
```

#### SQL140 未完成率较高的50%用户近三个月答卷情况

请统计SQL试卷上未完成率较高的50%用户中，6级和7级用户在有试卷作答记录的近三个月中，每个月的答卷数目和完成数目。按用户ID、月份升序排序。

  1. SQL试卷上未完成率较高的用户
  2. 6 7级用户在有试卷作答记录的三个月中
  3. 每个月的答卷数目和完成数目
  4. 按用户ID、月份升序排序
  
```sql
SELECT t1.uid,
    DATE_FORMAT(start_time,'%Y%m')start_month,  -- 取月份数
    COUNT(start_time) total_cnt, -- 答卷数
    COUNT(submit_time) complete_cnt  -- 完成数
FROM (
    SELECT * ,
    dense_rank()over(partition by uid order by date_format(start_time,'%Y-%m') desc) time_rk -- 对作答时间进行排序
    FROM exam_record
    )t1
    
RIGHT JOIN (
    SELECT * 
    FROM (
        SELECT uid,
        PERCENT_RANK()over( ORDER BY count(submit_time)/count(start_time) ) rate_rk -- 对完成率进行分数排序
        FROM exam_record
        WHERE  exam_id IN 
        (SELECT exam_id FROM examination_info WHERE tag='SQL') -- SQL试卷
        GROUP BY uid
        ) A
    WHERE rate_rk<=0.5  -- 查找排名低于50%的用户
    AND uid IN (SELECT uid FROM user_info WHERE level IN (6,7)) -- 查找6.7级用户
    )t2 ON t1.uid=t2.uid
    
WHERE time_rk<=3 -- 查找作答时间最近的3个月
GROUP BY uid,start_month 
ORDER BY uid,start_month -- 按照用户id和月份进行升序排序
;
```

### 5.2 聚合窗口函数