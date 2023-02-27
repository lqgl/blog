---
title: "Go Slice"
date: 2022-09-16T09:38:33+08:00
lastmod: 2020-12-14T11:04:51+08:00
draft: false
toc: true
comments: true
tags:
- Go Blog
---
英文原文:[Go Slices: usage and internals](https://go.dev/blog/slices-intro)

## 介绍
Go 的切片类型为处理类型化数据的序列提供了一种方便而有效的方法。切片类似于其他语言中的数组，但有一些不寻常的特性。本文将探讨什么是切片以及如何使用切片。

## 数组(Arrays)
切片类型是建立在Go的数组类型之上的一个抽象，因此要理解切片，我们必须先理解数组。

一个数组类型定义指定了一个长度和一个元素类型。例如，[4]int类型表示一个包含四个整数的数组。一个数组的大小是固定的；它的长度是其类型的一部分（[4]int和[5]int是不同的，不兼容的类型）。数组可以用常规方式进行索引，所以表达式s[n]可以访问第n个元素，从0开始。

```go
var a [4]int
a[0] = 1
i := a[0]
// i == 1
```

数组不需要显式初始化，数组的零值是一个随时可以使用的数组，其元素本身已经归零:
```go
// a[2] == 0, the zero value of the int type
```

[4]int的内存表示只是四个按顺序排列的整数值。
![](https://go.dev/blog/slices-intro/slice-array.png)

Go 的数组是数值。一个数组变量表示整个数组；它不是指向第一个数组元素的指针（在C语言中是这样的）。这意味着当你分配或传递一个数组的值时，你将复制其内容。(为了避免复制，你可以传递一个指向数组的指针，但那是一个指向数组的指针，而不是一个数组。) 考虑数组的一种方法是作为一种结构，但具有索引而不是命名字段：一个固定大小的复合值.

数组字面值可以这样指定：
```go
b := [2]string{"Penn", "Teller"}
```

或者，您可以让编译器为您计算数组元素:
```go
b := [...]string{"Penn", "Teller"}
```

在这两种情况下，b的类型都是[2]string

## 切片(Slices)
数组有自己的位置，但它们有点不灵活，所以在Go代码中不太常见。然而，切片随处可见。它们以数组为基础，提供强大的力量和便利。

切片的类型规范是[]T，其中T是切片元素的类型。与数组类型不同，切片类型没有指定的长度。

slice 字面量的声明就像数组字面量一样，只是省略了元素计数：
```go
letters := []string{"a", "b", "c", "d"}
```

切片可以用内置的make函数创建，它的签名是：
```go
func make([]T, len, cap) []T
```

T 代表要创建切片的元素类型。make函数接收元素类型，长度和可选容量。调用时，make 将分配一个数组并返回指向该数组的切片。
```go
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}
```

当省略容量参数时，默认为指定的长度。这是相同代码的更简洁版本：
```go
s := make([]byte, 5)
```

可以使用内置的len和cap函数检查切片的长度和容量
```go
len(s) == 5
cap(s) == 5
```

接下来的两个部分讨论长度和容量之间的关系。

切片的零值为nil。len和cap函数对于nil切片都将返回0。

切片也可以通过“切片”现有的切片或数组来形成。切片是通过指定一个半开的区间和两个用冒号分隔的索引来完成的。例如，表达式b[1:4]创建了一个包含b的元素1到3的切片(结果切片的索引将是0到2)。
```go
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b
```

切片表达式的开始和结束索引是可选的;它们的默认值分别为0和slice的长度:
```go
// b[:2] == []byte{'g', 'o'}
// b[2:] == []byte{'l', 'a', 'n', 'g'}
// b[:] == b
```

这也是在给定数组的情况下创建切片的语法:
```go
x := [3]string{"Лайка", "Белка", "Стрелка"}
s := x[:] // a slice referencing the storage of x
```

## 切片内部(Slice internals)
一个切片是一个数组段的描述符。它由指向数组的指针、段的长度和它的容量(段的最大长度)组成。
![](https://go.dev/blog/slices-intro/slice-struct.png)

之前由 make([]byte, 5) 创建的变量s的结构是这样的:
![](https://go.dev/blog/slices-intro/slice-1.png)

长度是切片引用的元素的数量。容量是底层数组中的元素数量(从切片指针引用的元素开始)。在接下来的几个示例中，我们将清楚地说明长度和容量之间的区别。

当我们对s切片时，观察切片数据结构的变化及其与基础数组的关系：
```go
s = s[2:4]
```
![](https://go.dev/blog/slices-intro/slice-2.png)

切片操作不会复制原始切片的数据。它创建一个指向原始数组的新切片值。这使得切片操作与操作数组下标一样高效。因此，修改新切片的元素(而不是切片本身)会修改原始切片的元素:
```go
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:]
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}
```

之前我们将s切成比其容量更短的长度。我们可以通过再次切片使s增长到它的容量：
```go
s = s[:cap(s)]
```
![](https://go.dev/blog/slices-intro/slice-3.png)

一个切片不能超过它的容量。尝试这样做将导致运行时恐慌，就像在切片或数组边界外进行索引一样。同样，不能将切片重新切到0以下以访问数组中较早的元素。

## Growing slices (the copy and append functions)
要增长一个切片的容量，必须创建一个新的、更大的切片，并将原始切片的内容复制到其中。这种技术就是来自其他语言的动态数组实现在幕后工作的方式。下面的例子通过创建一个新的切片t，将s的内容复制到t中，然后将切片值t赋给s，从而使s的容量翻倍:
```go
t := make([]byte, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
for i := range s {
    t[i] = s[i]
}
s = t
```

内置的copy函数简化了这个常见操作的循环部分。顾名思义，copy将数据从源片复制到目标片。它返回复制的元素数量。
```go
func copy(dst, src []T) int
```

copy 函数支持在不同长度的切片之间进行复制(它只复制较少数量的元素)。此外，copy函数可以处理共享相同底层数组的源片和目标片，正确处理重叠的切片。

使用copy，我们可以简化上面的代码片段:
```go
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```

一种常见的操作是将数据追加到一个切片的末尾。这个函数将字节元素追加到字节的切片中，必要时增加切片，并返回更新后的切片值：
```go
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

可以这样使用 AppendByte：
```go
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
// p == []byte{2, 3, 5, 7, 11, 13}
```

像 AppendByte 这样的函数很有用，因为它们提供了对切片增长方式的完全控制。根据程序的特点，可能需要分配更小或更大的块，或对重新分配的大小设置一个上限。

但是大多数程序不需要完全控制，所以Go提供了一个内置的 append 函数，这对大多数目的都很好;它有签名:
```go
func append(s []T, x ...T) []T
```

append 函数将元素x追加到切片s的末尾，如果需要更大的容量，则会增长切片。
```go
a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}
```

要将一个切片附加到另一个切片，使用…将第二个参数拓展为一个参数列表。
```go
a := []string{"John", "Paul"}
b := []string{"George", "Ringo", "Pete"}
a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
// a == []string{"John", "Paul", "George", "Ringo", "Pete"}
```

因为切片的零值(nil)就像一个零长度的切片，你可以声明一个切片变量，然后在循环中添加它:
```go
// Filter returns a new slice holding only
// the elements of s that satisfy fn()
func Filter(s []int, fn func(int) bool) []int {
    var p []int // == nil
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

## 一个可能的“陷阱”
如前所述，重新切片并不会生成底层数组的副本。整个数组将保存在内存中，直到它不再被引用。有时，这可能导致程序将所有数据保存在内存中，而实际上只需要一小部分数据。

例如，FindDigits 函数将一个文件加载到内存中，并在其中搜索第一组连续的数字，将它们作为一个新片返回。
```go
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

此代码的行为与所宣传的一样，但返回的[]byte指向包含整个文件的数组。由于切片引用原始数组，只要切片被保留在垃圾收集器周围，就不能释放数组;文件的几个有用字节将整个内容保存在内存中。

为了解决这个问题，你可以在返回之前将感兴趣的数据复制到一个新的切片:
```go
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

这个函数的一个更简洁的版本可以通过使用 append 构造。