# 一、牛客网刷题 —— 企业真题

## 1. 某音短视频

### 1.1 计算各类视频的平均播放进度，将进度大于60%的类别输出

- 播放进度=播放时长÷视频时长*100%，当播放时长大于视频时长时，播放进度均记为100%。
- 结果保留两位小数，并按播放进度倒序排序。
- 需要转换为百分号的形式

```sql
-- 自己写的答案 没有实现百分号
-- 利用concat做字符串拼接后 是否还能实现比较？
-- end_time - start_time差过1分钟的话，它的进制是100，不是60。
-- 而timestampdiff(second,start_time,end_time)进制得到的是60

-- 答案也不对 多一个 旅游 
-- 原因： 不能直接用时间相减
select b.tag,
    concat(round(sum(if(end_time - start_time >= duration,1,
     (end_time-start_time)/duration)) / count(start_time),2) * 100,'%')as avg_play_progress

from tb_user_video_log a left join tb_video_info b
    on a.video_id = b.video_id
group by b.tag
having avg_play_progress > 0.6
order by avg_play_progress desc;
```

```sql
-- 正确答案
select tag, concat(avg_play_progress,'%')  as avg_play_progress
from (
select b.tag as tag,
   round(avg(
    if(TIMESTAMPDIFF(second, start_time,end_time) > duration,1,
    TIMESTAMPDIFF(second,start_time,end_time) / duration)
    ) * 100,2) as avg_play_progress 
from tb_user_video_log a  join tb_video_info b
    on a.video_id = b.video_id
group by b.tag
having avg_play_progress > 60
order by avg_play_progress desc
) as t_1;
```

### 1.2 统计在有用户互动的最近一个月

（按包含当天在内的近30天算，比如10月31日的近30天为10.2~10.31之间的数据）中，每类视频的转发量和转发率（保留3位小数）。

- 从由互动的最后一天 往前倒推30天
  - timestampdiff ？！ ×
- 转发量 count(if_retweet): 转发量/播放量
- 找到最近播放的日期：max(date(start_time)) ---> 需要转化为日期格式
- 往过去推移30天: date_sub(max(start_time)),inrerval 30 day)
- 筛选最近的：where date(start_time) > (select date_sub(max(start_time)),inrerval 30 day) from tb_user_video_log)

```sql
select
  tag,
  sum(if_retweet) as retweet_cnt, 
  round(retweet_cnt / count(start_time), 3) as retweet_rate
  -- 不能直接调用上面输出的结果
from
  tb_user_video_log
  join tb_video_info using(video_id)
where
  DATE(start_time) > (
    SELECT
      DATE_SUB(MAX(DATE(start_time)), INTERVAL 30 DAY)
    FROM
      tb_user_video_log
  )
group by
  tag
order by
  retweet_rate desc;

select
  tag,
  sum(if_retweet) as retweet_cnt,
  round(sum(if_retweet) / count(start_time), 3) as retweet_rate
from
  tb_user_video_log
  join tb_video_info using(video_id)
where
  DATE(start_time) > (
    SELECT
      DATE_SUB(MAX(DATE(start_time)), INTERVAL 30 DAY)
    FROM
      tb_user_video_log
  )
group by
  tag
order by
  retweet_rate desc;

```

### 1.3 每个创作者每月的涨粉率及截止当前的总粉丝量

计算2021年里每个创作者每月的涨粉率及截止当月的总粉丝量

- 涨粉率=(加粉量 - 掉粉量) / 播放量
- 涨粉为1 脱粉为2 不能简单的用if 来做 利用case when
- group by author 月份呢？
- 数据格式的显示：date_format(start_time,'%Y-%m')

本题最大的难点在于累加粉丝数，这个地方要用窗口函数才能实现累加功能。

使用窗口函数的时候要注意：

author是大类，月份是累加单位，所以partition by author, order by date
两层sum是因为第一个sum是针对每一个月内进行计算，第二个sum是为了每一个author在不同月份的累加。

```sql
select author,
    date_format(start_time,'%Y-%m') month,
    round(sum(case when if_follow=1 then 1
                   when if_follow=2 then -1
                   else 0 end)/count(author),3) fans_growth_rate,
    sum(sum(case when if_follow=1 then 1
                 when if_follow=2 then -1
                 else 0 end)) 
        over (partition by author order by date_format(start_time,'%Y-%m')) total_fans
 from tb_user_video_log 
 left join tb_video_info using(video_id)
 where year(start_time)=2021
 group by author,month
 order by author,total_fans
```

