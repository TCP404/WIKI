---
title: 'Golang [基础] 14-错误与异常'
seotitle: 'Golang [基础] 14-错误与异常'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: b2fbd54c
date: 2021-07-17 11:47:17
updated: 2021-07-17 11:47:17
---

错误与异常的处理

<!--more-->

# 错误与异常

> 意料之中的叫**错误**
> 意料之外的叫**异常**
>
> 错误：是指可能出现问题的地方出现了问题，比如打开一个文件时失败，这种情况是在意料之中的。
> 异常：是指不应该出现问题的地方出现了问题，比如引用了空指针，这种情况在人们的意料之外。
> 所以错误是业务过程的一部分，而异常不是。

## 错误
> Go 中错误是一种类型。
> 错误用内置的 `error`类型表示。和 `int、float64` 是等价的。
> 
> `error` 是一个接口，接口是一种类型，所以说 `error` 是一种类型。
> 
> ```go
> type error interface {
> 	Error() string
> }
> ```
> `error`类型可以存储错误值，从函数中返回等等。



```go
package main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Open("test.txt")
    if err != nil {
        fmt.Println(err)
        return
    }
    // 根据f进行文件读写
    fmt.Println(f.Name(), "opened successfully")
}
```

### 抛出错误

在编写桩模块供人调用时，我们不知道调用者会传进来什么参数，所以我们需要对传进来的参数做判断。
如果发生错误我们需要进行处理，或者**抛出错误**。

要抛出错误，那得先会创建错误。

创建错误有两种方式：

1. 通过 `errors` 包的 `New()` 函数创建：`func New(errMsg string) error`
2. 通过 `fmt` 包的 `Errorf()` 函数创建：`func Errorf(format string, a ...interface{}) error`

创建错误
```go
errors.New(string)
fmt.Errorf(string, ...interface{})
```

这两种方式区别在于：
`fmt.Errorf()` 可以使用格式化字符来返回错误信息，而 `errors.New()` 不能。

eg：

1. `errors.New()`：
    ```go
    func div(a, b float64) (float64, error) {
        if b == 0 {
            return 0, errors.New("除数小于0")
        }
        return a / b, nil
    }
    ```
    调用的效果：
    ```go
    fmt.Println(div(10, 0))    // 除数小于0
    ```

2. `fmt.Errorf()`：
    ```go
    func div(a, b float64) (float64, error) {
        if b == 0 {
            return 0, fmt.Errorf("错误码: [%d] \t 除数小于0", 100)
        }
        return a / b, nil
    }
    ```
    调用的效果：
    ```go
    fmt.Println(div(10, 0))    // 0 错误码: [100]          除数小于0
    ```

### 捕获错误
当我们写驱动模块的时候，在调用桩模块时有可能返回错误，我们可以通过判断 最后一个返回值 err （最后一个返回值返回错误类型，这是约定俗成的）来判断是否有返回错误。
```go
func main() {
    quotient, err := div(10, 0)
    if err != nil {    // 捕获错误
        fmt.Println(err)
        return
    }
    fmt.Println(quotient)
}
```

### 自定义错误

简简单单的 `Error.New()` 和 `fmt.Errorf()` 有时候并不能满足我们的需求，所以我们还得自定义错误。

自定义错误分为三步：

1. 创建一个结构体
2. 实现 `error` 接口
3. 定义其他方法

其中第一步和第二步是必选的，第三步是可选的。

eg：

自定义错误
```go
// 创建一个结构体
type dataError struct {
    Err    string
    height int
    weight int
    age    int
}
// 实现 error 接口，使 dataError 成为 error 类型
func (d *dataError) Error() string { return d.Err }

// 定义其他方法
func (d *dataError) heightNegativeError() bool { return d.height < 0 }
func (d *dataError) weightNegativeError() bool { return d.weight < 0 }
func (d *dataError) ageNegativeError() bool { return d.age <= 0 }
```
抛出错误
```go
type status = bool

const (
    SUCCESS = true  // SUCCESS 验证通过
    FAIL    = false	// FAIL    验证未通过
)

type Person struct {
    height int
    weight int
    age    int
}
// 验证不通过时，抛出错误
func variety(p Person) (status, error) {
    if p.height < 0 || p.weight < 0 || p.age < 0 {
        return FAIL, &dataError{"数据有误", p.height, p.weight, p.age}    // 抛出错误
    }
    return SUCCESS, nil
}
```
捕获错误
```go
func main() {
    p := Person{178, 120, -18}
    s, err := variety(p)

    if err != nil {    // 捕获错误
        fmt.Println("Error:", err)
        return
    }
    fmt.Println(s, p)
}
```
输出结果：
```shell
$ go run main.go
Error: 数据有误
```


如果捕获到了错误，而我们又想进一步判断是什么错误，需要用到 **类型断言**。
```go
func main() {
    p := Person{178, 120, -18}
    s, err := variety(p)

    if err != nil {    // 捕获错误
        if ins, ok := err.(*dataError); ok {    // 类型断言，判断是哪种错误
            switch {    // 捕获后的处理，可以打印错误信息，也可以取个绝对值等其他操作。
            case ins.heightNegativeError(): ins.Err = "身高为负"
            case ins.weightNegativeError(): ins.Err = "体重为负"
            case ins.ageNegativeError():    ins.Err = "年龄为负"
            }
            fmt.Println("Error:", ins)
        }
        return
    }
    fmt.Println(s, p)
}
```
输出结果：
```shell
$ go run main.go
Error: 年龄为负
```




## 异常

