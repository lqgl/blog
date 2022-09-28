---
title: "When Generics"
date: 2022-09-19T09:19:20+08:00
lastmod: 2022-09-28T18:38:13+08:00
draft: false
author: "lqgl"
description: ""
resources:
- name: "featured-image"
  src: "images.jpeg"

tags: ["Go"]
categories: ["Go Blog"]

lightgallery: true

toc:
 auto: false
---

英文原文：[When To Use Generics](https://go.dev/blog/when-generics)

## 介绍

Go 1.18版本增加了一个主要的新语言特性:支持泛型编程。在本文中，我不打算描述泛型是什么，也不打算描述如何使用泛型。
本文讨论的是在Go代码中何时使用泛型，何时不使用泛型。

需要明确的是，我将提供一般的指导方针，而不是硬性规定。用你自己的判断。但如果您不确定，我建议使用这里讨论的指导方针。

## 编写代码

让我们从编写Go程序的一般指导原则开始:通过编写代码来编写Go程序，而不是通过定义类型。
谈到泛型，如果您通过定义类型参数约束开始编写程序，那么您可能走错了路。从编写函数开始。
很容易在以后添加类型参数，因为它们显然是有用的。

## 什么时候类型参数有用?

也就是说，让我们看看类型参数在哪些情况下是有用的。

### 当使用语言定义的容器类型时

一种情况是编写操作语言定义的特殊容器类型(切片、映射和通道)的函数。
如果一个函数具有这些类型的形参，并且函数代码没有对元素类型做任何特定的假设，那么使用类型形参可能会很有用。

例如，下面是一个函数，它返回任意类型映射中所有键的切片:
```go
// MapKeys returns a slice of all the keys in m.
// The keys are not returned in any particular order.
func MapKeys[Key comparable, Val any](m map[Key]Val) []Key {
    s := make([]Key, 0, len(m))
    for k := range m {
        s = append(s, k)
    }
    return s
}
```

这段代码没有对map键类型做任何假设，也根本不使用map值类型。它适用于任何map类型。这使得它成为使用类型参数的一个很好的候选者。

这类函数的类型参数的替代方案通常是使用反射，但这是一种更笨拙的编程模型，在构建时不进行静态类型检查，并且在运行时通常更慢。

### 通用数据结构

类型参数可能有用的另一种情况是用于通用数据结构。通用数据结构类似于切片或映射，但不是内置在语言中，如链表或二叉树。

今天，需要这种数据结构的程序通常会做两件事中的一件:用特定的元素类型编写它们，或者使用接口类型。
用类型参数替换特定的元素类型可以生成更通用的数据结构，可以在程序的其他部分或由其他程序使用。
用类型参数替换接口类型可以更有效地存储数据，节省内存资源;它还允许代码避免类型断言，并在构建时进行完全的类型检查。

例如，下面是使用类型参数的二叉树数据结构的一部分:
```go
// Tree is a binary tree.
type Tree[T any] struct {
    cmp  func(T, T) int
    root *node[T]
}

// A node in a Tree.
type node[T any] struct {
    left, right  *node[T]
    val          T
}

// find returns a pointer to the node containing val,
// or, if val is not present, a pointer to where it
// would be placed if added.
func (bt *Tree[T]) find(val T) **node[T] {
    pl := &bt.root
    for *pl != nil {
        switch cmp := bt.cmp(val, (*pl).val); {
        case cmp < 0:
            pl = &(*pl).left
        case cmp > 0:
            pl = &(*pl).right
        default:
            return pl
        }
    }
    return pl
}

// Insert inserts val into bt if not already there,
// and reports whether it was inserted.
func (bt *Tree[T]) Insert(val T) bool {
    pl := bt.find(val)
    if *pl != nil {
        return false
    }
    *pl = &node[T]{val: val}
    return true
}
```

树中的每个节点都包含类型参数T的值。当使用特定的类型参数实例化树时，该类型的值将直接存储在节点中。它们不会被存储为接口类型。

这是对类型参数的合理使用，因为Tree数据结构(包括方法中的代码)在很大程度上独立于元素类型T。

Tree数据结构确实需要知道如何比较元素类型T的值;它使用了一个传入的比较函数。
您可以在find方法的第四行看到这一点，在调用bt.cmp中。除此之外，类型参数根本不重要。

### 对于类型参数来说，宁可选择函数，也不选择方法，这一点都不重要。

Tree示例说明了另一个通用准则:当您需要比较函数之类的东西时，首选函数而不是方法。

我们可以定义Tree类型，这样元素类型就需要有一个 Compare 或 Less 方法。
这可以通过编写一个需要该方法的约束来实现，这意味着任何用于实例化Tree类型的类型参数都需要有该方法。

结果是，任何想要将Tree与简单数据类型(如int)一起使用的人都必须定义自己的整数类型并编写自己的比较方法。
如果我们定义Tree来接受一个比较函数，如上面所示的代码，那么很容易传入所需的函数。编写比较函数和编写方法一样容易。

如果Tree元素类型碰巧已经有一个Compare方法，那么我们可以简单地使用ElementType这样的方法表达式。Compare作为比较函数。

换句话说，将方法转换为函数要比向类型添加方法简单得多。因此，对于通用的数据类型，最好使用函数，而不是编写需要方法的约束。

### 实现一个通用方法

类型参数有用的另一种情况是，不同的类型需要实现一些公共方法，而不同类型的实现看起来都一样。

例如，考虑标准库的sort.Interface。它要求一个类型实现三个方法:Len、Swap和Less。

下面是实现排序的泛型类型SliceFn的一个示例。用于任何片类型的接口:
```go
// SliceFn implements sort.Interface for a slice of T.
type SliceFn[T any] struct {
    s    []T
    less func(T, T) bool
}

func (s SliceFn[T]) Len() int {
    return len(s.s)
}
func (s SliceFn[T]) Swap(i, j int) {
    s.s[i], s.s[j] = s.s[j], s.s[i]
}
func (s SliceFn[T]) Less(i, j int) bool {
    return s.less(s.s[i], s.s[j])
}
```

对于任何切片类型，Len和Swap方法完全相同。Less方法需要进行比较，这是名称SliceFn的Fn部分。与前面的树示例一样，我们将在创建SliceFn时传入一个函数。

下面是如何使用SliceFn使用比较函数对任何切片进行排序:
```go
// SortFn sorts s in place using a comparison function.
func SortFn[T any](s []T, less func(T, T) bool) {
    sort.Sort(SliceFn[T]{s, less})
}
```

这类似于标准库函数sort.Slice，但是比较函数是使用值而不是slice索引来编写的。

对这类代码使用类型参数是合适的，因为所有切片类型的方法看起来完全相同。

(我应该提到的是，Go 1.19而不是1.18很可能包含一个泛型函数，使用比较函数对切片进行排序，而该泛型函数很可能不会使用sort.interface。
见[建议#47619](https://go.dev/issue/47619)。
但是，即使这个特定的例子很可能没有用，但总的观点仍然是正确的:当您需要实现对所有相关类型看起来相同的方法时，使用类型参数是合理的。)

## 什么情况下类型参数没有用?

现在让我们讨论问题的另一面:什么时候不使用类型参数。

### 不要用类型参数替换接口类型

众所周知，Go有接口类型。接口类型允许一种泛型编程。

例如，广泛使用的io.Reader接口提供了一种通用机制，用于从包含信息(例如文件)或产生信息(例如随机数生成器)的任何值读取数据。
如果对某种类型的值所需要做的只是调用该值的方法，那么请使用接口类型，而不是类型参数。
io.Reader易于阅读，效率高，效果好。不需要使用类型参数通过调用read方法从值中读取数据。

例如，可能很容易将这里的第一个函数签名(只使用接口类型)更改为第二个版本(使用类型参数)。
```go
func ReadSome(r io.Reader) ([]byte, error)

func ReadSome[T io.Reader](r T) ([]byte, error)
```

不要做那种改变。省略type参数可以使函数更容易编写和读取，并且执行时间可能是相同的。

最后一点值得强调。虽然可以通过几种不同的方式实现泛型，而且实现会随着时间的推移而改变和改进，但在Go 1.18中使用的实现在很多情况下会将类型为类型参数的值与类型为接口类型的值处理得非常相似。
这意味着使用类型参数通常不会比使用接口类型快。因此，不要仅仅为了速度而从接口类型更改到类型参数，因为它可能不会运行得更快。

### 如果方法实现不同，不要使用类型参数

在决定是使用类型参数还是使用接口类型时，请考虑方法的实现。前面我们说过，如果一个方法的实现对于所有类型都是相同的，那么就使用类型参数。
相反，如果每个类型的实现都不同，那么就使用接口类型并编写不同的方法实现，而不要使用类型参数。

例如，从文件中读取的实现与从随机数生成器中读取的实现完全不同。这意味着我们应该编写两个不同的Read方法，并使用io.Reader这样的接口类型。

### 在适当的地方使用反射

Go有[run time reflection](https://pkg.go.dev/reflect)。反射允许一种泛型编程，因为它允许您编写适用于任何类型的代码。

如果某些操作必须支持甚至没有方法的类型(因此接口类型没有帮助)，并且如果每个类型的操作不同(因此类型参数不合适)，则使用反射。

一个例子是[encoding/json](https://pkg.go.dev/encoding/json)包。
我们不希望要求编码的每个类型都有MarshalJSON方法，因此不能使用接口类型。
但是对接口类型进行编码与对结构类型进行编码完全不同，因此不应该使用类型参数。
相反，这个包使用反射。代码并不简单，但它可以工作。详细信息请参见[源代码](https://go.dev/src/encoding/json/encode.go)。

## 一个简单的指南

最后，关于何时使用泛型的讨论可以简化为一个简单的指导原则。

如果您发现自己多次编写完全相同的代码，而副本之间的唯一区别是代码使用了不同的类型，那么请考虑是否可以使用类型参数。

另一种说法是，在注意到将要多次编写完全相同的代码之前，应该避免使用类型参数。