# channel

!!! warning
    **通道 channel 是引用类型！！！**

**`channel` 的作用**：
单纯的并发执行函数的没有意义的。函数间需要交换数据才能体现并发执行的意义。
就像 OS 的并发性和共享性，没有并发谈不上共享，没有共享并发没有意义。
所以 `channel` 就是用在 `goroutine` 之间的通信上的。

!!! note ""
    Communicate through shared memory rather than through shared memory.  ----- Golang

    Golang 提倡 **通过通信共享内存而不是通过共享内存实现通信**。

`goroutine` 是程序并发的执行体，`channel` 就是它们之间的通信管道。

`channel` 有时候简写为：`chan`，遵循 **FIFO 先进先出** 的规则，保证了收发数据的顺序。

每一个 `chan` 都是一个具体类型的管道，即声明 `channel` 时需要为其指定元素类型。

## 创建 channel
```go
var idn chan T
idn = make(chan T[, size])

idn := make(chan T[, size])    // 短声明
```

1. `chan` 的创建需要 `make()` 分配内存才能使用。单纯的声明时，默认值为 `nil`。
2. 分配内存时可以指定通道大小，也就是这条管子的容量。

!!! example
    ```go
    func main() {
        var ch chan int          // 声明通道变量
        fmt.Println(ch)          // nil
        ch = make(chan int)      // 无缓冲区，分配内存
        fmt.Println(ch)          // 0xc0000d4000

        ch2 := make(chan int, 2) // 有缓冲区
        fmt.Println(ch2)         // 0xc0000d5000
    }
    ```

对于无缓冲的通道：

- 一次发送、一次接收，都是阻塞的

对于有缓冲的通道：

- **发送**->缓冲区满了，才会阻塞；
- **接收**->缓冲区空了，才会阻塞。

无缓冲通道就好像快递员送快递，只能当面送给你，你签收之前他就一直阻塞在那里。
有缓冲通道就好像有了快递柜，放到快递柜里等你自己来拿。快递柜满了快递员也只能等着，阻塞在那里。


## 操作 channel
通道有 **发送（通道读入数据）**、**接收（通道写出数据）**、**关闭** 三种操作

```go
ch := make(chan int)
```

!!! info 
    发送和接收有点容易混淆，可以这么记：**左发右收、左写右读**。
    
    通道在左边：向通道发送；

    通道在右边：从通道接收

### 发送
将一个值发送到通道中（通道写入一个值）。
```go
ch <- 10    // 把 10 发送到 ch 中
```

### 接收
从一个通道中接收值（通道读出一个值）。
```go
x := <-ch     // 从 ch 中接收值并赋值给变量 x
<- ch         // 从 ch 中接收值，忽略结果
```

### 关闭
调用内置函数 `close()` 来关闭通道。
```go
close(ch)
```

- 已关闭的通道仍然可以获取数据直到通道为空
- 已关闭的通道数据取完只会接收到零值
- 已关闭的通道再发送值会导致 `panic`
- 关闭已经关闭的通道会导致 `panic`，最好用 `new(sync.Once).Do(func() { close(ch) })` 关闭

关于关闭通道需要注意的事情是：

- 只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道。
- 通道是可以被垃圾回收机制回收的，它和关闭文件是不一样的，在结束操作之后关闭文件是必须要做的，但关闭通道不是必须的。

#### 判断通道是否被关闭
`x, ok := <-ch`
从通道中接收值时，会返回两个值：1️⃣ 一个是数据，2️⃣ 一个是通道开启状态。

通道被关闭后 `ok` 为 `false`。

```go
func main() {
    ch := make(chan int, 3)
    ch <- 10
    ch <- 20
    ch <- 30
    close(ch)

    for {
        x, ok := <-ch
        if !ok {
            break
        }
        fmt.Println(x)
    }
}

// Output:
10
20
30
```

### 单向通道
有时候我们会将通道作为参数在多个任务函数之间传递，通常我们在不同的任务函数中使用通道都会对其进行限制，如只能发送或接收。

上面介绍的都是 **双向通道**，接下来我们使用 **单向通道** 可以处理这种情况（**单向通道** 也常用于参数）。

- **`<-chan T`** 是一个 **只读** 单向通道，只能从通道中读取数据，通道可以执行 **接收** 操作但不能执行发送操作。
- **`chan<- T`** 是一个 **只写** 单项通道，只能向通道中写入数据，通道可以执行 **发送** 操作但不能执行读取操作。
- 在函数传参及任何赋值操作中可以将双向通道转换为单向通道，但反过来不可以。

```go
func producer(out chan<- int) {    // 生产者，只能向通道写入数据
    for i := 0; i < 100; i++ {
        out <- i
    }
    close(out)
}

func consumer(in <-chan int) {    // 消费者，只能从通道读取数据
    for i := range in {
        fmt.Println(i)
    }
}

func main() {
    ch := make(chan int, 100)
    go producer(ch)
    go consumer(ch)
}
```

## 通道总结
`channel` 常见异常总结：

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210211100625576_17942.png)

关闭已经关闭的`channel`也会引发`panic`。



## worker pool
`Worker Pool` 是一种模型，其中固定数量的 *m* 个 worker，通过 worker 队列中的 *n* 个任务工作。
worker 一直排在队列中，直到 worker 完成其当前任务并提出新任务为止。
在 Golang 中 worker 使用 `goroutine` 实现，任务使用通道实现。


```go
package main

import (
    "fmt"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Println("worker:", id, "start job:", j)
        time.Sleep(time.Second)
        fmt.Println("worker:", id, "end job:", j)
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    // 开启3个goroutine
    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }
    // 5个任务
    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)
    // close(results)

    for a := 1; a <= 5; a++ {
        fmt.Println(<-results)
    }
}
```

## select 多路复用
```go
select {
    case communication clause:
        statements
    case communication clause:
        statements
    ...
    default:    /* 可选 */
        statements
}
```

- 唯一一个可用的通道会被选择。
- 如果多个通道可用，会随机公平地选择一个，其他不会执行。
- 如果没有通道可用，default 情况将被执行。
    - 如果没有 default，select 将会阻塞，直到某个通道可以运行；Golang 不会重新对 channel 或 值进行求值。

!!! example
    ```go
    func main() {
        ch := make(chan int)

        go func() {
            ch <- 10
            fmt.Println("数据已写入")
        }()

        select {
        case x := <-ch:
            fmt.Println("数据已读出:", x)
        default:
            fmt.Println("default..")
        }
    }
    ```

    _输出_:

    ```shell
    default..
    ```

由于 `goroutine` 来不及启动完成，`select` 就执行了，此时 `case` 不满足，所以运行 `default`。
如果 `select` 之前睡眠一下，给 `goroutine` 点时间，就可以运行到 `case` 了。

!!! example
    ```go
    func main() {
        ch := make(chan int)

        go func() {
            ch <- 10
            fmt.Println("数据已写入")
        }()

        time.Sleep(500 * time.Millisecond)
        select {
        case x := <-ch:
            fmt.Println("数据已读出:", x)
        default:
            fmt.Println("default..")
        }
    }
    ```

    _输出_:

    ```shell
    数据已写入
    数据已读出: 10
    ```