### 1.4 国庆期间每类视频点赞量和转发量 （困难）

**问题**：统计2021年国庆头3天每类视频每天的近一周总点赞量和一周内最大单天转发量，结果按视频类别降序、日期升序排序。假设数据库中数据足够多，至少每个类别下国庆头3天及之前一周的每天都有播放记录。

1. 先按天进行聚合统计

因为原数据是以天为单位的统计数据，每一天都会有多条if_like和if_retweet记录，所以先要按照tag，date进行统计，得到每天的总点赞量like_cnt，和总转发量retweet_cnt

2. 滑动窗口的设置(ROWS BETWEEN CURRENT ROW AND 6 PRECEDING)

思路：在09.25-10.03这个区间内，按tag聚合，dt逆序，统计得到CURRENT ROW及后6行的点赞量统计sum_like_cnt_7d，和转发量sum_retweet_cnt_7d

3. 记录的筛选

在外面再套一层SELECT，取出所有字段，按照tag, dt聚合，HAVING限定日期为10月1号到3号，按照题目要求排序就大功告成啦。

```sql
select * 
from(
    select tag,dt,
    sum(like_cnt) over w sum_like_cnt_7d,
    max(retweet_cnt) over w sum_retweet_cnt_7d
    from (
        select tag,date(start_time) dt,
            sum(if_like) like_cnt,
            SUM(if_retweet) retweet_cnt
        from tb_video_info
        left join tb_user_video_log using(video_id)
        where date(start_time) between  '2021-09-25' AND '2021-10-03'
    group by 1,2) t1
  WINDOW w AS (PARTITION BY tag ORDER BY dt DESC ROWS BETWEEN CURRENT ROW AND 6 FOLLOWING)
) t2
GROUP BY 1, 2
HAVING dt BETWEEN '2021-10-01' AND '2021-10-03'
ORDER BY 1 DESC, 2
```

### 1.5 近一个月发布的视频中热度最高的top3视频

找出近一个月发布的视频中热度最高的top3视频。结果中热度保留为整数，并按热度降序排序。

1. 热度最高

   热度=(100视频完播率+5点赞数+3评论数+2转发数)*新鲜度

   - 视频完播率 sum(if(end_time-start_time-duration>=0,1,0))/count(video_id)

   - 点赞数 sum(if_like)

   - 评论数 sum(if(comment_id is not null,1,0))

   - 转发数  sum(if_retweet)

   - 新鲜度：新鲜度=1/(最近无播放天数+1)
    - 当播放次数为0时，最近无播放天数=当前日期-发布日期；

    - 当播放次数不为0时，最近无播放天数=当前日期-最近一次播放日期
     -
     - `select max(end_time) from tb_user_video_log`

2. top3 视频
   limit函数： limit 0,3

3. 近一月

  排序窗口函数 datediff(最新日期-发布日期)<=29

如果想把(select max(end_time) from tb_user_video_log) 拿出来单独作为变量 只能放在 from 后面 不如直接进行嵌套

```SQL
SELECT video_id,
       round((100*finish_rate+5*like_index+3*comment_index+2*retweet_index)/(fresh_index+1),0) hot_index -- 计算热度
FROM(
    SELECT a.video_id,
           sum(if(timestampdiff(second,a.start_time,a.end_time)-b.duration>=0,1,0))/count(a.video_id) finish_rate,
           sum(a.if_like) like_index,
           sum(if(a.comment_id is not null,1,0)) comment_index,
           sum(a.if_retweet) retweet_index,
           if(count(a.video_id)=0,datediff(date((select max(end_time) from tb_user_video_log)),date(b.release_time)),
              datediff(date((select max(end_time) from tb_user_video_log)),max(date(a.end_time)))) fresh_index
            -- 如果没有播放记录： 现在时间-发布，如果有播放记录：现在时间-最近一次播放时间        
    FROM tb_user_video_log a
        join tb_video_info b 
        on a.video_id=b.video_id
        -- 需要过滤掉没有播放记录的视频 使用内连接
    where DATEDIFF(date((select max(end_time) from tb_user_video_log)),DATE(b.release_time))<=29
        -- 计算最近一个月内 从现在时间开始 往前递推 29天
    group by a.video_id
    ) fir_sheet
order by hot_index DESC
limit 0,3

```

