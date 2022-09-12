# day6 Git

## 1. 版本控制历史

版本控制 ---> CASE工具中非常重要的一个工具
            Computer Aided Software Engineering

Unix ---> MINIX --> Linux(1991) ---> 社会haunt编程 ---> 版本控制
                        ---> Android+

2005 ---> Git ---> Linux
     ---> 合并模式 / 分布式版本控制系统

## 2. 使用Git

### 2.1 Git Bash

敲的是 Linux命令
Shell常用的命令

        - `clear` ---> 清空屏幕
        - `pwd` - print working directory ---> 打印当前工作目录
        - `ls` - list directory contents ---> 列出文件夹下的内容
        - `-a` ---> all ---> 列出所有（包括隐藏文件和文件夹）
        - `-l` ---> long ---> 长格式
        - `mkdir` - make directory  ---> 创建文件夹
        - `-p` ---> parents ---> 自动创建父级文件夹
        - `cd` -change directory --->切换目录
          - 相对路径：以当前文件夹为基础 `cd Desktop/git0326`
            - `cd ..`：去到当前目录
            - `cd .`：去到上一级目录 
          - 绝对路径：以根目录文件夹为基础 `cd /Users/HP/Desktop/git026`
        - `rm` -remove ---> 删除（文件/文件夹）
        - `-r` ---> recursive ---> 递归式删除（可以删除文件夹）
        - `-f` ---> force ---> 强制删除

使用Git：

- `git init`  ---> 将本地文件夹初始化为版本控制的本地仓库
- `git add` ---> 将文件从工作区放到仓库的缓存区（暂存区）
  - `git add .` ---> 把所有文件都放到暂存区
- `git rm --cached 文件` ---> 将文件从暂存区中移除
- `git status` ---> 查看版本控制的状态（特别重要）
第一次使用git执行commit操作前，需要先配置用户名和邮箱
  - `git config -- global user.name CEZZI`
  - `git config -- global user.email`  

- `git commit -m '...提交的原因'`--->提交（将文件同步到本地仓库）
- `git log` ---> 查看提交的日志（版本控制的历史日志）
  - `git reflog` ---> 查看日志（可以看到未来的版本（比Head更加靠后的版本））

- `git restore` ---> 从仓库内容恢复工作区
  - 如果没有执行git add: `git restore <filename>`
  - 如果已经执行了git add：`git restore --staged <filename>`
- `git reset` ---> 版本重置（版本回退）
  - `git reset --hard 版本号` --->  仓库、暂存区、工作区都退回到指定的版本
  - `git reset --mixed 版本` ---> 默认为mixed，仓库、暂存区都退回到指定的版本，工作区不回退
  - `git reset --soft 版本号` ---> 仓库回退到指定的历史版本，暂存区和工作区不回退
- `git remote -v` ---> 查看远端仓库（企业Git私有服务器）
  - `git remote add origin https://gitee.com/haveniceday/git0326.git`  远端仓库别名为 origin

- `git pull` ---> 下拉(将远端仓库同步到本地仓库)
- `git clone` ---> 克隆项目（从服务器项目下载到Be本地）
  - `git clone --depth 1`  <https://gitee.com/bravojimoon/git0326> 只要最新的版本

本地：

    1. git init
    2. git add .
    3. 第一次使用git做commit操作之前，要先配置用户名和邮箱
    4. git commit -m ''
    5. git log

远端：

    1. git remote add origin 仓库地址 / git remote remove origin / git remote -v
    2. git push -u origin master
    3. git pull 下拉
    4. git clone 仓库地址 别名

## 3. 配置免密访问Git远端仓库

配置免密访问：

    - 在本地创建密钥对（加密和解密不是同一个密钥，公钥和私钥）
        ssh-keygen -t rsa -b 2048 -C "1226127750@qq.com"
    - 免密中的地址要是用 ssh地址
