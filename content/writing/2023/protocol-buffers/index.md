---
title: "Protocol Buffers"
date: 2023-03-05T23:27:43+08:00
lastmod: 2023-03-05T23:27:43+08:00
draft: false
toc: false
images:
tags:
  - gRPC
---

## 开发环境

Vscode的拓展插件：`vscode-proto3` 和 `Clang-Format`.

Windows还需要安装`Clang`, [Clang for Windows 64-bit](https://releases.llvm.org/download.html); 

Mac: `brew install clang-format`

## 前提

### 标量类型
- `数值型`, 数值型有很多种形式：double，float，int32，int64，uint32，uint64，sint32，sint64，fixed32，fixed64，sfixed32，sfixed64.根据需要选择对应的数值类型

- `布尔型`, bool 型可以有true和false两个值
- `字符串`，string表示任意长度的文本，但是它必须包含的是UTF-8编码或7位ASCII的文本，长度不可超过2^32。
- `字节型`，bytes可表示任意的byte数组序列，但是长度也不可以超过2^32，最后是由你来决定如何解释这些bytes。例如你可以使用这个类型来表示一个图片。

### 字段的数值（Tag）

在Protocol Buffers里面，字段的名字其实没那么重要，但是在写C#代码时，字段名还是很重要的。

对于protobuf来说，这个`tag`是更为重要的。

可以使用的最小的tag数值是1，最大值是 2^29  - 1，或者536,870,911。但是你不可以使用`19000`到`19999`之间的数，这部分数是保留的。

还有 一点值得注意的是：

从1到15的Tag数只占用1个字节的空间，所有它们应该被用在频繁使用的字段上。而从16到2047，则占用2个字节，它们可以用在不频繁使用的字段上。

### 保留的字段

如果你对你的消息类型进行了更新，例如删除某个字段或者注释掉某个字段，那么其他开发者在以后更新这个消息类型的时候可能会重新使用被你删除/注释掉的字段的数值（tag）。如果以后还需要使用这个消息类型的老版本的proto文件，那么这将会引起严重的问题，例如数据损坏、隐私漏洞等等。

那么一种避免此类事情发生的解决办法就是将你删除/注释掉的这些字段的数值（或/并且包括字段名，因为字段名可引起JSON序列化的问题）标记为`reserved`，如果其他人再使用这个数值作为字段标识符，那么编译器就会有错误提示。

### 字段的默认值

当消息被解析的时候，如果编码的消息里不含有特定的的一个singular元素，那么在被解析对象里相应的字段就会被设为默认值。

常用类型的默认值如下：

- string: 空字符串

- bytes：空的byte数组

- bool：false

- 数值型：0

- 枚举enum：枚举里定义的第一个枚举值，值必须是0

- repeated：通常是相应开发语言里的空list


### 枚举

枚举里面定义的第一个值就是这个枚举的默认值。

Enum的tag必须从0开始，所以0就是枚举的数值默认值。

枚举里面的常量的值必须不能超过32位整型的数值，不建议使用负数。

枚举可以定义在message里面，也可以在外边单独定义以便复用。如果另一个消息想使用Person里面这个Gender枚举，那么可以使用Person.Gender这种形式。

### 使用其它的信息类型

可以使用自定义的message类型作为字段的类型。

### 引入定义

如果想要使用的信息类型已经在其它的proto文件定义好了呢？这个时候就需要引入信息类型的定义。

```protobuf
import "xxx.proto"
```

### 打包

你可以向proto文件添加可选的打包（package）说明符，以避免消息类型间的名称冲突。

```protobuf
syntax = "proto3"; // 使用 proto3 语法
option go_package = "protobuf/;proto";
// 具体语法为：option go_package = "{out_path};out_go_package";
// 前一个参数用于指定生成文件的位置，后一个参数指定生成的.go件的package
package proto; 
// 表示当前protobuf文件属于import包，但这个package不是Go语言中的那个package
// 这个package主要在导入外部proto文件时用到,下面是例子：
import "person.proto"; 
// 导入后则通过被导入文件包名.结构体名使用。
```
#### 完整例子

目录结构如下：

```sh
pb$ tree
├── protocol
│       ├── compoent.pb.go
│       └── computer.pb.go
├── computer.proto
└── component.proto
```

`component.go`

```protobuf
syntax = "proto3";
option go_package = "protocol/;proto";
package component;

message CPU {
  string Name = 1;
  int64 Frequency = 2;
}
```

`computer.proto`

```protobuf
syntax = "proto3";
option go_package = "protocol/;proto";
package computer;
import "component.proto";

message Computer {
  string name = 1;
  component.CPU cpu = 2;
}
```

`main.go`

```go
pb := proto.Computer{}
```

### 更新消息类型的规则

不要修改任何现有字段的数字(tag)

你可以添加新的字段，那些使用旧的消息格式的代码仍然可以将消息序列化，您应该注意这些元素的默认值，以便新代码可以与旧代码生成的消息正确交互。类似的，新代码所创建的消息也可以被旧代码解析:旧的二进制在解析的时候会忽略新的字段。

字段可以被删除，只要它们的数字(tag) 在更新后的消息类型中不再使用即可。你也可以把字段名改为使用“OBSOLETE ”前缀而不是删除字段，或者把这些字段的数字(tag) 进行保留(reserved)，以免未来其它开发者不小心使用了删除字段的数字。

对于数据类型的变化，例如int32到int64，string到bytes等等， 可以参考官方文档:
https://developers.google.com/protocol-buffers/docs/proto3#updating。但是建议还是尽量不要
去修改字段的数据类型。

#### 默认值

默认值在更新Protocol Buffer消息定义的时候有很重要的作用，它可以防止对现有代码/新代码造成破坏性影响。它们也可以保证字段永远不会有null值。

但是，默认值还是非常危险的:

`你无法区分这个默认值到底是来自一个丢失的字段还是字段的实际值正好等于默认值`。

应该怎么办?

需要保证这个默认值对于业务来说是一个毫无意义的值。例如int32 pop (人口)默认值就可
以设置为-1。

再就是，可能需要在你的代码里来做一些对默认值的判断，从而进行处理。

#### 枚举

enum同样可以进化，就和消息的字段一样，可以添加、删除值，也可以保留值。

但是如果代码不知道它接收到的值对应哪个enum值，那么enum的默认值将会被采用。

## 设置Protocol Buffer编译器

1. protoc编译器主要就是用来生成代码的，[下载地址](https://github.com/protocolbuffers/protobuf/releases/)
2. 运行以下命令安装Go protocol buffers插件:
    ```bash
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    ```
3. 生成代码
    ```bash
    $ protoc --go_out=. *.proto
    ```

## 参考

[protocol buffer消息定义](https://www.bilibili.com/video/BV1eE411T7GC?p=2&vd_source=c0bd412f4e3efd5d80e196d81c024209)

[Language Guide (proto 3) | Protocol Buffers Documentation](https://protobuf.dev/programming-guides/proto3/)

[gRPC 官方文档中文版_V1.0 (oschina.net)](http://doc.oschina.net/grpc?t=60133)

[protobuf教程(一)---引入其他proto文件](https://www.lixueduan.com/posts/protobuf/01-import/)