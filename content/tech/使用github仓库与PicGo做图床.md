---
title: 使用 github repo + PicGo 搭建博客图床
date: 2020-12-14
updated:
tags:
 - Github
 - PicGo
categories: Blog
keywords: 'Github,PicGo'
description: 使用 github repo + PicGo 搭建博客图床
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20201214110203.png
---

## 创建github repo,并生成token

### 创建仓库

   ![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214102438.png)

### 生成token

进入个人中心设置

![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214102542.png)

进入`Developer settings`中，选择`Personal aceess token`,然后`Generate new token`.

![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214102805.png)

生成 token(Note 部分随便写即可，下边的权限把 `repo` 相关的打上勾即可)

![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214103050.png)

接下来便会生成一个 token，将它复制下来，因为一旦刷新网页，你将见不到这个 token 了。

## 下载安装 PicGo

### 下载并安装PicGo
[PicGo仓库](https://github.com/Molunerfinn/PicGo)。
## Github图床配置

![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214102145.png)
### Github图床配置
PicGo的Github图床配置有固定的格式:
仓库名格式为:`用户名/仓库名`
分支名设置为:`master`
Token设置为:上面获取到的`token`
存储路径可以随意,如设置为:`img/`
自定义域名格式为:

```
https://raw.githubusercontent.com/用户名/仓库名/分支名
```

## githubPlus插件配置

`githubPlus`可以同步相册,当在`PigGo`的相册中**删除图片后,会同步删除`github`上的图片**.

### 同步更新删除Github

在`插件设置`的搜索框中,输入插件的名称:`picgo-plugin-github-plus`,然后安装这个插件.
![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214103627.png)

### 配置githubPlus

这个插件的配置和上面的Github图片配置差不多一样.
![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214103806.png)

## PicGo其他配置

### 上传快捷方式

进入PicGo设置，修改上传快捷键(像这样，配合[Snipaste](https://www.snipaste.com/)食用更佳，每次`F1`截完图，选择复制，然后按绑定的组合键即可实现快速上传（快捷键上传的是剪切板中的东西）)

![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201214104157.png)

## 参考文档
[PicGo GitHub图床](https://lanlan2017.github.io/blog/b19c6a80/)
[配置picgo成为多平台图床工具](https://www.antmoe.com/posts/c9c6437b/index.html)

