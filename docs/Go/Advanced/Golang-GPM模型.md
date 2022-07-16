调度模型

## 前置基础知识

### 协程的来源

线程分为 `用户态线程` 和 `内核态线程`。

CPU 能感知的只有 内核态线程，它负责内核态线程的创建、初始化、切换、销毁等任务；
而我们的任务是放在用户态线程去执行的，我们负责用户态线程的创建、初始化、切换、销毁等；
所以用户态线程必须和内核态线程必须绑定起来，使得CPU可以操作用户态线程。

>   另外，用户态线程的创建、初始化、切换、销毁等工作由`调度器`去做，用户（程序员）并不需要亲自动手。

既然用户态线程和内核态线程需要绑定起来，那也就有不同的绑定方式。一共有三种：

-   1 : 1（一对一）其实跟多线程多进程无异，切换的成本昂贵。
-   1 : N（一对多）无法利用多个 CPU，且如果一个用户态线程阻塞了，后面的用户态线程也执行不了。
-   M : N（多对多）能够利用多核，但是过于依赖调度器的优化和算法。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210805145156.png)

Golang 中的并发方案，采用的是多对多。早期 Golang 的调度器其实性能很差，直到后来经过优化以后有了质的飞跃，也成了 Golang 的一张名片。

经过优化以后，Golang 的调度器对 Goroutine 进行优化，一个 Goroutine 的占用仅几 KB，可以随意大量开辟。而现在采用的 GPM 模型，调度灵活，切换成本低。 

## GPM 线程调度模型

### GPM 模型

>   **G** 全称 Goroutine，**P** 全称 Processor，**M** 全称 Machine。

- **G**：goroutine，即需要分担出去的任务；

- **P**：一个装满 G 的队列，用于维护一些任务；

- **M**： 一个操作器，用于将一个 G 搬到线程上执行；

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210805192928.png)

-   **G** 是 Goroutine，装载着要执行的任务

-   **P** 是处理器，它的个数由 `GOMAXPROCS` 决定，这个个数是真正能够并行的数量。

-   **M** 是内核态线程，想要运行任务就得获取 P；
    -   获取途径：从 P-LRQ 中获取，若 P-LRQ 为空：
        -   则从全局队列**拿**一批 G 放到自己的 P-LRQ
        -   或从别的 P-LRQ 中**偷**一半到自己的 P-LRQ
    -   M 运行 G，G 执行完后，M 会从 P-LRQ 列获取下一个 G，一直重复；
    -   当 M 阻塞时，Golang 会自动创建一个新的 M 并接手阻塞 M 的 P-LRQ
    -   当 M 空闲时，有可能会回收或睡眠

-   **全局队列（Global Queue，GQ）**：存放等待运行的 G

-   **P 的本地运行队列（P's Local Running Queue，LRQ）**：
    -   存放 P 即将要运行的 G
    -   G 的数量 $\le$​​ 256 个
    -   队列中的 G 新创建的子 Goroutine —— G' 优先放在 LRQ（本地队列），如果 LRQ 满了，会把本地队列中一半的 G 放到全局队列。
-   **P 列表**：
    -   **P 的创建时机**：程序启动时创建，并保存在数组中；在确定了 P 的最大数量 n 后，运行时系统会根据这个数量创建 n 个 P。
    -   **P 的数量**：最多有环境变量 `$GOMAXPROCS` 个，可通过 `runtime.GOMAXPROCS(num)` 设置，但实际运行中受限于运行机器上的 CPU 有几个核心。
-   **M 列表**：
    -   **M 的创建时机**：
        -   原本的 M 阻塞了，就创建新的 M；
        -   P 中还有很多就绪任务，而没有空闲的 M 来执行，就创建新的 M。
    -   **M 的数量**：Golang 本身限定 M 的最大量是 10000，但一般用不到那么多，可通过 `runtime/debug.SetMaxThreads()` 设置 

### 调度策略

Golang 的调度器的设计策略大概有 4 点：`复用线程`、`利用并行`、`抢占`、`全局 G 队列`。

#### 复用线程

关于复用线程，有两种机制：`Work Stealing (偷取)机制` 和 `Hand Off (接手)机制`。

-   **Work Stealing 机制**

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210805193307.png)

    如上图，M1 的 P-LRQ 中有好几个 G 准备运行，而 M1 正在运行着 G1；另一边 M2 却没事干，想运行但没有 G，于是去 M1 的 P-LRQ 中从队尾偷走 G4 过来给自己执行。这就是偷取机制。

-   **Hand Off 机制**

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210805194044.png)

    如上图左边，G1 执行了一个阻塞操作时，会导致 M1 进入阻塞态，后面的 G2、G3 都无法运行。此时 Golang 会向操作系统申请创建一个新的内核态进程 M3，然后把 M1 的 P-LRQ 接过来给 M3 去执行。这就是接手机制。

#### 利用并行

`GOMAXPROCS` 设置 P 的数量，最有多 `GOMAXPROCS` 个线程分布在多个 CPU 上同时运行。默认的 `GOMAXPROCS` 等于运行机器上的核心数。

#### 抢占

在 coroutine 中要等待一个协程主动让出 CPU 才执行下一个协程，属于协作式调度，如果占用着 CPU 的 coroutine 不让出，别的 coroutine 就不可能。

