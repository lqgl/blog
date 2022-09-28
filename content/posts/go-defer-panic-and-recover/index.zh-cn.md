---
title: "Defer Panic and Recover"
date: 2022-09-16T09:49:15+08:00
lastmod: 2022-09-28T18:27:16+08:00
draft: false
author: "lqgl"
description: ""
resources:
- name: "featured-image"
  src: "images.png"

tags: ["Go"]
categories: ["Go Blog"]

lightgallery: true

toc:
 auto: false
---

英文原文: [Defer, Panic, and Recover](https://go.dev/blog/defer-panic-and-recover)

## 介绍
Go有常见的控制流机制：if、for、switch、goto。它也有go语句来运行单独的goroutine中的代码。这里我想讨论一些不太常见的机制：defer、panic和recover。

### defer
defer语句将一个函数调用推到一个列表中。保存的调用列表在周围的函数返回后被执行。defer通常被用来简化执行各种清理动作的函数。

例如，让我们看看一个打开两个文件并将一个文件的内容复制到另一个文件的函数:
```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }

    written, err = io.Copy(dst, src)
    dst.Close()
    src.Close()
    return
}
```

这可以工作，但有一个错误。如果对os.Create的调用失败，该函数将在没有关闭源文件的情况下返回。这可以通过在第二个返回语句之前调用 src.Close 来轻松解决，但如果这个函数更复杂，问题可能就不会那么容易被注意和解决了。通过引入defer语句，我们可以确保文件始终被关闭:
```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

延迟语句允许我们考虑在打开每个文件后立即关闭它，确保无论函数中返回语句的数量如何，文件都将被关闭。

延迟语句的行为是直接的和可预测的。有三个简单的规则:

    1. 当对defer语句求值时，对deferred函数的实参进行求值。

在本例中，当Println调用被延迟时，表达式“i”被求值。延迟调用将在函数返回后打印“0”。
```go
func a() {
    i := 0
    defer fmt.Println(i)
    i++
    return
}
```

    2. 延迟的函数调用在周围的函数返回后按照后进先出的顺序执行

这个函数输出"3210"：
```go
func b() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i)
    }
}
```    

    3. 延迟函数可以读取并赋值给返回函数的命名返回值。

在本例中，延迟函数在周围的函数返回后将返回值i加1。因此，这个函数返回2:
```go
func c() (i int) {
    defer func() { i++ }()
    return 1
}
```    

这便于修改函数的错误返回值;稍后我们将看到一个这样的例子。

### Panic
Panic 是一个内置函数，它会停止正常的控制流程，开始panic。当函数F调用panic时，F的执行将停止，F中的任何defer函数将正常执行，然后F返回给调用者。对于调用者来说，F的行为就像是在呼叫panic。该过程沿着堆栈向上继续，直到当前goroutine中的所有函数都返回，此时程序将崩溃。panic可以通过直接调用panic而引发。它们也可能由运行时错误引起，例如越界数组访问。

### Recover
Recover 是一个内置函数，它可以重新控制一个处于panic状态的goroutine。Recover只在延迟函数中有用。在正常执行期间，调用恢复将返回nil，没有其他效果。如果当前goroutine处于panic状态，则调用recover将捕获给panic的值并恢复正常执行。

下面是一个演示panic和defer机制的示例程序:
```go
package main

import "fmt"

func main() {
    f()
    fmt.Println("Returned normally from f.")
}

func f() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in f", r)
        }
    }()
    fmt.Println("Calling g.")
    g(0)
    fmt.Println("Returned normally from g.")
}

func g(i int) {
    if i > 3 {
        fmt.Println("Panicking!")
        panic(fmt.Sprintf("%v", i))
    }
    defer fmt.Println("Defer in g", i)
    fmt.Println("Printing in g", i)
    g(i + 1)
}
```

函数g接受int i，如果i大于3，它会发生panic，否则它调用自身，参数是i+1。函数f延迟调用recover的函数并打印恢复的值(如果它是非空值)。在继续阅读之前，试着想象一下这个程序的输出可能是什么。

该程序将输出：
```go
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
Recovered in f 4
Returned normally from f.
```

如果从f中删除deferred函数，则panic没有恢复，并且到达goroutine的调用堆栈的顶部，从而终止程序。修改后的程序将输出:
```go
Calling g.
Printing in g 0
Printing in g 1
Printing in g 2
Printing in g 3
Panicking!
Defer in g 3
Defer in g 2
Defer in g 1
Defer in g 0
panic: 4

panic PC=0x2a9cd8
[stack trace omitted]
```
### More

关于panic和recover的真实示例，请参阅Go标准库中的[json包](https://go.dev/pkg/encoding/json/)。它用一组递归函数对接口进行编码。如果在遍历值时发生错误，将调用panic以将堆栈展开到顶层函数调用，该函数从panic中恢复并返回适当的错误值(请参阅[encode.go](https://go.dev/src/pkg/encoding/json/encode.go)中encodeState类型的' error '和' marshal '方法)。

Go库中的惯例是，即使一个包在内部使用panic，它的其他外部API仍然显式的错误返回值。

defer 的其他用途（除了前面给出的file.Close例子之外）包括释放一个mutex：
```go
mu.Lock()
defer mu.Unlock()
```

打印页脚：
```go
printHeader()
defer printFooter()
```
以及更多。

总之，defer语句(带或不带panic和recover)为控制流提供了一种不同寻常的强大机制。它可以用于建模由其他编程语言中的特殊目的结构实现的许多特性。试一下。
