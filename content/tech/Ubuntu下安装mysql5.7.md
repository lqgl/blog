---
title: Ubuntu20.04安装和配置MySQL5.7
date: 2021-03-08
updated: 
tags:
 - Ubuntu
 - Mysql
categories: Ubuntu
keywords: 'Ubuntu,Mysql5.7'
description: Ubuntu20.04安装和配置MySQL5.7
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20210304111156.png
---

## Ubuntu换源

### 更新数据源
备份原来的sorce文件
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.old
```
修改sources.list文件
```
sudo vim /etc/apt/sources.list
```
选择的是清华镜像源。将sources.list内容清空，然后选择一个源粘贴到sources.list，保存退出。
```
# 清华镜像源
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
```
更新镜像源和软件
```
# 更新镜像源
sudo apt-get update
```
### apt-get install 方式安装 MySQL
由于Mysql5.7有许多依赖
```
# 执行下面安装命令

# 安装libtinfo5
sudo apt-get install libtinfo5

# 安装mysql-client-core-5.7
sudo apt-get install mysql-client-core-5.7 

# 安装mysql5.7客户端
sudo apt-get install mysql-client-5.7 

# 安装mysql5.7服务端
sudo apt-get install mysql-server-5.7
 
```
选择配置项，设置root密码
### 安装完成后查看mysql版本和服务
```
mysql -V # 查看mysql版本
netstat -tap | grep mysql # 查看mysql服务
```

### 检查状态
```bash
sudo netstat -tap | grep mysql
```

## 配置远程访问

- 修改配置文件

```text
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

- 注释掉(语句前面加上 # 即可)：

```text
bind-address = 127.0.0.1
```

- 重启 MySQL

```text
service mysql restart
```

- 登录 MySQL

```text
mysql -u root -p
```

- 授权 root 用户允许所有人连接

```text
grant all privileges on *.* to 'root'@'%' identified by '你的 mysql root 账户密码';
flush privileges;
重启mysql服务
```

### 因弱口令无法成功授权解决步骤

- 查看和设置密码安全级别

```text
select @@validate_password_policy;
```

```text
set global validate_password_policy=0;
```

- 查看和设置密码长度限制

```text
select @@validate_password_length;
```

```text
set global validate_password_length=1;
```

## 远程连接mysql遇到的问题

```
Can't connect to MySQL server on 'x.x.x.x' (10060)
```
### 解决方法：
服务器配置安全组，开启相应的端口号

## 常用命令

### 启动

```text
service mysql start
```

### 停止

```text
service mysql stop
```

### 重启

```text
service mysql restart
```

## 其它配置

修改配置 `mysqld.cnf` 配置文件

```text
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

### 配置默认字符集

在 `[mysqld]` 节点上增加如下配置

```text
[client]
default-character-set=utf8
```

在 `[mysqld]` 节点底部增加如下配置

```text
default-storage-engine=INNODB
character-set-server=utf8
collation-server=utf8_general_ci
```

### 配置忽略数据库大小写敏感

在 `[mysqld]` 节点底部增加如下配置

```text
lower-case-table-names = 1
```
# 卸载mysql：
```
1.sudo apt-get autoremove mysql* --purge
2.sudo apt-get remove mysql-server
3.sudo apt-get remove mysql-common
 
# 清理残留数据 
sudo dpkg -l |grep mysql|awk '{print $2}' |sudo xargs dpkg -P 
sudo rm -rf /etc/mysql/ 
sudo rm -rf /var/lib/mysql
 
# 检查是否删除完毕
whereis mysql
sudo find / -name mysql
在终端中查看MySQL的依赖项：
dpkg --list|grep mysql
```
### 终端无法退格：

```
# apt-get install ncurses-base
# apt-get install ncurses-term
然后reboot一下就可以了
```

## 参考文档

[Linux 安装 MySQL](https://www.funtl.com/zh/linux/Linux-%E5%AE%89%E8%A3%85-MySQL.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)

