---
title: "Docker 安装卸载与镜像容器管理"
date: 2022-09-29T09:25:18+08:00
lastmod: 2022-09-29T09:25:18+08:00
draft: false
toc: true
images:
tags:
  - 运维
  - Docker
---
## Docker 安装与卸载

### 安装与验证
1. 更新数据源：
```bash
sudo apt update
```

2. 安装一些必备软件包，让 apt 通过 HTTPS 使用软件包。
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

3. 将官方 Docker 版本库的 GPG 密钥添加到系统中：
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

4. 将 Docker 版本库添加到APT源：
```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

5. 用新添加的 Docker 软件包来进行升级更新。
```bash
sudo apt update
```
6. 确保要从 Docker 版本库，而不是默认的 Ubuntu 版本库进行安装：
```bash
apt-cache policy docker-ce  # 执行后可以看到 docker-ce 来自 Docker 官方版本库。
```

7. 安装 Docker
```bash
sudo apt install docker-ce
```

8. 检查 Docker 是否正在运行：
```bash
sudo systemctl status docker
```

### 卸载 Docker
在卸载 Docker 之前，你最好移除所有的容器，镜像，卷和网络。
运行下面的命令停止所有正在运行的容器，并且移除所有的 docker 对象
```bash
docker container stop $(docker container ls -aq)
docker system prune -a --volumes
```

接下来你可以使用 apt 命令来卸载 Docker
```bash
sudo apt purge docker-ce
sudo apt autoremove
```

## Docker 删除容器和镜像

### 管理容器
```bash
docker ps -aq # 列出所有容器 ID
docker ps -a # 查看所有运行或者不运行容器
docker stop $(docker ps -a -q) # 停止所有的 container（容器），这样才能够删除其中的 images
docker rm $(docker ps -a -q) # 如果想要删除所有 container（容器）的话再加一个指令
docker container prune # 删除所有停止的容器
docker stop Name或者ID  # 停止一个容器
docker start Name或者ID  # 启动一个容器
docker kill Name或者ID  # 杀死一个容器
docker restart name或者ID # 重启一个容器
```

### 管理镜像
```bash
docker images # 查看当前有些什么 images
docker rmi imageid # 删除 images（镜像），通过 image 的 id 来指定删除谁
docker rmi $(docker images | grep "^<none>" | awk "{print $3}") # 想要删除 untagged images，也就是那些 id 为的 image 的话可以用
docker rmi $(docker images -q) # 要删除全部 image（镜像）的话
docker rmi -f $(docker images -q) # 强制删除全部 image 的话
```