## 2. 用户增长场景（某度信息流）

### 2.1 2021年11月每天的人均浏览文章时长

统计2021年11月每天的人均浏览文章时长（秒数），结果保留1位小数，并按时长由短到长排序

- 人均浏览时长：round(sum(timestampdiff(second,in_time,out_time))/count(distinct uid),1)
  - 人均 除以 所有阅读的uid数
  - 使用timestampdiff方便计算秒数 而不是 datediff
- 利用 date（） 直接转换数据格式

**timestampdiff()函数与datediff()函数的使用**

1. timestampdiff()函数的作用是返回两个日期时间之间的整数差。
   而datediff()函数的作用也是返回两个**日期值**之差。

它们的函数语法分别为：

TIMESTAMPDIFF(unit,datetime_expr1,datetime_expr2)

DATEDIFF(expr1,expr2)

2. 由于TIMESTAMPDIFF可返回两个日期时间的小时差、月份差和年份差，因此第一个参数可取hour\month\year等参数。

```SQL
select date(in_time) as dt,
   round(sum(timestampdiff(second,in_time,out_time))/count(distinct uid),1) as  avg_viiew_len_sec
from tb_user_log
where date_format(in_time,"%Y-%m") = "2021-11" and artical_id  != 0
-- 2021年11月 阅读文章的人
group by dt
order by avg_viiew_len_sec;
```

### 2.2 每篇文章同一时刻最大在看人数

问题：统计每篇文章同一时刻最大在看人数，如果同一时刻有进入也有离开时，先记录用户数增加再记录减少，结果按最大人数降序。

- 如何实现+1 和-1 
  
1. 对原表编码并联立；

2. 按artical_id维度，dt升序 ，diff降序，对diff进行SUM开窗统计，得到每个artical_id的瞬时观看人数 instant_viewer_cnt；

3. 最外层SELECT按artical_id聚合，通过MAX（instant_viewer_cnt）取出瞬时观看最大值max_uv，并排序。

```sql
select artical_id,
   max(instant_viewer_cnt)  as max_uv
from 
(
    select artical_id, 
    sum(diff) over (partition by artical_id order by dt,diff desc) instant_viewer_cnt
    -- SUM窗口函数，按文章id维度，统计按时间戳升序的观看人数变化情况
    from 
    (
    -- 编码+联立
    -- 某篇文章artical_id，在给定的时间戳dt的，瞬时观看人数变化diff
    -- 原表in_time和out_time进行编码，in为观看人数+1， 
    -- out为观看人数-1，进行两次SELECT联立，并按artical_id升序，时间戳升序    
        select artical_id,in_time dt, 1 diff
        from tb_user_log
        where artical_id != 0
        union all
        select artical_id,out_time dt, -1 diff
        where artical_id != 0
    ) t1
) t2
group by 1
order by 2 desc;

```

### 2.3 2021年11月每天新用户的次日留存率

1. 查询出每个客户的第一次登录时间
   
   ```sql
   select uid,
    min(date(in_time)) as dt
  from tb_user_log
  group by uid;
   ```
2. 利用并集考虑到跨天活跃（第一天登录第二天才离开）
   
   ```sql
   select uid, date(in_time) dt from tb_user_log
   union
   select uid, date(out_time) from tb_user_log
  
   ```
3. 表1 和表2 右链接
4. 根据日期分组计算次日留存率

```sql
select t1.dt,round(count(t2.uid)/count(t1.uid),2) uv_rate
from (select uid
      ,min(date(in_time)) dt
      from tb_user_log 
      group by uid) as t1  -- 每天新用户表
left join (select uid , date(in_time) dt
           from tb_user_log
           union
           select uid , date(out_time)
           from tb_user_log) as t2 -- 用户活跃表
on t1.uid=t2.uid
and t1.dt=date_sub(t2.dt,INTERVAL 1 day)
where date_format(t1.dt,'%Y-%m') = '2021-11'
group by t1.dt
order by t1.dt

```

### 2.4 统计活跃间隔对用户分级结果

统计活跃间隔对用户分级后，各活跃等级用户占比，
结果保留两位小数，且按占比降序排序。

嵌套的子查询的次数很多，需要查很多次

0. 计算每个用户的首次登录的时间最近一次登录的时间

