---
title: "gRPC Tutorial"
date: 2023-03-04T21:53:13+08:00
lastmod: 2023-03-04T21:53:13+08:00
draft: false
toc: false
images:
tags:
  - gRPC
---

## 简介

`gRPC` 是来自`Google`的一个开源`RPC`框架，速度快，执行率高，基于`HTTP/2`构建，低延迟, 支持流，与开发语言无关，并且可以简单的插入身份认证、负载均衡、日志和监控等功能。

![](https://grpc.io/img/landing-2.svg)

## 如何学习gRPC

先学`Protocol Buffers`https://developers.google.com/protocol-buffers/,用它来定义消息和服务。

然后，你只需要实现服务即可，剩余的`gRPC`代码将会自动为你生成。

`.proto` 文件适用于十几种开发语言（包括服务端和客户端），并且它允许你使用同一个框架来支持每秒百万级以上的RPC调用。

## 开发模式

gRPC使用的是合约优先的API开发模式，它默认使用`Protocol buffers`(也叫protobuf)作为接口设计语言（IDL），这个`.proto`文件包括两部分：

- gRPC服务的定义

- 服务端和客户端之间传递的消息

## 为什么使用Protocol Buffers? 

它和开发语言无关

可以生成所有主流开发语言的代码

数据是二进制格式的，串行化的效率高，Payload比较小

也适合传递大量数据

通过设定某些规则，使得API的进化也很简单

## 学习使用Protocol Buffer

[Protocol Buffer](/writing/2023/protocol-buffers)

## gRPC结构

![](./grpc-structure.png)

## gRPC生命周期

![](./grpc-lifecycle.png)

## 身份认证

`这里指的不是用户的身份认证`，而是指多个`server`和`client`之间，它们如何识别出来谁是谁，并
且能安全的进行消息传输。

在身份认证这方面，`gRPC`一共有`4种身份认证的机制`:

- 不采取任何措施的连接，也就是不安全的连接。

- TLS/SSL连接。
- 基于Google Token的身份认证。
- 自定义的身份认证提供商。

## 消息传输类型

`gRPC`的消息传输类型有`4种`:

- 第一种是一元的消息，就是简单的`请求`-`响应`。
- 第二种是server streaming (流)，server 会把数据streaming回给client。
- 第三种是client streaming, 也就是client会把数据streaming给server。
- 最后是双向streaming。

### 消息类型

当`gRPC`使用`Protocol Buffer`作为传输协议的时候，`Protocol Buffer`里所有的规则仍然都适用。

但是在`gRPC使用Protocol Buffer的时候`，会添加一些额外的规则和语法，以便让`gRPC`能和它完美配合。

## 生成gRPC代码

```bash
$ protoc --go_out=. --go-grpc_out=. *.proto
```

## gRPC Demo

[gRPC Demo](https://github.com/lqgl/grpc-demo)

## 参考

[plugins are not supported : grpc](https://github.com/golang/protobuf/issues/1070)

[gRPC in C#*2/Go/C++](https://www.bilibili.com/video/BV1eE411T7GC/)
