# day10 搭建服务器

## 1. 数据接口跨域访问

同源：

  协议相同、域名相同、端口相同

  举例来说，<http://www.example.com/dir/page.html这个网址，协议是http://，域名是www.example.com，端口是80>（默认端口可以省略）。它的同源情况如下：

- <http://www.example.com/dir2/other.html>：同源
- <http://example.com/dir/other.html>：不同源（域名不同）
- <http://v2.www.example.com/dir/other.html>：不同源（域名不同）
- <http://www.example.com:81/dir/other.html>：不同源（端口不同）

## 2. Linux系统常用指令

Shell ---> 壳程序 ---> 人机交互环境

命令格式 ---> 命令 参数 目标对象

~ 功能键 / 快捷键

    - tab - 命令和路径的自动补全
    -  --help ---> 获取命令的帮助
    -  Ctrl+w - 删除光标前面的内容
    -  Ctrl+u - 删除光标前所有的内容
    -  Ctrl+k - 删除光标后所有的内容
    -  Ctrl+c - 终止进程

~ 基础命令：

        - clear - 清除屏幕的输出
        - history -历史命令 ---> !历史命令编号 
           -c: 清楚历史命令 
        - who / w -查询接入的用户
        -  whoami - 查看当前用户的用户名
        -  cal / date - 日历 
        -  exit / logout - 退出登录
        -  shutdown 关机或者重启
        -  man - manual -查看命令的帮助手册
        -  whatis -查看命令的描述信息
        -  whereis
        - pwd - print working directory - 打印工作目录
        - cd - change directory - 切换目录
        - ls - list directory contents - 列出目录
            -l : 长格式
            -a ： 所有文件
        - mkdir -make directory - 创建文件夹
          - p : 没有父文件夹 就自动创建父文件夹
          - cp / mv - copy / move - 复制/移动文件夹 
          -r ： 递归式
        - rm - remove - 删除
            -r ： 递归式删除
            -f ： 强制删除
            - i ： 交互式删除 
        - cat - concatenate -连接多个文件（显示文件内容）
        -n ：显示行号  
    
    - gzip / gunzip - 压缩和解压缩
    - tar - 归档和接归档  把多个文件变成一个文件
    - wget - 网络下载器
      - 下载百度网页：wget http://www.baidu.com/
    - ps - 查看进程
       - -aux /-ef ---> 查看所有进程
     - kill / pkill / killall - 结束进程
       - kill PID
       - Pkill 进程名字
    - top - 类似于windows的任务管理器
    - jobs - 查看当前会话的后台任务 --->  编号
      - 可以在命令后面使用 & 将任务放在后台运行
    - fg %编号 - 将后台任务放到前台运行
    - bg %编号 - 任务在后台运行起来
        
    - netstat - 检测网络和端口状态
      - n: 以数字的方法显示网络地址
      - l: listen 查看处于监听状态的防落服务
      - p：
      - t：基于TCP的网络服务    

## 3. Linux安装软件的方法

### 3.1 使用包管理工具

最方便最简单
使用包管理工具：CentOS / Redhat : yum / rpm

1. YUM - yellow update Modified

    - 搜索： yum search Nginx
    - 安装： yum install  -y nginx --->遇到问题一律为yes！
    - 更新：yum update nginx
    - 卸载：yum erase -y nginx
    - 查看信息：yum info nginx
    - 查看所有安装的包：yum list installed

2. RPM - Redhat Package Manager
    - 安装：rpm -ivh 软件包文件
    - 卸载：rpm -e 软件包名字
    - 查看：rpm -qa | grep 软件包

Ubuntu：apt / apt-get

### 3.1.1 安装MySQL

1. 下载安装文件

    wget <https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.37-1.el7.x86_64.rpm-bundle.tar>

3. 解归档但指定文件夹

    mkdir mysql

    tar -xf mysql-5.7.37-1.el7.x86_64.rpm-bundle.tar -C mysql

    ---> 将解归档的文件 放到上面新建的mysql文件夹中
5. 卸载mariadb

7. 补充底层依赖项
     yum install -y libaio libaio-devel
