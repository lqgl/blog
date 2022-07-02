---
title: Springboot集成FlyWay
date: 2020-12-24
updated:
tags:
 - SpringBoot
 - FlyWay
categories: SpringBoot
keywords: 'SpringBoot,Mysql,FlyWay,数据库版本管理'
description: Springboot集成FlyWay，使用FlyWay进行数据库版本管理
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20201224182958.png
---
## 依赖
在pom.xml添加maven依赖
```
<!-- mysql 依赖 -->
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
</dependency>

<!-- 数据库版本管理 依赖 -->
<dependency>
<groupId>org.flywaydb</groupId>
<artifactId>flyway-core</artifactId>
</dependency>

<!-- 数据库访问依赖 -->
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```
```
<plugin>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-maven-plugin</artifactId>
</plugin>
```
## 配置 `application.yml`
在application.ym文件添加Flyway配置
需要先配置数据源（连接上数据库），可以自己换成druid

```
# 数据源
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true&serverTimezone=PRC&useSSL=false
    username: root
    password: 密码
    # MySQL 8.x以下: com.mysql.jdbc.Driver
    driver-class-name: com.mysql.cj.jdbc.Driver
---
# FLYWAY (FlywayProperties)
spring:
  flyway:
    url: jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC
    user: root
    password: 密码
    baseline-on-migrate: true #在没有元数据表的情况下，针对非空Schema执行迁移时是否自动调用基线。
```
### 在resource目录下创建db/migration目录添加sql脚本
![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201224132145.png)
手动创建 db/migration 目录，然后在该目录下创建数据库脚本，数据库脚本的命名方式如下：

```
V<VERSION>__<NAME>.sql #注意：是两条下划线
#比如V1_0_0__Create_Table_User.sql
```
首先是大写字母 V，然后是版本号，要是有小版本可以用下划线隔开，例如 2_1，版本号后面是`两个下划线`，然后是脚本名称，文件后缀是 `.sql`。
### 验证是否成功:
项目启动时,会运行flyway执行sql语句.生成schema_version表,用于记录sql执行情况.
去数据库`test`看`flyway_schema_history`表
![](https://raw.githubusercontent.com/lqgl/repo/main/img/20201224132524.png)

## 其他配置
```
spring:
  flyway:
    enabled: true #是否开启 flyway，默认就是开启的
    url: jdbc:mysql://127.0.0.1:3306/mygame?serverTimezone=UTC # 地址
    user: root #用户名
    password: 365472 #密码
    baseline-on-migrate: true #在没有元数据表的情况下，针对非空Schema执行迁移时是否自动调用基线。
    encoding: UTF-8 #flyway 字符编码
    locations: classpath:db/migration #sql 脚本的目录，默认是 classpath:db/migration，如果有多个，用 , 隔开
    clean-disabled: true #这个属性非常关键，它表示是否要清除已有库下的表，如果执行的脚本是 V1__xxx.sql，那么会先清除已有库下的表，然后再执行脚本，这在开发环境下还挺方便，但是在生产环境下就要命了，而且它默认就是要清除，生产环境一定要自己配置设置为 true。
    table: flyway_schema_history #配置数据库信息表的名称，默认是 flyway_schema_history。
```

## 参考文档
[SpringBoot整合Flyway](https://blog.csdn.net/qq_41402200/article/details/89247317)
[简化Spring Boot项目部署，Flyway搞起来](https://baijiahao.baidu.com/s?id=1659024978275677262&wfr=spider&for=pc)