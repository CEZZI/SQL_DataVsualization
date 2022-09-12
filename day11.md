# day11 项目部署上线

项目部署上线过程：

1. 克隆项目到云服务器
   git clone --depth 1 <https://gitee.com/jackfrued/data_viz_2103.git>

2. 启动和初始化数据库
 
   systemctl start mysqld
   mysql -u root -p </root/data_viz_2103/order.sql
   mysql -u root -p </root/data_viz_2103/stock.sql
   然后输出密码 ---> 连接mysql mysql -u root -p

```sql
--创建用户
mysql> create user 'guest@localhost' identified by 'Guest.618';
-- 给用户权限
mysql> grant select on stock.* to 'guest@localhost';
mysql> grant select on data_viz.* to 'guest@localhost';
```

3. 修改Nginx配置文件

把静态资源的路径考下来 pwd ---> /root/data_viz_2103/static --->找到nginx.conf

- 利用vim编辑配置文件
vim nginx.conf
vim /etc/nginx/nginx.conf   ---> nginx默认的配置（不要随意更改）
  - 第五行中 改为user root；
  - 38-54行 改为 http{server{root/root/data_viz_2103/static;}}

4. 启动Nginx

systemctl start nginx
systemctl restart nginx

5. 创建虚拟环境

- 安装virtualenv工具: pip3 install virtualenv  --下载创建虚拟环境的工具

- 查看版本：virtualenv --version
- 去到项目的主目录  cd data_vi
- 建立虚拟环境并重命名为enev: virtualenv --python=/usr/bin/python3 enev
- 激活虚拟环境：source venv/bin/activate  
- 安装依赖项: pip install -r requirements.txt
 

6. 启动Gunicorn

   - gunicorn -w 4 -b 127.0.0.1 main:app

    #CPU是几核的就写数字几
    - export DB_USER=guest
    - export DB_PASS=Guest.618

7. 修改Nginx配置并重启

- vim /etc/nginx/nginx.conf

8. 域名绑定

   购买域名：<http://wangwang.aliyun.com/>
   域名解析：A ---> @ ---> 云服务器公网IP地址
   通过备案系统备案

9. 配置HTTP(了解)
    - 购买域名
    - 修改Nginx的配置文件 ---> pass

---> 重启