8. 利用RPM安装    ---> 按照顺序装
   - rpm -ivh mysql-community-common-5.7.37-1.el7.x86_64.rpm
   - rpm -ivh mysql-community-libs-5.7.37-1.el7.x86_64.rpm
   - rpm -ivh mysql-community-libs-compat-5.7.37-1.el7.x86_64.rpm
   - rpm -ivh mysql-community-devel-5.7.37-1.el7.x86_64.
   - rpm -ivh mysql-community-client-5.7.37-1.el7.x86_64.rpm
   - rpm -ivh mysql-community-server-5.7.37-1.el7.x86_64.rpm

9. 启动MYsql

   systemctl start mysqld
10. 查看初始密码

    cat /var/log/mysqld.log | grep password

### 3.2 源代码构建安装

---> 比较麻烦

1. 从官网网站下载python3.8

      wget <https://www.python.org/ftp/python/3.8.10/Python-3.8.10.tgz>
2. 解压缩和解归档

      gunzip Python3.8.10.tgz /后缀为xz xz -d Python-3.8.10.tar.xz

      tar -xf Python3.8.10.tar
3. 补充安装Python解释器需要的底层依赖项

   yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel libdb4-devel libpcap-devel xz-devel libffi-devel libxml2

4. 安装前的配置 ---> 出现文件Makefile

    cd Python-3.8.10

    ./configure --prefix=/usr/local/python38

5. 构建和安装

   make &&  make install

6. 配置环境变量

    cd ~   ---> 回到用户主目录

    vim .bash_profile

        ---> 10G$a (光标移到第10行末尾 在末尾进行编辑)
        ---> Path=.../usr/local/python3/bin
        ---> export PAYH
        ---> Esc ---> :wq (进入末行模式，保存退出)

    重新登录

    python3 --version

    pip3 --version

7. 创建一个符号链接（相当于Windows的快捷方式）

    ln -s /usr/local/python38/bin/python3 /usr/bin/python3

## 4.Vim的使用

Vim ---> 文本编辑神器

### 工作模式

- **命令模式**
  - 进去编辑模式： i / o / a 大小写均可
  - 命令模式里面按: ---> 进入末行模式
  - 移动光标：h向上 j 往下 k想左 l 向右 / w 光标移到下个单词
  - G 光标直接移到末尾 
    - 100G 移到第100行 
    - 显示行数：通过末行模式下输入set nu
  - 翻页： Ctrl+f 向后翻页，Ctrl+b 向前翻页
  - 删除：dd  
    - d0从光标所在处删到行首  
    -  d$ 从光标所在处删到行尾
  - 复制/黏贴：yy / p  ---> 6yy 从光标处开始复制6行
  - 撤销 / 恢复 ： u / Ctrl+r
  - 保存和退出 ：zz
- **编辑模式**
  - 回到命令模式：esc
- **末行模型**（底线命令模式）
  - 替换：1,100s/新闻/报纸  
     从1-100行把新闻替换为报纸；数字可写为$即全局替换
    - c 每一次都需要确认
    - g 全局模式
    - i 忽略大小写
  - 退出：q / qa / q! 强制退出
  - 保存：w /w! /w hello.py ---> 保存文件名为hello.pu
  - 保存退出： wq
  - 显示行号 / 隐藏行号： set nu / set nonu
  - 语法高亮：syntax on / syntax off
  - 自动缩进：set autoindent
  - 设置制表键（Tab）的空格数： set ts=4
  - 将制表键转成空格：

## 5. 源代码构建安装git

1. 下载GIt源代码

   wget <https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.32.0.tar.xz>

2. 解压 / 解归档

    - xz -d 文件名
    - tar -xvf 文件名
3. 补充依赖项

    yum install -y curl libcurl libcurl-devel

4. 安装前的配置

    cd git- 2.32.0
    ./configure --prefix=/usr/local
5. 构建和安装
    make && make install
6. 检查安装是否成功
    git --version

## 6. 云服务器

    云服务器 ---> web服务器（网站服务器） ---> Apache / Nginx
        ---> 浏览器 ---> <http://47.108.204.85>
        ---> 购买域名 ---> caozi.top
        ---> DNS解析 ---> <http://caozi.top>

    启动和停止服务：
        - 启动nginx：systemctl start nginx
        - 停止nginx： systemctl stop nginx
        - 重启nginx： systemctl restart nginx
        - 查看状态： systemctl status nginx
        - 开机启动：systemctl enable nginx
        - 禁止自启：systemctl disable nginx

启动和停止MySQL服务：

    - 启动：systemctl start mysqld
    - 停止：systemctl stop mysqld
