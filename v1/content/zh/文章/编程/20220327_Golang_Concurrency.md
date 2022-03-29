---
title: "Golang | Gouroutines，Channels和并发"
date: 2022-03-27T16:12:32+08:00
draft: false
分类: ["编程"]
标签: ["Golang", "并发"]
# weight: 1
cover:
    image: "https://user-images.githubusercontent.com/4528223/32051797-3a0de658-ba80-11e7-8f45-202c1b5c6066.jpg"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: "My notes on avoiding common problems in Goroutines and Channels"
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "关于Goroutines和Channels易错点的笔记"
author: "AidenCLX"
translationKey: "GolangGoroutinesAndChannels"
---

## 关于Goroutines和Channels

与传统的多进程和多线程不同，golang提供了一种不同的并发方式，即通过使用goroutines。这是一种轻量化、快速的并发方式，并且使用方式非常直观。但说到底，goroutines是什么，与多进程和多线程又有什么不同的地方？既然他与进程和线程不同，那他处在应用程序的什么地方呢？

---

### Goroutines

首先先讲一下进程和线程相关。进程的定义是**正在被一个或多个线程所执行**的电脑程序的一个实例。那线程是什么呢？它就像一个在进程里的“虚拟进程”，并会要求一部分进程内的内存空间。一个进程内是可以有多个线程的，而它们之间的关系看起来像是这样：

![Process and Thread](/concurrency/process_thread.jpeg#center)

那Goroutines又在那里呢？长话短说，goroutines都是线程的一部分。

![Goroutine](/concurrency/goroutine.jpeg#center)

创建Goroutine也非常简单。我们只需要在函数前面加入`go`关键词就可以了。依照惯例，用于创建goroutine的函数不会返回任何东西。当然，事实上你可以返回一些东西，但主goroutine并不会接收到任何返回值...所以这么做只是在浪费时钟周期罢了。这也是为什么`Channels`被利用的原因。

### Channels

什么是`channel`？其实并不复杂。一个`channel`就像是linux系统当中的pipe一样，虽然他并不是一个真正的pipe。`channel`被用于不同的goroutines之间的数据交换与同步，但他并不是一个共享变量。你可以利用`channel`来发送或接受数据，但按照惯例，用于数据交换的`channel`应仅被用于接受或发送数据当中的一种，而不是同时使用，来避免死锁的情况。举个例子：

```go
func readAndWrite(ch chan int) {
    ch <- 1
    for x := <- ch {
        fmt.Println(x)
    }
}

func main() {
    ch := make(chan int)
    go readAndWrite(ch)
    ch <- 1
    // 主副goroutine可能会同时写
    // 导致阻塞
}
```

所以一般来说，更推荐使用单向的`<-chan`和`chan<-`。

---

## 典型的并发问题

创建一个并发程序在go中很简单。然而，这也带来了许多的问题，特别是当程序员无法正确使用goroutines和channels的时候。以下我罗列了一些我曾遇到的问题，仅作未来的参考和笔记。

---

### 循环变量捕获

这个情况不仅仅在一般编程中很常见，在使用嵌套函数的时候也非常的常见：

```go
func main() {
    ch := make(chan int)
    for i := 0; i < 5; i++ {
        go func(ch chan<- int) {
            ch <- i
        }(ch)
    }
    count := 0
    for count < 5 {
        fmt.Println(<-ch)
        count++
    }
    fmt.Println("Done")
}
```

以上程序的输出将会是：

```text
5
5
5
5
5
Done
```

为什么呢？因为当创建嵌套函数的时候，如果他访问了外部的变量，外部变量是以引用的形式被使用的。用全局变量来举个例子：

```go
var num int

func main() {
    add()
    fmt.Println(num)
}

func add() {
    num++
}

func init() {
    num = 1
}
```

`add()`函数会怎么访问num变量呢？很显然他会以引用的形式利用`num`变量，而访问`i`变量时的情况也是一致的。直到goroutine实际被执行之前，他都不会被访问到；这就代表着`i`变量的变化将会在goroutines实际运行时体现出来，造成实际问题。 这种情况下，最好显式地将变量传递给函数或创建一个复制，这样即可避免这个问题。

---

### Gouroutine泄露

Goroutine泄露是一个用goroutine编程时会遇到的典型问题。其定义为：部分goroutine被阻塞，而其他的goroutine无法解除阻塞。比如：

1. 许多goroutines都想向一个无缓存的channel去写，而几乎没有goroutine去读
2. 许多goroutines都想向一个无缓存的channel去读，而几乎没有goroutine去写

这是一个很典型的问题，但大部分时候利用缓存的channel就能解决这个问题。

---

### 并发数过多

是的！这也是一个非常典型的问题。说的准确些，其实这并不是由goroutine带来的问题，而更多的是外部的瓶颈条件，比如网速的限制，读写速度的限制，等等等等。举个例子：

```go
// 假设我们正在一个有着无数文件
// 的文件夹下面，而文件名都是：
// "1.txt", "2.txt", ...

var NUMOFFILES int

func main() {
    for i := 0; i < NUMOFFILES; i++ {
        go func(i int) {
            f, err := os.Open(fmt.Sprintf("%d.txt", i))
            defer f.close()
        }(i)
    }
}
```

上述的代码会无限制地把文件描述符用干净，然后很快系统错误就会出现。然而，解决这个问题也非常简单，我们依旧可以利用一个带缓存的channel来限制goroutines的数量，这个技巧在`The Go Programming Language`中被称为`counting semaphore`。

---

### 主goroutine过早退出

不像`fork()`如果主goroutine过早的退出了，其他的sub goroutine也会被强制退出。这将在sub goroutines正在做需要特殊垃圾收集处理的工作时可能会造成问题。


```go
func main() {
    for i := 0; i < 10; i++ {
        go alloc()
    }
}

func alloc() {
    // 分配内存。假设很耗时。
    defer free()
}
```

上述代码表现了这一点。内存可能被分配了却没能够被释放，因为主goroutine过早的退出了。我们可以使用`sync.WaitGroup`来解决这个问题。