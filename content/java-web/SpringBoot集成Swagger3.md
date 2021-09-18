---
title: SpringBoot集成Swagger3.0.0
date: 
updated: 
tags:
 - SpringBoot
 - Swagger
categories: SpringBoot
keywords: 'SpringBoot,Swagger'
description: SpringBoot集成Swagger
top_img: https://raw.githubusercontent.com/lqgl/repo/main/img/top_img.jpg
cover: https://raw.githubusercontent.com/lqgl/repo/main/img/20210304110042.png
---

## 概述

Swagger 是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。

总体目标是使客户端和文件系统作为服务器以同样的速度来更新。文件的方法、参数和模型紧密集成到服务器端的代码，允许 API 来始终保持同步。

**3.0版本在配置上与2.9稍有差别，包括依赖包改为: springfox-boot-starter，启用注解更改为: `@EnableOpenApi`等。**

[swagger 3.0 release notes](https://github.com/springfox/springfox/releases)

## 引入依赖

在 `pom.xml` 文件中引入 `springfox-boot-starter` 依赖，该依赖会自动引入 `Swagger` 相关依赖

```xml
            <dependency>
                <groupId>io.springfox</groupId>
                <artifactId>springfox-boot-starter</artifactId>
                <version>3.0.0</version>
            </dependency>
```

## 自定义配置类

创建一个 Swagger3 配置类, `SwaggerConfig`

```java
/**
* Swagger配置类
*/
@Configuration
@EnableOpenApi
public class SwaggerConfig {
    @Bean
    public Docket docket(){
       return new Docket(DocumentationType.OAS_30)
                .apiInfo(apiInfo()).enable(true)
                .select()
                //apis： 添加swagger接口提取范围(contoller层包名)
                .apis(RequestHandlerSelectors.basePackage("com.example.troller"))
                //.apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }
    
    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("XX项目接口文档")
                .description("XX项目描述")
                .contact(new Contact("作者", "作者URL", "作者Email"))
                .version("1.0")
                .build();
    }
}
```

## 在你的 Controller 上添加 swagger 注解

```java
@Api(tags="用户管理")
@RestController
@RequestMapping("/user")
public class UserController {
    private static Logger logger = LoggerFactory.getLogger(UserController.class);

    @Resource
    UserService userService;
    
	@ApiOperation("查询所有用户")
    @GetMapping("/getUser4Page")
    public List<User> getUser4Page(){
        logger.info("开始查询所有用户");
        return userService.getUser4Page();
    }
    ...
}
```

## 如启用了访问权限，还需将 swagger相关 uri 允许匿名访问

具体需要添加的uri有：

```
/swagger**/**
/webjars/**
/v3/**
/doc.html
```

## 启动应用，访问 `http://ip:port/swagger-ui/index.html`

![image-20210304095648667](https://raw.githubusercontent.com/lqgl/repo/main/img/image-20210304095648667.png)

## 参考文档

[spring boot 集成 swagger 3.0 指南](https://segmentfault.com/a/1190000037455077)

