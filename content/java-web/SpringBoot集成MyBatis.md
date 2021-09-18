---
title: SpringBoot集成MyBatis
date: 
updated: 
tags:
 - SpringBoot
 - MyBatis
categories: SpringBoot
keywords: 'SpringBoot,MyBatis'
description: SpringBoot集成MyBatis
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20210304105052.png
---
## 概述

**MyBatis**是一个[Java](https://zh.wikipedia.org/wiki/Java)[持久化框架](https://zh.wikipedia.org/wiki/持久化框架)，它通过[XML](https://zh.wikipedia.org/wiki/XML)描述符或注解把[对象](https://zh.wikipedia.org/wiki/对象_(计算机科学))与[存储过程](https://zh.wikipedia.org/wiki/存储过程)或[SQL](https://zh.wikipedia.org/wiki/SQL)语句关联起来，映射成数据库内对应的纪录。与其他[对象关系映射](https://zh.wikipedia.org/wiki/对象关系映射)框架不同，MyBatis没有将[Java](https://zh.wikipedia.org/wiki/Java)[对象](https://zh.wikipedia.org/wiki/对象_(计算机科学))与[数据库](https://zh.wikipedia.org/wiki/数据库)表关联起来，而是将Java方法与[SQL](https://zh.wikipedia.org/wiki/SQL)语句关联。MyBatis允许用户充分利用[数据库](https://zh.wikipedia.org/wiki/数据库)的各种功能，例如存储过程、[视图](https://zh.wikipedia.org/wiki/视图)、各种复杂的查询以及某数据库的专有特性。

## 引入依赖

在 `pom.xml` 文件中引入 `mapper-spring-boot-starter` 依赖，该依赖会自动引入 MyBaits 相关依赖

```xml
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
```

## 配置 `application.yml`

配置 MyBatis

```xml
mybatis:
  type-aliases-package: com.monoya.cake.business.model  # 实体类的存放路径
  mapper-locations: classpath:mapper/*.xml # mapper.xml的存放路径
```

