---
title: "Programming | Thread and Process"
date: 2022-04-18T13:16:00+08:00
draft: false
categories: ["Programming"]
tags: ["Thread", "Process"]
# weight: 1
cover:
    image: "https://i.ytimg.com/vi/Dhf-DYO1K78/maxresdefault.jpg"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    caption: "I do not own this picture, and it is used for non-commercial purposes only. If this violated your rights, please contact me and I will remove it."
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "I really hate the term Thread because it is very confusing."
author: "AidenCLX"
translationKey: "Threads and Processes"
---

The reason I write this article is very simple: I was asked a question about process and thread. I thought I had a fairly good understanding of thread and process, but it turned out that my understandings are not detailed enough. So I tried to look into more materials, and therefore I write this article to share the knowledge I acquired.

---

## Process

Why start with process? The reaons is trivial: it is simpler. Not technically simpler, but easier to understand. So what is a process after all? Essentially, a process is an abstraction. It is created to isolate the user created programs from the low-level hardwares. A process, in formal words, is the minimum unit of resource allocation. You CANNOT allocate a smaller chunk of resource for an independent program, say, a smaller unit of memory like part of a stack, to a program on the **system level**. I emphasize again: you CANNOT do so on the **system level**. The reason will be discussed in the thread section.

So what does a process have? An independent memory address space (This is what we call as the "different memory space" not shared between processes), depending on your system. If it is 32-bit, your program will have 32-bit virtual memory space each. If it is 64-bit, then your program has 64-bit virtual memory space each. And they do not share the virtual memory space. A process also has its own file descriptors, PID, dedicatedly allocated stack, heap, data section, text section, etc. Switching between processes has a lot of overhead because of the isolation nature of processes: you have to save the current state, switch to another virtual memory space, suffer from page-miss, etc.

But does it mean a process has no advantages? That is absolutely not true. Thanks to the isolation nature of processes, crashing of one process will not influence other processes; which means multi-processing could add availability. And that is also the reason why you see daemon processes in linux systems, but never daemon threads.

---

## Thread

People get easily confused by threads, so did I. The reason why threads are confusing is about the definition: what is a thread? Well, it really depends on the context, but people are usually referring to kernel threads, or in another word, system threads. You might be wondering, what? Are there any more types of threads? Well, my answer will be an absolutely yes because that's where the confusion comes.

A thread is something of a smaller scale than a process, that is something for sure. No one would doubt it. But to what extent? Well, that depends on the implementation. I repeat, that depends on the implementation.

When your professors or teachers refer to "a thread", they are definitely referring to a kernel thread, which is what we call a ``POSIX Thread``, or ``pthread`` for short if you ever use c++ to implement multi-threading. These kind of threads are created, maintained and terminated by the operation system - you don't have much control over them, all what you can do is to assign each thread with some tasks then leave them to the system. And these kind of threads requires resource allocation within the process they reside in: they will reserve their own stack, heap in the process. However, they share things like file descriptors and data or text section in the process. They also share the same virtual memory space.

To be more precise, a process has at least one thread within and nowadays, the cpu scheduler only schedule kernel threads, not processes (though in academy we can say the os schedules processes). But I have to mention that in linux system, kernel threads are actually light-weighted processes...

That is kernel thread. Now we should move on to the user thread.

What is a user thread? Well, it is nothing but another abstraction. It is used so that programmers have more control over the scheduler. By using user threads, we can also achieve cooperative scheduling instead of pre-emptive scheduling, which is not efficient enough speaking of blocking I/O.

To be precise, a user thread is a thread, but the scheduler is implemented depending on the programming language, not the os. At the same time, a user thread might or might not need to allocate dedicated resources like kernel threads do, depending on the implementation.

There are also terms like coroutines, goroutines, which are quite confusing. What are they? Well, that's a good question. I used to think that these things are something smaller than a thread, but to be frank, they are threads. To be precise, they are **user threads**. It's not like they reside in a thread, it is wrong. They ARE threads. They ARE user threads.

You must have heard the term user threads and kernel threads and their mappings from your professor if you are a computer science major student. Now you know exactly what your professors or teachers are referring to. Let's take go as an example. Go will initialize its kernel thread pool when you run the program according to your virtual CPU's number. Say you have 4 logical cores, then go will initialize 4 kernel threads in the pool for the program to use, but you can change the number of maximum allowed kernel threads in the pool by manually assigning. At runtime, each goroutine will be mapped to one of the threads to do computations, with go's dedicatedly designed scheduler. So in short, kernel threads and user threads are not mutually exclusive. Instead, they can work together. Goroutines are just like that. The os will allocate time for the kernel threads in go's thread pool, and go's scheduler will determine which goroutine, namely user thread, to run for which kernel thread.

---

## Conclusion

Many of the confusions come from the term thread. When talking about thread, we usually refer to kernel threads instead of user threads, and we usually refer coroutines to user threads. Hope this clarification can help you with your understanding.