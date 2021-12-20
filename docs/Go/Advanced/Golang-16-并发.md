# 并发

![串行、并行、并发](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210210160730868_457.png)

## goroutine
Golang 并发通过 `goroutine` 实现，类似于线程，属于用户态的线程。
`goroutine` 开销非常小，所以一个程序开辟数千个都没问题。

Golang 在语言层面已经内置了调度和上下文切换的机制，会智能地将 `goroutine` 种的任务合理地分配给每个CPU。

当需要让某个任务并发的执行时，只要把这个任务包装成一个函数，开启一个 `goroutine` 去执行这个函数即可。

## 主 goroutine
`main()` 函数自己本身就处于 `主 goroutine`。
`主 goroutine` 结束的时候，其他执行完毕的、未执行完毕的 `子 goroutine` 也会一并被结束。

`主 goroutine` 的任务：

1. **设定每个 `goroutine` 所能申请的栈空间的最大尺寸**。
   在 32 位系统中最大尺寸为 250MB，64 位系统中最大尺寸为 1GB。如果某个 `goroutine` 的占空间尺寸大于这个限制，则运行时系统会引发一个栈溢出（Stack Overflow）的 `panic`，随后这个程序也会终止。
2. **执行初始化工作**：
    1. 创建一个特殊的 `defer` 语句，用于`主 goroutine` 退出时做必要的善后处理。因为 `主 goroutine` 也可能非正常结束。
    2. 启动专用于 `GC(Garbage Clean)` 的 `goroutine`，并设置 GC 可用的标识。
    3. 执行 `main` 包中的 `init()` 函数。
    4. 执行 `main()` 函数。
3. 在 `main()` 函数结束后，**检查 `主 goroutine` 是否引发了运行时 panic**，并进行必要的处理。
4. 最后，**结束自己及当前进程的运行**。

## 使用
一个 `goroutine` 必对应一个函数，可以创建多个 `goroutine` 去执行相同的函数。
只需要在函数调用处加上 `go` 关键字修饰，即可为一个函数创建一个 `goroutine`。

=== "单个goroutine"
    === "串行程序"
        ```go
        func hello() {
            fmt.Println("Hello World")
        }

        func main() {
            hello()
            fmt.Println("Main Function")
        }
        ```

        _输出_:

        ```shell
        Hello World
        Main Function
        ```

    === "goroutine 并发程序"
        ```go
        func hello() {
            fmt.Println("Hello World")
        }

        func main() {
            go hello()    // 为 hello() 开启一个 goroutine
            fmt.Println("Main Function")
            time.Sleep(time.Second)    // 让主程序休眠一下
        }
        ```

        _输出_:

        ```shell
        Hello World
        Main Function
        ```

=== "多个goroutine"
    在单个 `goroutine` 的例子中，为了防止 `主 goroutine` 快于 `子 goroutine` 完成，导致 `子 goroutine` 来不及完成就退出程序，在 `main()` 中添加了一条 `time.Sleep()`。虽然暂时可以解决这个问题，但是不优雅，也不科学。

    !!! info
        所以我们需要使用 `sync.WaitGroup` 来同步多个 `goroutine`。

    使用步骤：

    1. 声明一个 `sync.WaitGroup` 对象
    2. 添加一个计数器
    3. 调用 `Wait()` 等待
    4. 在 `goroutine` 中 `defer` 修饰 `sync.WaitGroup` 的 `Done()` 方法


    ```go
    package main

    import (
        "fmt"
        "sync"
    )

    var wg sync.WaitGroup    // 声明一个 WaitGroup 对象

    func main() {
        wg.Add(10)    // 添加一个计数器
        for i := 0; i < 10; i++ {
            go hello(i)
        }
        fmt.Println("Main Function")
        wg.Wait()    // 等待计数器归零再往下走
    }

    func hello(i int) {
        defer wg.Done()    // 计数器 -1
        fmt.Println("Hello ", i)
    }
    ```

    _输出_:

    ```shell
    Hello  1
    Hello  2
    Hello  3
    Hello  4
    Main Function
    Hello  9
    Hello  6
    Hello  7
    Hello  8
    Hello  0
    Hello  5
    ```

