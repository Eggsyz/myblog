---
title: "Goroutine用法及调度器原理"
date:  2020-03-21T20:22:03
categories: golang
tags:
- golang
- goroutine
---

本文主要介绍Golang实现高并发的机制——Goroutine（协程），首先介绍并发相关概念，接着深入学习其内部调度器的实现架构（G-P-M 模型）。

## 并发与并行

如下图所示，并发不等于并行，并发是在“某段时间”内可以同时运行多个任务，而并行是存在多个任务同时运行。在单核CPU上，并发的实现是通过时间片来进行多任务切换的，看起来像是同时运行多个任务，这就是**并发**。而在多核CPU上，可以让多个任务同时运行，这就是**并行**。

<img src="../../../../../../../../Desktop/00831rSTly1gd1uo6gw2wj30ru0hiaas.png" style="zoom:40%">

## 进程、线程、协程

+ 进程

  进程是系统进行资源分配的基本单位，有独立的内存空间。

+ 线程

  线程是 CPU 调度和分派的基本单位，线程依附于进程存在，每个线程会共享父进程的资源。

+ 协程

  **协程是一种用户态的轻量级线程，**协程的调度完全由用户控制，协程间切换只需要保存任务的上下文，没有内核的开销。

进程中可以包含多个线程，由于各个线程共享了同一片内存空间，线程之间的通信是通过共享内存来实现的，相比于重量级的进程，线程显得比较轻量，所以我们可以在一个进程中创建出多个线程。

虽然线程相对进程比较轻量，但是线程仍然会占用较多的资源并且调度时也会造成比较大的额外开销，OS线程（操作系统线程）一般都有固定的栈内存（通常为2MB），一个goroutine的栈在其生命周期开始时**只有很小的栈（典型情况下2KB）**，goroutine的栈不是固定的，可以按需增大和缩小，goroutine的栈大小限制可以达到1GB。所以在Golang中一次创建十万左右的goroutine也是可以的。

并且线程进行切换时不止会消耗较多的内存空间，对寄存器中的内容进行恢复还需要向操作系统申请或者销毁对应的资源，每一次线程上下文的切换都需要消耗 `~1ms` 左右的时间，但是 Go 调度器对 Goroutine 的上下文切换 `~0.2ms`，减少了 `80%` 的额外开销。除了**减少上下文切换**带来的开销，Golang 的调度器还能够更有效地利用 CPU 的缓存。

## Golang 协程的用法

Golang中，使用协程的方式非常的简单，只需要在对应函数前面加上go关键字即可，这就创建了一个goroutine。

```go l
func hello() {
    fmt.Println("Hello Goroutine!")
}
func main() {
    go hello() // 这里就创建了一个协程
    fmt.Println("main goroutine done!")
}
```

在Golang中使用 Goroutine 并行执行任务并将 **Channel** 作为 Goroutine 之间的通信方式，虽然使用互斥锁和共享内存在 Golang中也可以完成 Goroutine 间的通信，但是使用 Channel 才是更推荐的做法 — **不要通过共享内存的方式进行通信，而是应该通过通信的方式共享内存**。

<img src="../../../../../../../../Desktop/00831rSTly1gd1uspfrncj30by0bmt8u.png" style="zoom:50%">

## Goroutine调度机制

### GPM模型

**GPM**是Golang运行时（runtime）层面的实现，是Golang自己实现的一套调度系统。区别于操作系统调度OS线程。**G-P-M 模型**，包括 4 个重要结构，分别是**G、P、M、Sched：**

<img src="../../../../../../../../Desktop/00831rSTly1gd1vdg2k1fj30jm0drweu.png" style="zoom:50%">

其中：

+ **G**：表示 Goroutine，每一个 Goroutine 都包含堆栈、指令指针和其他用于调度的重要信息，每个 Goroutine 对应一个 G 结构体，G 存储 Goroutine 的运行堆栈、状态以及任务函数，可重用，G 并非执行体，每个 G 需要绑定到 P 才能被调度执行。
+ **P**：表示调度的上下文，它可以被看做一个运行于线程 M 上的本地调度器，**P 的数量决定了系统内最大可并行的 G 的数量**
+ **M**：表示操作系统的线程，它是被操作系统管理的线程，代表着真正执行计算的资源；
+ **Sched：Go 调度器，**它维护有存储 M 和 G 的队列以及调度器的一些状态信息等。

