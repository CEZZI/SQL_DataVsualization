# day4 DCL与数据导入

DCL(数据控制语言) --->授予或者召回用户权限 ---> grant / revoke

访问者权限一定要最小化(权限刚刚给够)

## 1. 创建用户和授权

### 1.1 创建用户

#### 1.1.1 创建用户

   `create user 'guest'@'%' identified by 'Guest.618';`

    - @后面加上IP地址 指定哪些机器可以链接
    - @ '%' 表示任意的机器都能链接
    - @ 'localhost' 只有本地账户
    - 如果兼容5.7密码保护方式 则需要 `create user 'guest'@'%' identified (with mysql_native_password) by 'Guest.618';`
    - ---> 创建好账号后,连没有授权查看数据库的权限都没有

- 查找 IP地址 `cmd --->ipconfig--->10.19.219.123`

#### 1.1.2 删除用户

   `drop user if exists 'guest'@'Guest.618';`

### 1.2 授权与召回

#### 1.2.1 授权 `GRANT`

   `grant select on hrs.* to 'guest'@'%';`

   `grant insert on hrs.* to 'guest'@'%';`

- 如果想要一句话给多个权限，例如：建库建表 增删改查
   `grant create,select,insert,delete,update on hrs.* to 'guest'@'%';`

- 把对hrs数据库的所有权限都给一个账号

    `grant all privileges on hrs.* to 'guest'@'%';`
- 如果是*.* 就是对所有数据库都有所有权限 比root差一点 但是不能授权,加上with grant option 就是可以授权

    `grant all privileges on *.* to 'guest'@'%' with grant option;`

```sql
-- 游客访问
use hrs;

drop database hrs;
-- Error Code: 1044. Access denied for user 'guest'@'%' to database 'hrs'

insert into tb_dept values (50,'搞笑部','成都');
--  没有 insert权限 
-- Error Code: 1142. INSERT command denied to user 'guest'@'LAPTOP-VIMSS3AJ' for table 'tb_dept'
insert into tb_dept values (50,'电视剧部','成都');

delete from tb_dept where dno=50;
-- 没有删除权限
-- Error Code: 1142. DELETE command denied to user 'guest'@'LAPTOP-VIMSS3AJ' for table 'tb_dept'


insert into tb_dept values (60,'搞笑部','长沙');
-- 召回权限Error Code: 1142. INSERT command denied to user 'guest'@'LAPTOP-VIMSS3AJ' for table 'tb_dept'

```

#### 1.2.2 召回 `REVOKE`

   `revoke insert on hrs.* from 'guest'@'%';`

## 2. Pyton程序接入MySQL

- 三方库：
  - mysqlclient ---> C ---> 有可能因为底层C语言库的缺少而失败
  - pymysql ---> Python ---> 安装一定成功 --->import pymysql

## 2.1 步骤

- **第一步： 建立连接**

  - 连接自己 `127.0.0.0.1 or 'localhost'`
  - `host` - 主机 -确定连接哪一台服务器 （IP地址或者主机名）
  - `port` - 端口- 确定连接符服务器上的哪个服务（端口是用来区分不同的服务的）
  - `user password` - 用户名和口令(提示：一般不能用超级管理员账号)
  - `database` - 连接的数据库，如果如要连接两个数据库 需要建立两个连接  

- **第二步：获得游标对象** --->用完后要关闭连接对象

- **第三步：通过游标向数据库服务器发出SQL语句**，获取执行结果
  - 不允许字符串拼接 要是用安全的占位符
  - `insert into tb_dept values(%s,%s,%s)', (no, name, location)`

- **第四步：提交上面的操作**（成功）/ (回滚) ---> 关闭连接

### 2.2 增删改

- 捕获异常
  - `except`指定的异常类型的父类型，那么都可以捕获到子类型的异常
  - 异常捕获需要遵循面向对象编程中的里式替换原则
  - 里式替换(`LSP -Liskov Substitution Principle` )：任何时候都可以用子类型替换掉父类型，父类型未必能替换子类型



