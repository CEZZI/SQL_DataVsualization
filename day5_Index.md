# day5 索引视图和函数过程

## 1. 执行计划和索引

### 1.1 执行计划查询 `explain`
  
  **在代码前面加上explain获取查询效率**,而不是查询结果

  `explain select eno,ename,job from tb_emp where ename='张三丰';`

在SQL执行计划中，有几项值得我们关注:

1. `select_type`:查询的类型。

   - SIMPLE:简单SELECT，不需要使用UNION操作或子查询。
   - PRIMARY:如果查询包含子查询，最外层的SELECT被标记为PRIMARY。
   - UNION: UNION操作中第二个或后面的SELECT语句。
   - SUBQUERY:子查询中的第一个SELECT.
   - DERIVED:派生表的SELECT子查询。
  
2. `table`:查询对应的表。
3. `type`: MySQL在表中找到满足条件的行的方式，也称为访问类型，

   包括:ALL(全表扫描) 、index(索引全扫描，只遍历索引树)、range(索引范围扫描)、ref(非唯一索引扫描) 、eq_ref(唯一索引扫描)、const/system(常量级查询)、NULL(不需要访问表或索引)在所有的访问类型中，很显然ALL是性能最差的，它代表的全表扫描是指要扫描表中的每一行才能找到匹配的行。
4. `possible_keys`: MySQL可以选择的索引，但是有可能不会使用。
5. `key`: MySQL真正使用的索引，如果为NULL就表示没有使用索引。
6. `key_len`:使用的索引的长度，在不影响查询的情况下肯定是长度越短越好。
7. `rows`:执行查询需要扫描的行数,这是一个预估值。
8. `extra`:关于查询额外的信息。

    - Using filesort : MySQL无法利用索引完成排序操作。
    - Using index: 只使用索引的信息而不需要进一 步查表来获取更多的信息。
    - Using temporary: MySQL需要使用临时表来存储结果集，常用于分组和排序。
    - Impossible where: where 子句会导致没有符合条件的行。
    - Distinct: MySQL发现第一 个匹配行后，停止为当前的行组合搜索更多的行。
    - Using where: 查询的列未被索引覆盖，筛选条件并不是索引的前导列。
  
### 1.2 索引 Index

1. 建立索引

   `create index ide_emp_ename on tb_emp(ename);`

```SQL
CREATE 
  [UNIQUE -- 唯一索引
  | FULLTEXT -- 全文索引
  ] 
  INDEX index_name ON table_name -- 不指定唯一或全文时默认普通索引
  (column1[(length) [DESC|ASC]] [,column2,...]) -- 可以对多列建立组合索引  

-- 在duration列创建普通索引idx_duration、在exam_id列创建唯一性索引uniq_idx_exam_id、在tag列创建全文索引full_idx_tag。
create index idx_duration on examination_info(duration);
CREATE unique index uniq_idx_exam_id on examination_info(exam_id);
create fulltext index full_idx_tag on examination_info(tag);
```

2. 删除索引

   `drop index ide_emp_ename on tb_emp;`

   删除索引无需写上索引的类型，只有创建索引才需要

3. 前缀索引
 
    `create index idx_emp_ename on tb_emp(ename(1));`
   - 回表现象：

    索引没有覆盖到所有的列，那么通过索引后定位到数据之后，
    还需要做一次回表（再查一次聚集索引），才能获得所有的数据 ---> 覆盖索引

4. 覆盖索引:索引已经覆盖到了所有想获取的所有列

     `create index idx_emp on tb_emp(ename,job);` 把两个列复合起来

   复合索引：遵循最左匹配，如果左边的索引没法使用，则右边的索引就失效了

**我们简单的为大家总结一下索引的设计原则:**

1. 最适合索引的列是出现在WHERE子句和连接子句中的列。

2. 索引列的基数越大(取值多重复值少)，索引的效果就越好。

3. 使用前缀索引可以减少索引占用的空间，内存中可以缓存更多的索引。