在早期的 Golang 中也是如此，但是在 go1.14 起引入了异步抢占机制，当一个 Goroutine 运行超过 10 ms，Go 会尝试抢占它的运行资源分配给别的 Goroutine。

#### 全局 G 队列

当 M 从其他 P 中偷不到 G 时，可以从全局 G 队列中获取。

### go func() 的执行过程

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210805224713.png)

1.   通过 `go func()` 来创建一个 Goroutine；
2.   加入队列：
     -   如果本地队列 P-LRQ 未满，则先加入 P-LRQ；
     -   如果本地队列 P-LRQ 已满，则加入全局队列 GQ；
3.   M 处于**初始态**，获取 G：
     -   如果 M 对应的 P-LRQ 中有 G，则直接获取 G；
     -   如果 M 对应的 P-LRQ 中没 G，则去偷别的 M 的 P-LRQ 中的 G（从队尾偷一半）；
     -   如果 M 对应的 P-LRQ 中没 G，别的 P-LRQ 也没 G，则从全局队列 GQ 中获取；
4.   M 转为**就绪态**：获取到 G 之后绑定、初始化、各种资源就绪，准备执行；
5.   M 转为**执行态**：获取到 CPU 调度，开始执行 G 中的 `func()函数`
     -   M 在执行当前 G 时发生阻塞，该 M 转为**阻塞态**，Go 会找新的 M 来接管原本的 P-LRQ，重复步骤 4；
     -   新的 M 来源于：
         -   从 M 池中寻找正在休眠的可用的 M；
         -   M 池中无可用 M 时创建新的 M。
     -   阻塞的 M 获得条件后接触阻塞，G 返回原本的 P-LRQ 队尾，M 进入 M 池或销毁
6.   M 转为**初始态**：
     -   当 M 时间片结束时，会转为初始态，G 重入 LRQ 队尾，M 获取对头的 G，即步骤 3；
     -   当 G 的任务结束，M 转为 初始态，即步骤 3；

### 「M0」和「G0」

-   M0：启动程序后编号为 0 的主线程，是进程中唯一的
    -   在全局变量 `runtime.m0` 中，不需要在 heap 上分配；
    -   负责执行初始化操作和启动第一个 G；
    -   启动第一个 G 之后，M0 就和其他的 M 一样了。

-   G0：每次启动一个 M，都会第一个创建的 goroutine，就是 G0，是线程中唯一的
    -   每个 M 都有自己的 G0；
    -   G0 仅用于负责调度其他的 G；
    -   G0 **不**指向任何可**执行**的函数；
    -   在调度或系统调用时，M 会切换到 G0 来调度，使用的是 G0 的栈空间 ；
    -   M0 的 G0 会放在全局空间；



### Go 调度器执行过程

1. Go 先根据 `$GOMAXPROCS（默认为本机的逻辑CPU数）` 创建不同的P，并将它们存储在空闲P列表中。
2. `新的 goroutine` 或 `goroutine 进入就绪态后`，会唤醒一个 P 去分配资源，这个 P 会创建一个带有关联 OS 线程的 M。
3. 与 P 一样，M 空闲时（即 没有 goroutine 等待运行），会从系统调用返回，甚至被 GC 强制停止，转到空闲M列表中。

在示例中，打印 hello 的goroutine会使用主 goroutine，打印 world 的goroutine 会从 空闲列表中 获得 M 和 P。

runtime 创建 P -> P 创建 m0、g0 -> 关联 m0和 g0


## CSP 模型

Communicating Sequential Process 通信顺序进程（简称CSP），是一种并发编程模型，是一个很强大的并发数据模型。于上个世纪七十年代提出，用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型。

相对于Actor模型，CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

严格来说，CSP 是一门形式语言（类似于 ℷ calculus），用于描述并发系统中的互动模式，也因此成为一众面向并发的编程语言的理论源头，并衍生出了 Occam/Limbo/Golang…

而具体到编程语言，如 Golang，其实只用到了 CSP 的很小一部分，即理论中的 Process/Channel（对应到语言中的 goroutine/channel）：这两个并发原语之间没有从属关系， Process 可以订阅任意个 Channel，Channel 也并不关心是哪个 Process 在利用它进行通信；Process 围绕 Channel 进行读写，形成一套有序阻塞和可预测的并发模型。

### Golang CSP


与主流语言通过共享内存来进行并发控制方式不同，Go 语言采用了 CSP 模式。这是一种用于描述两个独立的并发实体通过共享的通讯 Channel（管道）进行通信的并发模型。

Golang 就是借用CSP模型的一些概念为之实现并发进行理论支持，其实从实际上出发，go语言并没有，完全实现了CSP模型的所有理论，仅仅是借用了 process和channel这两个概念。

process是在go语言上的表现就是 goroutine 是实际并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。

Go语言的CSP模型是由协程Goroutine与通道Channel实现：

- Go协程 goroutine: 是一种轻量线程，它不是操作系统的线程，而是将一个操作系统线程分段使用，通过调度器实现协作式调度。是一种绿色线程，微线程，它与Coroutine协程也有区别，能够在发现堵塞后启动新的微线程。

- 通道channel: 类似Unix的Pipe，用于协程之间通讯和同步。协程之间虽然解耦，但是它们和Channel有着耦合。



