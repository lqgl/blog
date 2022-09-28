# Go Maps


英文原文:[Go maps in action](https://go.dev/blog/maps)

## 介绍

哈希表是计算机科学中最有用的数据结构之一。许多哈希表实现具有不同的属性，但通常它们提供快速查找、添加和删除。Go提供了实现哈希表的内置映射(map)类型。

## 声明和初始化

Go 映射(map)类型看起来像这样:
```go
map[KeyType]ValueType
```

其中 KeyType 可以是任何[comparable](https://go.dev/ref/spec#Comparison_operators)的类型(稍后详细介绍)，
ValueType 可以是任何类型，包括另一个映射(map)!

变量 m 是字符串键到 int 值的映射:
```go
var m map[string]int
```

map 类型是引用类型，比如指针或切片，所以上面 m 的值为nil;它不指向一个初始化的映射。
空映射(nil map)在读取时表现得像空映射(empty map)，但试图写入空映射(nil map)将导致运行时panic; 不要这样做。要初始化一个map ，使用内置的make函数:
```go
m = make(map[string]int)
```

make 函数分配并初始化一个散列映射(hash map)数据结构，并返回指向它的映射(map)值。该数据结构的细节是运行时的实现细节，并不是由语言本身指定的。
在本文中，我们将重点讨论映射(maps)的使用，而不是它们的实现。

## 使用映射(maps)

Go 为处理映射(maps)提供了一种熟悉的语法。该语句将键“route”的值设置为66:
```go
m["route"] = 66
```

该语句检索存储在键“route”下的值，并将其赋值给一个新变量i:
```go
i := m["route"]
```

如果请求的键不存在，则得到值类型的零值。在本例中，值类型是int，所以0值是0:
```go
j := m["root"]
// j == 0
```

内置的 len 函数根据映射中的项数返回:
```go
n := len(m)
```

内置的 delete 函数从映射中删除一个条目:
```go
delete(m, "route")
```

delete 函数不返回任何内容，如果指定的键不存在，则不执行任何操作。

双值赋值测试键是否存在:
```go
i, ok := m["route"]
```

在这个语句中，第一个值(i)被分配到存储在键“route”下的值。如果该键不存在，i是值类型的零值(0)。
第二个值(ok)是一个bool值，如果该键在映射中存在则为true，如果不存在则为false。

要在不检索值的情况下测试键，在第一个值处使用下划线:
```go
_, ok := m["route"]
```

要遍历映射的内容，使用range关键字:
```go
for key, value := range m {
    fmt.Println("Key:", key, "Value:", value)
}
```

要用一些数据初始化一个映射，使用一个映射字面量:
```go
commits := map[string]int{
    "rsc": 3711,
    "r":   2138,
    "gri": 1908,
    "adg": 912,
}
```

可以使用相同的语法初始化一个空映射(empty map)，在功能上与使用 make 函数相同:
```go
m = map[string]int{}
```

## 利用零值

当键不存在时，映射检索产生零值是很方便的。

例如，布尔值的映射可以用作类似集合的数据结构(回想一下布尔类型的零值为false)。
这个示例遍历一个节点链表并打印它们的值。它使用Node指针的映射来检测列表中的循环。
```go
    type Node struct {
        Next  *Node
        Value interface{}
    }
    var first *Node

    visited := make(map[*Node]bool)     // key line
    for n := first; n != nil; n = n.Next {
        if visited[n] {                 // key line
            fmt.Println("cycle detected")
            break
        }
        visited[n] = true               // key line
        fmt.Println(n.Value)
    }
```

表达式 visited[n] 如果访问过n则为真，如果n不存在则为假。没有必要使用二值形式来测试映射中是否存在n;默认的0值为我们做了这些。

另一个有用的零值实例是切片的映射。向nil切片添加值只是分配一个新切片，所以向切片的映射添加值是一行程序;不需要检查键是否存在。
在下面的示例中，People切片使用Person值填充。每个人都有一个名字和一个点赞切片。该示例创建了一个映射，将每个点赞与喜欢它的人的一个切片联系起来。
```go
    type Person struct {
	    Name  string
        Likes []string
    }
    var people []*Person

    likes := make(map[string][]*Person)    // key line
    for _, p := range people {
        for _, l := range p.Likes {
            likes[l] = append(likes[l], p) // key line
        }
    }
```

打印一个喜欢奶酪的人的列表:
```go
    for _, p := range likes["cheese"] {
        fmt.Println(p.Name, "likes cheese.")
    }
```

打印喜欢培根的人的数量:
```go
    fmt.Println(len(likes["bacon"]), "people like bacon.")
```

请注意，由于range和len都将nil切片视为零长度的切片，所以即使没有人喜欢奶酪或培根，最后两个例子也可以工作(不管这种情况有多不可能)。

## 键类型

如前所述，映射键可以是任何可比较的类型。
[语言规范](https://go.dev/ref/spec#Comparison_operators)对此进行了精确的定义，但简而言之，可比较的类型是布尔型、数值型、字符串型、指针型、通道型和接口型，以及只包含这些类型的结构体或数组。
值得注意的是，列表中没有切片、映射和函数;这些类型不能使用==进行比较，也不能用作映射键。

显然，字符串、int和其他基本类型应该作为映射键可用，但结构体键(struct keys)可能出乎意料。Struct可用于多个维度的键数据。
例如，这个map of maps可以用来按国家统计网页点击量:
```go
hits := make(map[string]map[string]int)
```

这是string到(string到int)的映射。外部映射的每个键都是到具有自己内部映射的网页的路径。
每个内部映射键都是两个字母的国家代码。这个表达式获取一个澳大利亚人加载文档页面的次数:
```go
n := hits["/doc/"]["au"]
```

不幸的是，这种方法在添加数据时变得笨拙，因为对于任何给定的外部键，你必须检查内部映射是否存在，并在需要时创建它:
```go
func add(m map[string]map[string]int, path, country string) {
    mm, ok := m[path]
    if !ok {
        mm = make(map[string]int)
        m[path] = mm
    }
    mm[country]++
}
add(hits, "/doc/", "au")
```

另一方面，使用一个带有struct键的单一映射的设计消除了所有的复杂性:
```go
type Key struct {
    Path, Country string
}
hits := make(map[Key]int)
```

当一个越南人访问主页时，增加(可能创建)相应的计数器是一行代码:
```go
hits[Key{"/", "vn"}]++
```

同样，也很容易看出有多少瑞士人读过spec:
```go
n := hits[Key{"/ref/spec", "ch"}]
```

## 并发

[映射对于并发使用是不安全的](https://go.dev/doc/faq#atomic_maps):它没有定义当您同时读写它们时会发生什么。
如果需要从并发执行的goroutine中对映射进行读写，则访问必须通过某种同步机制进行调节。
保护映射的一种常用方法是使用[sync.RWMutex](https://go.dev/pkg/sync/#RWMutex)。

这条语句声明了一个计数器变量，它是一个匿名结构体，包含一个映射和一个嵌入的sync.RWMutex。
```go
var counter = struct{
    sync.RWMutex
    m map[string]int
}{m: make(map[string]int)}
```

要从计数器读取，取读锁:
```go
counter.RLock()
n := counter.m["some_key"]
counter.RUnlock()
fmt.Println("some_key:", n)
```

要写入计数器，取写锁:
```go
counter.Lock()
counter.m["some_key"]++
counter.Unlock()
```

## 迭代顺序

当使用range循环在映射上迭代时，不会指定迭代顺序，也不能保证从一个迭代到下一个迭代是相同的。
如果需要稳定的迭代顺序，则必须维护指定该顺序的独立数据结构。
下面的例子使用一个单独的排序的键切片按键的顺序打印一个map[int]字符串:
```go
import "sort"

var m map[int]string
var keys []int
for k := range m {
    keys = append(keys, k)
}
sort.Ints(keys)
for _, k := range keys {
    fmt.Println("Key:", k, "Value:", m[k])
}
```

