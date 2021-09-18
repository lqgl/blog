---
title: 在阿里云Ubuntu服务器上安装Mysql
date: 2020-12-15
updated:
tags:
 - Ubuntu
 - Mysql
categories: Ubuntu
keywords: 'Ubuntu,Mysql,Mysql大小写敏感,部署'
description: 在阿里云Ubuntu服务器上安装Mysql,并解决Mysql8.0大小写敏感问题
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20210304111156.png
---

## 安装

### 更新数据源
```bash
apt-get update
```
### 安装 MySQL
```bash
apt-get install mysql-server
```
### 查看 MySQL 版本：
```
mysqladmin -p -u root version
```
## 解决Mysql8.0大小敏感问题
备份配置文件，卸载，删除所有数据库和MySQL相关数据。:
```
cp /etc/mysql/mysql.conf.d/mysqld.cnf /etc/mysql/mysql.conf.d/mysqld.cnf.backup
service mysql stop
apt-get --purge autoremove mysql-server
rm -R /var/lib/mysql
```
恢复保存的配置文件，编辑文件（在[mysqld]行下增加一行）。:
```
cp /etc/mysql/mysql.conf.d/mysqld.cnf.backup /etc/mysql/mysql.conf.d/mysqld.cnf
vim /etc/mysql/mysql.conf.d/mysqld.cnf

...
lower_case_table_names=1
...
```
重新安装MySQL（保留配置文件），配置其他设置:
```
apt-get install mysql-server
service mysql start
mysql_secure_installation
mysql

SHOW VARIABLES LIKE 'lower_case_%';
exit
```
![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201215130757.png)
### `mysql_secure_installation` 安全配置向导
运行mysql_secure_installation会执行几个设置：
--为root用户设置密码
--删除匿名账号
--取消root用户远程登录
--删除test库和对test库的访问权限
--刷新授权表使修改生效

### 为root用户设置密码
```
use mysql;
ALTER USER root@localhost IDENTIFIED with caching_sha2_password BY 'MyPasssword';
```

### 检查状态
```
systemctl status mysql.service
```
### 配置远程访问
* 修改配置文件
```bash
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
* 注释掉(语句前面加上 # 即可))：
```
# bind-address = 127.0.0.1
或者
bind-address = 0.0.0.0
```
* 重启 MySQL
```
service mysql restart
```
* 登录 MySQL
```
mysql -u root -p
```
* 授权 root 用户允许所有人连接
```
#创建账户
create user 'root'@'%' identified by  '你的 mysql root 账户密码';
grant all privileges on *.* to 'root'@'%' with grant option;
flush privileges;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '你的 mysql root 账户密码';
```
* 更新所有操作权限
```
flush privileges;
```
## 常用命令
### 启动
```
service mysql start
```
### 停止
```
service mysql stop
```
### 重启
```
service mysql restart
```

## 参考文档
[Mysql8.0大小写敏感](https://stackoverflow.com/questions/53103588/lower-case-table-names-1-on-ubuntu-18-04-doesnt-let-mysql-to-start/53175727#53175727)
[ubuntu下mysql8.0.21安装及修改root用户密码](https://blog.csdn.net/crazy_zh/article/details/109164552)
[SQLyog连接MySQL8报错:2058的解决方法](https://www.cnblogs.com/chengmi/p/12403761.html)
[mysql8.0版本创建账户和赋予权限](https://blog.csdn.net/shenhonglei1234/article/details/84786443?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.control)
[MySQL----mysql_secure_installation 安全配置向导](https://blog.csdn.net/qq_32786873/article/details/78846008)