当一个程序遇到意料之外的错误时，应该抛出异常，停止程序的运行。
Go 中引入两个内置函数 `panic()` 和 `recover()` 来出发和终止异常处理流程，同时引入关键字 `defer` 来延迟执行延迟函数。

### 复习一下 defer
```go
func fn() {
    fmt.Println("start")
    defer fmt.Println(1)
    defer fmt.Println(2)
    defer fmt.Println(3)
    fmt.Println("end")
}
// ---------------------
// Output:
start
end
3
2
1
```
先 `defer` 的后执行，后 `defer` 的先执行


### panic

1. 内置函数
2. 发生 `panic` 的情况有两种：
    1. 手动抛出，即编写 `panic` 语句
    2. 程序发生错误由系统抛出。
3. 如果函数中出现了 `panic` 语句，则不会继续执行 `panic` 下面的代码，而是会逆序执行 `defer` 修饰的函数，然后返回调用处
4. 如果一路都没有遇到 `recover()`，则一路传递回所在协程的起点，然后该协程结束，终止其他所有协程（包括主协程）。
5. 最终整个 goroutine 退出，报告错误。

`panic` 的签名如下：
```go
func panic(v interface{})
```
`panic` 中接受一个任何类型的参数，可以传递一个错误消息的字符串，也可以传递一个错误码。

eg：
```go
func fn() {
    fmt.Println("start")
    defer fmt.Println(1)
    defer fmt.Println(2)

    panic(100)

    defer fmt.Println(3)
    fmt.Println("end")
}

func main() {
    defer fmt.Println("main defer...")
    fn()
    fmt.Println("After fn()...")
}

// ---------------------
// Output:
start
2
1
main defer...
panic: 100

goroutine 1 [running]:
main.fn()    e:/---CODE/GO/root/src/boii.xyz/study/Helloworld/main.go:19 +0x166
main.main()  e:/---CODE/GO/root/src/boii.xyz/study/Helloworld/main.go:26 +0x27
exit status 2
```
可以看出发生 `panic` 之后，在 `panic` 之前 `defer` 的代码依然会逆序执行，然后回到调用处。
上面的例子执行完 `fn()` 中 `defer` 函数后，便返回了 `main()` 函数，
接着执行 早在调用 `fn()` 之前就 `defer` 的 打印语句，然后就终止程序了。

{% note done, 结论：`defer函数`的执行 优先于 `panic` %}

### recover

1. 内置函数
2. 用来终止一个协程中的 panicking 行为，捕获 `panic`，从而影响应用的行为。
3. `recover` 需要用 `defer` 修饰。
4. 一般的调用建议：
    1. 在 defer 函数中，通过 `recover` 来终止一个 协程的 panicking 过程，从而恢复正常代码的执行
    2. 可以获取通过 `panic` 传递的 `error`


简而言之，go中可以抛出一个 `panic`的异常，然后在 `defer` 中通过 `recover` 捕获这个异常，然后正常处理。

{% noteblock warning %}
Q：为什么 `recover` 需要 `defer` 修饰？
A：因为发生 `panic` 后，只会执行 `defer` 修饰的函数，然后返回函数调用处。
      而 `recover` 是专门用来恢复 `panic` 的，所以必须用  `defer` 修饰 `recover`，
      且 `recover` 需放置在出现或可能出现 `panic` 的地方**之前**，否则一旦发生 `panic`，程序控制流就开始往回走，`panic` 下   方的任何代码包括 `defer` 修饰的地方也不会执行，这样就捕获不到 `panic` 了。
{% endnoteblock %}


eg：
发生数组下标越界，没有 `recover`时
```go
func traverse(a [5]int) {
    for i := 0; i <= 5; i++ {
        fmt.Println(a[i])
    }
    fmt.Println("under for")
}

func main() {
    defer fmt.Println("main defer...")

    a := [5]int{1, 2, 3, 4, 5}
    traverse(a)

    fmt.Println("After div()...")
}
// ---------------------
// Output:
1
2
3
4
5
main defer...
panic: runtime error: index out of range [5] with length 5

goroutine 1 [running]:
main.traverse(0x1, 0x2, 0x3, 0x4, 0x5)
        e:/---CODE/GO/root/src/boii.xyz/study/Helloworld/main.go:17 +0xbe
main.main()
        e:/---CODE/GO/root/src/boii.xyz/study/Helloworld/main.go:25 +0x105
exit status 2
```
最后的 `exit status 2`说明程序是非正常退出的。

下面是使用了 `recover` 的效果
```go
func traverse(a [5]int) {
    defer func() {
        if msg := recover(); msg != nil {
            fmt.Println(msg)
        }
    }()
    for i := 0; i <= 5; i++ {    // 这里会发生数组下标越界，引发panic
        fmt.Println(a[i])
    }
    fmt.Println("under for")
}

func main() {
    defer fmt.Println("main defer...")

    a := [5]int{1, 2, 3, 4, 5}
    traverse(a)

    fmt.Println("After div()...")
}
// ---------------------
// Output:
1
2
3
4
5
runtime error: index out of range [5] with length 5
After div()...
main defer...
```

可以看到，发生数组下标越界时，程序会引发 `panic`，然后开始执行 `defer` 函数；
在 `defer` 函数里遇到了 `recover`，使得程序恢复正常运行，打印了错误信息后，回到主函数继续执行
但是我们会发现虽然被 `recover` 恢复了，但是 for 循环下面的 语句依然不会执行。

{% noteblock done %}
结论：
`recover` 必须用 `defer` 修饰
`recover` 必须放在可能引发 `panic` 的地方之前
即使 `recover` 了，在函数里，引发 `panic` 的地方下面那些代码依然没有机会运行。
{% endnoteblock %}

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210207173528135_25602.png)

