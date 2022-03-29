---
title: "Golang | Gouroutines, Channels and Concurrency"
date: 2022-03-27T16:12:32+08:00
draft: false
categories: ["Programming"]
tags: ["Golang", "Concurrency"]
# weight: 1
cover:
    image: "https://user-images.githubusercontent.com/4528223/32051797-3a0de658-ba80-11e7-8f45-202c1b5c6066.jpg"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: "My notes on avoiding common problems in Goroutines and Channels"
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "My notes on avoiding common problems in Goroutines and Channels"
author: "AidenCLX"
translationKey: "GolangGoroutinesAndChannels"
---

## About Goroutines and Channels

Different from traditional multi-processing and multi-threading, golang provides a different method to achieve concurrency: Goroutines. It is light-weighted and fast, while providing pretty straight forward usage. But what is a goroutine after all? Since it is different from processes and threads, where does it exist?

---

### Goroutines

First of all we need to figure out what is a process and what is a thread. A process is defined as an instance of a computer program that is **being executed** by one or more threads. And what is a thread? It is like a "Virtual Process" which resides in an actual process, taking some RAM space acquired by the process it resides in. There can be more than one threads in a process, and they look like this:

![Process and Thread](/concurrency/process_thread.jpeg#center)

Then where do goroutines reside in? In short, goroutines are part of a thread:

![Goroutine](/concurrency/goroutine.jpeg#center)

And to create a goroutine is very simple, too. Just need to add the `go` keyword before a function. By convention, the function used to create a goroutine doesn't `return` anything. Of course you can make the function `return` something, but the main goroutine (where you create other goroutines) will never receive the return value after all. And that is the reason `channels` are used.

### Channels

What is a `channel`? Well, it is not complicated at all. A channel is like a `pipe` in linux, thought it is not exactly a `pipe`. It is used to transmit values between different goroutines. However, it is not a shared variable. You can use a channel to send and/or receive data from another goroutine, but by convention, it would be better for a goroutine to use a `channel` either for sending or for receiving to prevent deadlock situations. Let's take an example:

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
    // Might be both wanting to write in
    // Then block
}
```

So it would usually be better to use restrained channels like `<-chan` and `chan<-`.

---

## Typical Concurrency Problems

Creating a concurrent program is simple using go. However, it can also bring a lot of problems when not using goroutines and channels properly. I listed several problems I have encountered in the past here, as a reference in the future, just in case.

---

### Loop Variable Capture

This situation is not only common in normal programming exercises when using enclosed function literals, like the follows:

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

The output of the above code will be:

```text
5
5
5
5
5
Done
```

Why? Because when constructing an enclosed function literal, if it accessed some external variables, the variables would be accessed by reference. Think of a global variable like this:

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

How will `add()` access the num variable? It is trivial that add will access the `num` variable by reference. Similar for the `i` variable, it will not be accessed until the goroutine starts to run, which means the change of `i` variable will be reflected in the running goroutines, causing the problem. In this case, it is better to create an explicit copy of the variable and pass to the function to avoid such problems.

---

### Gouroutine Leak

Goroutine Leak is a typical problem while progamming with goroutines. It is a problem defined to be: some goroutines are blocked while none other goroutines can de-block these blocked goroutines. For example:

1. Lots of goroutines are created to write to an unbuffered channel while only received several times.
2. Lots of goroutines are trying to receive from an unbuffered channel while only several goroutines write to the channel.

This is a very typical situation, and a typical solution to this problem would be using a buffered channel.

---

### Paralleling too much

Yes, this could be a very practical problems encountered, and to be precise, this is not caused by goroutines but rather bottleneck in some other part of the system, e.g. limited I/O speed, limited bandwidth, etc. Let's take an example:

```go
// Suppose we are working under a directory
// with file named "1.txt", "2.txt", ....
// and suppose the number of files is large
// enough

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

The code above is draining the file descriptors so fast that system errors will happen after some time. So paralleling too much could cause problems, too. However, the solution to this problem is simple: limit the number of active goroutines at a moment. To be precise, we can use a buffered channel, and it is called a `counting semaphore`, as indicated in `The Go Programming Language`.

---

### Main Goroutine Exiting too Early

Unlike `fork()`, if the main goroutine exits, all other goroutines will be forced to exit, too. This causes problem when those sub goroutines are doing jobs that requires special clean-up.


```go
func main() {
    for i := 0; i < 10; i++ {
        go alloc()
    }
}

func alloc() {
    // Allocate memory which takes a long time
    defer free()
}
```

The above code shows it. The memory might be allocated but the sub goroutines never had time to free the memories. We can use `sync.WaitGroup` to solve this problem.