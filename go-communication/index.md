# Share Memory By Communicating


英文原文: [Share Memory By Communicating](https://go.dev/blog/codelab-share)

传统的线程模型（例如，在编写Java、C++和Python程序时通常使用）要求程序员使用共享内存在线程之间进行通信。
通常情况下，共享数据结构受到锁的保护，而线程将争夺这些锁来访问数据。在某些情况下，使用线程安全的数据结构(如Python的Queue)会使这更容易。

Go的并发原语 - goroutines和channels - 为构造并发软件提供了一种优雅而独特的方法。(这些概念有一个[有趣的历史](https://swtch.com/~rsc/thread/)，
始于C.A.R.Hoare的 [Communicating Sequential Processes](http://www.usingcsp.com/)）。
Go鼓励使用通道在goroutine之间传递对数据的引用，而不是明确地使用锁来调解对共享数据的访问。这种方法可以确保在给定时间内只有一个goroutine可以访问数据。
这个概念在[Effective Go](https://go.dev/doc/effective_go.html)文件中得到了总结（任何Go程序员都必须阅读）:

*不要通过共享内存来进行通信，相反，通过通信来共享内存*

考虑一个程序，它轮询一个url列表。在传统的线程环境中，数据的结构可能是这样的:
```go
type Resource struct {
    url        string
    polling    bool
    lastPolled int64
}

type Resources struct {
    data []*Resource
    lock *sync.Mutex
}
```

然后一个轮询器函数(其中许多会在单独的线程中运行)可能看起来像这样:
```go
func Poller(res *Resources) {
    for {
        // get the least recently-polled Resource
        // and mark it as being polled
        res.lock.Lock()
        var r *Resource
        for _, v := range res.data {
            if v.polling {
                continue
            }
            if r == nil || v.lastPolled < r.lastPolled {
                r = v
            }
        }
        if r != nil {
            r.polling = true
        }
        res.lock.Unlock()
        if r == nil {
            continue
        }

        // poll the URL

        // update the Resource's polling and lastPolled
        res.lock.Lock()
        r.polling = false
        r.lastPolled = time.Nanoseconds()
        res.lock.Unlock()
    }
}

```

这个函数大约有一页那么长，需要更多的细节来完成它。它甚至不包括URL轮询逻辑(它本身只有几行)，也不会优雅地处理耗尽资源池的问题。

让我们来看看使用Go习语实现的相同功能。在本例中，Poller是一个函数，它从输入通道接收要轮询的资源，并在完成时将它们发送到输出通道。
```go
type Resource string

func Poller(in, out chan *Resource) {
    for r := range in {
        // poll the URL

        // send the processed Resource to out
        out <- r
    }
}
```

前面示例中微妙的逻辑明显不存在了，而且我们的Resource数据结构不再包含记账数据。事实上，剩下的都是重要的部分。这应该会让您对这些简单的语言特性的功能有一个初步的了解。

上面的代码片段有许多遗漏。有关使用这些思想的完整的、惯用的Go程序的演练，请参见Codewalk [Share Memory By Communicating](https://go.dev/doc/codewalk/sharemem/)。




