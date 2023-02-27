---
title: "Portainer Docker可视化工具"
subtitle: ""
date: 2022-12-14T10:03:14+08:00
lastmod: 2022-12-14T10:03:14+08:00
draft: false
toc: true
comments: true
tags:
  - 运维
  - Docker
---

Portainer是一个可视化的Docker操作界面，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。

### 前提条件

安装好 Docker Desktop版本，后续内容也是基于Docker的Desktop版本的。

### 安装

1. 创建用于存储Portainer服务器数据库的卷

```docker
docker volume create portainer_data
```

2. 下载并安装Portainer服务器容器

```docker
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

这里安装的Portainer社区版最新版, 更详尽的内容在[官网](https://docs.portainer.io/start/install/server/docker/wsl)。

### 登录

```
https://localhost:9443
```

*注*：访问的必须是https协议。

原因如下：[client-sent-an-http-request-to-an-https-server](https://portal.portainer.io/knowledge/client-sent-an-http-request-to-an-https-server)