```sql
select min(date(in_time)) as first_dt,
       max(date(out_time) as last_dt),
group by uid
```

1. 计算最早最晚活跃里当前 天数差

- 计算当前日期(最近的一次退出日期) max(date(out_time)) as cur_dt
- 统计总用户数：count(distanct uid) as user_cnt
- 最早活跃胡距当前 天数差 timestampdiff(day,first_dt,cur_dt) as first_dt_diff
- 最近一次活跃日期距当前 天数差： timestampdiff(day,last_dt,cur_dt) as last_dt_diff

2. 用户分级


  - 忠实用户(近7天活跃过且非新晋用户) ---> 难以表示 
  - 新晋用户(近7天新增)
  - 沉睡用户(近7天未活跃但更早前活跃过)
  - 流失用户(近30天未活跃但更早前活跃过)

```SQL
CASE 
  WHEN last_dt_diff >=  30 then '流失用户',
  when last_dt_diff >= 7 then '沉睡用户',
  when frist_dt_diff < 7 then '新晋用户'
  else '忠实用户'
end as user_grade
```
3. 各活跃等级用户占比
   
  在实现
   

```sql
select user_grade,round(count(uid)/max(user_cnt),2) as ratio
from 
    (
    select uid,user_cnt,
    case 
        when last_dt_diff >= 30 then '流失用户'   -- 计算用户等级
        when last_dt_diff >=7 then "沉睡用户"
        WHEN first_dt_diff < 7 THEN "新晋用户"
        ELSE "忠实用户"
    END as user_grade
    from 
        (
        select uid,user_cnt,  --  计算用户的首次登录和最近一次登录距离当前日期的时间差
            timestampdiff(day,first_dt,cur_dt) as first_dt_diff, 
            timestampdiff(DAY,last_dt,cur_dt) as last_dt_diff
        from
            (
            select uid,min(date(in_time)) as first_dt,
            max(date(out_time)) as last_dt
            from tb_user_log
            group by uid
            ) as t_uid_first_last --  计算用户的首次登录时间和最近一次登录时间 
        left join
        (
        select max(date(out_time)) as cur_dt, -- 计算当前日期
            count(distinct uid) as user_cnt -- 计算总用户数
            from tb_user_log
        ) as t_overall_info on 1
    ) as t_user_info
) as t_user_grade 
group by user_grade
order by ratio desc;
```


### 2.5  每天的日活数及新用户占比

新用户占比=当天的新用户数÷当天活跃用户数（日活数）。
如果in_time-进入时间和out_time-离开时间跨天了，在两天里都记为该用户活跃过。
新用户占比保留2位小数，结果按日期升序排序。

1. 建立用户信息活跃表

跨天登录记为 在登录时间和退出时间都活跃了 使用 union

```sql
select uid,date(in_time) dt
        from tb_user_log
        union all
        select uid,date(out_time) dt
        from tb_user_log
        group by uid,dt
```

2. 计算新用户数

```sql
select uid,min(date(in_time)) dt
          from tb_user_log
          group by uid
```

```SQL
select a.dt,count(distinct a.uid) dau,
    round(count(distinct b.uid)/count(distinct a.uid),2) -- 计算新用户占比
from 
        (select uid,date(in_time) dt
        from tb_user_log
        union all
        select uid,date(out_time) dt
        from tb_user_log
        group by uid,dt
         ) a  -- 用户活跃表
left join
         (select uid,min(date(in_time)) dt
          from tb_user_log
          group by uid
         ) b -- 每个用户的首次登录时间
on a.uid=b.uid and a.dt=b.dt
group by a.dt
order by a.dt;

```

### 2.6 连续签到领金币

计算每个用户2021年7月以来每月获得的金币数（该活动到10月底结束，11月1日开始的签到不再获得金币）。结果按月份、ID升序排序。


从2021年7月7日0点开始，用户每天签到可以领1金币，并可以开始累积签到天数，连续签到的第3、7天分别可额外领2、6金币。
每连续签到7天后重新累积签到天数（即重置签到天数：连续第8天签到时记为新的一轮签到的第一天，领1金币）

## 3. 电商场景（某东商城）

## 4. 出行场景（某滴打车）

## 5. 某宝店铺分析（电商模式）

## 6. 牛客直播课分析（在线教育行业）

## 7. 某乎问答（内容行业）

sum(if_like)