```Python
import pymysql

no = int(input('部门编号：'))
name = str(input('部门名称：'))
location = str(input('部门部门所在地：'))

# 第一步：建立连接
conn = pymysql.connect(host='localhost', port=3306,
                       user='guest', password='Guest.618',
                       database='hrs', charset='utf8mb4')
print(conn)  # 通过输出结果，检测是否连接成功

try:
    # 第二步：获得游标对象 --->用完后要关闭连接对象
    with conn.cursor() as cursor:

        # 第三步：通过游标向数据库服务器发出SQL语句，获取执行结果
        affected_rows = cursor.execute(
            'insert into tb_dept values(%s,%s,%s)', (no, name, location)
        )
        # 如果想要删除 'delete from tb_dept where dno = %s',no
        # 执行后 返回一个整数 ---> 多少行受影响
        if affected_rows == 1:
            print('添加部门成功！')

    # 第四步：提交上面的操作（成功）
    conn.commit()
except pymysql.MySQLError as err:  # 捕获mysql所有错误的父类型
    print(err)
    # 第四步：回滚（操作失败）
    conn.rollback()
finally:
    conn.close()
```

- 开启事务环境
  
  不会直接删除，可以利用rollback进行返回
```SQL
begin;

delete from tb_emp where dno = 20;
commit;  -- 成功了就提交
rollback; -- 失败了回滚
```

### 2.3 查找select

  - 不在需要   `rollback()`
  - `fetchone() / fetchall() / fetchmany()`

```SQL
# 第三步：通过游标指定SQL
        cursor.execute('select dno,dname,dloc from tb_dept')
        # 第四步：通过游标抓取数据（查询结果）
        print(cursor.fetchone()) --抓取一行
        print(cursor.fetchall()) -- 抓取全部
```
`fetchall()` 十分占用内存空间,数据体量很大时不建议使用,建议一行一行抓取，可以采用如下两种方法

```sql
row = cursor.fechmany(3)
        while row:
            print(row)
            row = cursor.fetchone()
```




## 3. 数据中导出数据到Excel

- `enumerate()` 用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，**同时列出数据和数据下标**。
- 如果希望不新建一张表，把默认的sheet激活 `表.active`
- openpyxl操作一个Excel，行和列的索引都是从1开始的

```python
import pymysql
import openpyxl

conn1 = pymysql.connect(host='localhost', port=3306,
                        user='guest', password='Guest.618',
                        database='hrs', charset='utf8mb4')
try:
    with conn1.cursor() as cursor:
        cursor.execute(
            'select eno,ename,job,sal,dname from tb_emp t1 inner join tb_dept t2 on t1.dno=t2.dno'
        )
        # 创建工作簿 openpyxl操作一个Excel，行和列的索引都是从1开始的
        wb = openpyxl.Workbook()
        ws = wb.create_sheet('员工表')  # 新建一张员工表
        titles = ('工号', '姓名', '职位', '月薪', '部门')
        for col_idx, col_name in enumerate(titles):
            ws.cell(1, col_idx + 1, col_name)

        for row_idx, emp_row in enumerate(cursor.fetchall()):
            # enumerate 直接取出列出数据和数据下标 是下表和数据的二元组
            for col_idx, col_value in enumerate(emp_row):
                ws.cell(row_idx + 2, col_idx + 1, col_value)
        wb.save('人力资源管理.xlsx')

except pymysql.MySQLError as err:
    print(err)
finally:
    conn1.close()
```


## 4. Excel数据读入数据库中

### 4.1 使用批量处理

`executemany` 里面是批量的元组
```sql
-- 建立表
-- 从Excel文件中读取数据写入数据库
create database stock default character set utf8mb4;
use stock;

create table tb_babastock
(
stock_id bigint unsigned auto_increment comment '编号',
trade_date date not null comment '交易日',
high_price decimal(12,4) not null comment '最高价',
low_price decimal(12,4) not null comment '最低价',
open_price decimal(12,4) not null comment '开盘价',
close_price decimal(12,4) not null comment'收盘价',
trade_volume bigint unsigned not null comment '交易量',
primary key ( stock_id)
);
```

```Python

import openpyxl
import pymysql

wb = openpyxl.load_workbook('阿里巴巴2020年股票数据1.xlsx')
ws = wb.active
params = []
for row_idx in range(2, ws.max_row + 1):
    values = []
    for col_idx in range(1, ws.max_column):
        values.append(ws.cell(row_idx, col_idx).value)
    params.append(values)

conn1 = pymysql.connect(host='localhost', port=3306,
                        user='guest', password='Guest.618',
                        database='stock', charset='utf8mb4')


try:
    with conn1.cursor() as cursor:
        # 执行批量插入操作
        cursor.executemany(
            'insert into tb_babastock '
            '(trade_date, high_price,low_price,open_price,close_price,trade_volume) '
            'values '
            '(%s,%s,%s,%s,%s,%s)',
            params
        )
    conn1.commit()
except pymysql.MySQLError as err:
    print(err)

finally:
    conn1.close()
```