4. 索引不是越多越好，虽然索引加速了读操作(查询)， 但是写操作(增、删、改)都会变得更慢，因为数据的变化会导致索引的更新，就如同书籍章节的增删需要更新目录一样。

5. 使用InnoDB存储引擎时，表的普通索引都会保存主键的值，所以主键要尽可能选择较短的数据类型，这样可以有效的减少索引占用的空间，利用提升索引的缓存效果。

最后，还有一点需要说明，InnoDB使用的B-tree索引， 数值类型的列除了等值判断时索引会生效之外，使用>、<、>=、<=、BETWEEAN.... <>时，索引仍然生效;对于字符串类型的列，如果使用**不以通配符开头的模糊查询**，索引也是起作用的，但是其他的情况会导致索引失效，这就意味着很有可能会做全表查询。

## 2. 视图

`crete viewc XX`
`select show view`
`drop view XX`

- 一次查询的快照，限制用户的权限，不能直接查表
- 常用复杂的查询，可以直接写成视图方便查询
- 只要有权限，就能通过修改来视图背后的表,

  `update vw_emp set ename='胡二刀' where eno=1359;`

既然视图是一张虚拟的表， 那么视图的中的数据可以更新吗?视图的可更新性要视具体情况而定，以下类型的视图是不能更新的:

1. 使用了聚合函数(SUM、MIN、MAX、AVG、COUNT等)、DISTINCT、 GROUP BY、HAVING、UNION或者UNION ALL的视图。
2. SELECT中包含了子查询的视图。
3. FROM子句中包含了-一个不能更新的视图的视图。
4. WHERE子句的子查询引用了FROM子句中的表的视图。

**有原始数据表跟视图对应的可以更新，通过聚合、运算、临时表得到的视图都不能更新**
  
```sql
create view vw_emp as
select eno,ename,job,mgr,dno from tb_emp;


create view vw_dept_emp_count as 
select dname,ifnull(total,0) from tb_dept t1 left join
(
select dno, count(*) as total from tb_emp group by dno
) t2
on t2.dno=t1.dno;
```

## 3. 函数

### 3.1 定义函数的语法

    delimiter $$ 
    create function 函数名(自变量 类型) returns 类型
    begin
        ...;
        ...;
    return 因变量;
    end $$

```SQL
delimiter $$
-- 函数里面的每一句话都需要写分号
-- 原先写分号是表示整个代码写完
-- delimiter 表示整个代码结束以$$为终止符

create function truncate_string(x varchar(16383)) 
returns varchar(16383) on sql
begin
 declare result varchar(16383) default '';  -- result 用于保存返回值
    if char_length(x) > 50 then
  set result = concat(substring(x,1,50),'___');
 else
  set result = x;
 end if;
    return result;
end $$

delimiter ; -- 回复现场 用完要改回去
```

## 4.过程(存储过程)

存储在数据那边提前已经编译好的一组SQL语句，直接调用过程，就相当于执行了这一组SQL语句。

- 创建过程
`create procedure 过程名()`

- 调用过程
`call 过程名();`



```SQL
delimiter $$

create procedure uppgrade_sal()
begin
	update tb_emp set sal=floor(sal+sal*0.05) where dno=10;
    update tb_emp set sal=floor(sal+sal*0.2) where dno=20;
    update tb_emp set sal=floor(sal+sal*0.1) where dno=30;
commit;
end $$

delimiter ;


select * from tb_emp;
call uppgrade_sal();
```

- in & out 输入参数和输出参数

利用out来输出参数，默认就输入参数。通过变量out把计算出平均工资带出去
```SQL
delimiter $$

create procedure get_avg_sal(
	in dept_no integer unsigned,
    out avg_sal decimal(10,2)
)
begin
	SELECT avg(sal) into avg_sal from tb_emp where dno=dept_no;
end $$

delimiter ;

call get_avg_sal(20,@a);
select @a;
```