### 调度模型

 在Go中，线程是运行goroutine的实体，调度器的功能是把可运行的goroutine分配到工作线程上。Go 调度器中有两个不同的运行队列：**全局运行队列(global runqueue: GRQ)和本地运行队列(local runqueue: LRQ)。**

从图可以看出，每个M对应一个P，每个P也有一个正在运行的G，而其他的G在排队等待调度。其中，P 的数量由用户设置的 GoMAXPROCS 决定，但是不论 GoMAXPROCS 设置为多大，P 的数量最大为 256。**P的数量决定了最大并行G的数量**。图中绿色的G并没有运行，处于ready状态。每个 P 都有一个 **LRQ**，用于管理分配给在 P 的上下文中执行的 Goroutines，这些 Goroutine 轮流被和 P 绑定的 M 进行上下文切换。**GRQ** 适用于尚未分配给 P 的 Goroutines。

其中，M运行任务就得获取P，从P的本地队列获取G，当P队列为空时，M也会尝试从其他P的本地队列偷一半放到自己P的本地队列，如果无法获取，则从全局运行队列拿一批G放到P的本地队列。M运行G，G执行之后，M会从P获取下一个G，不断重复下去。当新建G时，G优先加入到P的本地运行队列，如果本地运行队列满了，则会把本地队列中一半的G移动到全局队列。

<img src="https://tva1.sinaimg.cn/large/00831rSTly1gd1vuerkxsj30jm0ii3z4.jpg" style="zoom:50%">

Golang中，**为了更加充分利用线程的计算资源，Go 调度器采取了以下几种调度策略：**

+ **任务窃取（work-stealing）**

  当本线程无可运行的G时，尝试从其他线程绑定的P偷取G，而不是销毁线程。

+ **减少阻塞（hand off）**

  当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执行。

接下来将分别介绍对应的场景。

#### 任务窃取

在实际的运行场景中，有的 Goroutine 运行的快，有的慢，那么势必肯定会带来的问题就是，这就导致了这个处理器P很忙，但是其他的P还有任务，此时如果global runqueue没有任务G了。为了提高 Go 并行处理能力，调高整体处理效率，当每个 P 之间的 G 任务不均衡时，调度器允许从 GRQ，或者其他 P 的 LRQ 中获取 G 执行。

<img src="https://tva1.sinaimg.cn/large/00831rSTly1gd1w7yohonj312o0l175j.jpg" style="zoom:40%">

如图所示，当P没有任务G需要调度时，会从其他的P**窃取**G来进行调度。一般就拿run queue的一半，这就确保了每个OS线程都能充分的使用。

#### 减少阻塞

在实际的场景中，还存在正在执行的 Goroutine 阻塞了线程 M 。这种情况下，如果不做出改变，则对应P的LRQ队列里面的G都会得不到调度。为了应对这种情况，Go调度器设置了**减少阻塞**的策略。

<img src="https://tva1.sinaimg.cn/large/00831rSTly1gd1wkhuyuzj30y60jowfq.jpg" style="zoom:40%">

当一个OS线程M0陷入阻塞时，P转而在运行在M1，M1可能是正被创建，或者从线程缓存中取出。当MO返回时，它必须尝试取得一个P来运行goroutine，一般情况下，它会从其他的OS线程那里拿一个P过来，
如果没有拿到的话，它就把goroutine放在一个global runqueue里，然后自己睡眠（放入线程缓存里）。所有的P也会周期性的检查global runqueue并运行其中的goroutine，否则global runqueue上的goroutine永远无法执行。

## Reference

+ [Go 为什么这么“快”](https://mp.weixin.qq.com/s/enjlUh9ldfpLUdU1VQFkRA)
+ [Go 语言调度器与 Goroutine](https://mp.weixin.qq.com/s/CkP7teMBd3TAIgLkWa_yvw)
+  [go语言之行--golang核武器goroutine调度原理、channel详解](https://www.cnblogs.com/wdliu/p/9272220.html)

