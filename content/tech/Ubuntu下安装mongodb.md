---
title: Ubuntu20.04安装MongoDB
date: 2021-03-08
updated: 
tags:
 - Ubuntu
 - MongoDB
categories: Ubuntu
keywords: 'Ubuntu,MongoDB'
description: Ubuntu20.04环境下安装MongoDB
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20210304113111.png
---

本教程描述了如何在Ubuntu20.04上安装MongoDB4.4

## 安装MongoDB

Ubuntu 20.04默认存储库中不提供最新版本的MongoDB，因此需要在系统中添加官方的MongoDB存储库。

### 首先安装gnupg软件包

```sh
sudo apt-get install gnupg
```

### 导入包管理系统使用的公钥

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```

### 添加MongoDB存储库

```sh
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```

### 更新存储库

```sh
sudo apt-get update
```

### 使用以下命令安装MongoDB

```sh
sudo apt install mongodb-org
```

### 启动MongoDB服务

启动MongoDB，同时使用以下命令使其在重新启动时启动

```sh
sudo systemctl start mongod.service
sudo systemctl enable mongod
```

### 检查MongoDB服务的状态

```sh
sudo systemctl status mongod
```

输出内容如下：

```sh
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2020-12-20 19:51:14 PST; 3min 22s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 5818 (mongod)
     Memory: 58.5M
     CGroup: /system.slice/mongod.service
             └─5818 /usr/bin/mongod --config /etc/mongod.conf

12月 20 19:51:14 admin systemd[1]: Started MongoDB Database Server.
```

### 验证安装是否成功完成

```sh
mongo --eval 'db.runCommand({ connectionStatus: 1 })'
```

## 配置MongoDB

安装MongoDB后，默认的配置文件位于 `/etc/mongod.conf` ，我们可以通过编辑该文件进行相应的配置。

```sh
vim /etc/mongod.conf
```

编辑MongoDB配置文件后，重新启动mongod服务以使更改生效：

```sh
sudo systemctl restart mongod
```

## 相关命令

检查MongoDB服务状态：

```sh
sudo systemctl status mongod
```

关闭MongoDB服务：

```sh
sudo systemctl stop mongod
```

重新启动MongoDB服务：

```sh
sudo systemctl restart mongod
```

关闭开机启动：

```sh
sudo systemctl disable mongod
```

进入MongoDB shell：

```sh
mongo
```

## 参考文档

[Ubuntu20.04安装MongoDB](https://www.cnblogs.com/bubbleboom/p/14167409.html)