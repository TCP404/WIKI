# 8-函数
如果一个动作需要重复执行，那就应该把这个动作封装到函数中。

在 Go 中，函数是一等公民


!!! warning
    **函数是引用类型！！！**
## 定义

Go 语言中函数的基本形式：
```go
func 函数名(参数列表)(返回值){
    函数体
}
```

- **函数名**：由字母、数字、下划线组成，但不能数字开头；同一个包内，函数名不能重名
- **参数**：由参数变量和变量类型组成，多个参数使用`,` 分隔
- **返回值**：由返回变量和变量类型，也可只写变量类型，多个返回值必须用`()`包裹，并用`,`分隔
- **函数体**：实现指定功能的代码


!!! example
    求两数之和
    ```go
    func sum(a int, b int) int {
        return a + b
    }
    ```

    无参数无返回值的函数
    ```go
    func sayHello() {
        fmt.Println("Hello")
    }
    ```

## 调用
通过`函数名(参数)`的方式调用，如果定义的函数没有要求参数，则调用时不需要给参数

!!! example
    ```go
    func main() {
        res := sum(1, 3)
        fmt.Println(res)
    }
    ```
调用有返回值的函数时可以不接受其返回值。

## 参数
### 类型简写
如果相邻变量类型相同，可以只写一个

eg：
```go
func fn(x, y int, p, q string) {
    fmt.Println(x, y, p, q)
}
```
x 和 y 均为 int 型，x 后面就可省略；p 和 q 都是 string 型，p 后面就可以省略

### 可变参数
如果参数数量不固定，可以在类型前面加上`...`，但是只能接收这种类型的参数，且此时要注意类型简写的方法

!!! warning "注意"
    和固定参数搭配时，可变参数要放在固定参数后面

!!! example
    ```go
    func fn(x int, y ...int) {
        fmt.Println(x)            // 1
        fmt.Println(y)            // [2 3 5]
        fmt.Printf("%T \n", y)    // []int
    }

    fn(1, 2, 3, 5)
    // 错误，只能传递int型：fn(1, 2, "B", "C")
    // 错误，全都要int型：  fn(1, "A", "B", "C")
    ```
可变参数实际上是通过切片实现的。


- 关于类型简写：
    ```go
    // 错误
    func fn(x, y ...int) {}

    // 正确
    func fn(x ...int) {}
    func fn(x int, y ...int) {}
    func fn(x, z int, y ...int) {}
    ```

- 关于可变参数位置
    ```go
    // 错误
    func fn(y ...int, x int) {}
    func fn(x int, y ...int, s string) {}

    // 正确
    func fn(x int, y ...int) {}
    ```

## 返回值
### 多返回值
Go 中支持多返回值，如果有多个返回值时，必须用`()`包裹起来

!!! example
    ```go
    func calc(x, y int) (int, int) {
        return x + y, x - y
    }
    ```

调用之后可以接收返回值，也可以不接收，如果指向接收一个，另一个可以用匿名变量 `_` 去接

```go
// 两个都接收
sum, sub := calc(10, 5)

// 两个都不接收
calc(10, 5)

// 只接收一个
_, sub := calc(10, 5)
sum, _ := calc(10, 5)
```

### 返回值命名
函数定义时可以给返回值命名，并在函数体中直接使用这些变量，最后通过`return` 返回

```go
func calc(x, y int) (sum, sub int){
    sum = x + y
    sub = x - y
    return
}
```

如果给返回值命名了，但是又 return 了别的变量，以别的变量为准
```go
func calc(x, y int) (sum int, diff int) {
    sum = x + y
    diff = x - y
    mul := x * y
    div := x / y
    return mul, div
}

fmt.Println(calc(10, 5))    // 50 2
```
上面的 `#!go return mul, div` 可以看成 `#!go mul, div = sum, diff; return sum, diff`

## 函数进阶
### 变量作用域



### 函数类型
函数可以有类型，使用关键字`type`可以定义一个函数类型

eg：
```go
type calculation func(int, int) int
```
上面的例子定义了一个 `calculation` 类型，它是一种函数类型，一种接收两个int参数返回一个int值的函数类型。
方式满足上面特点的函数都是 calculation 类型。

!!! example
    ```go
    func add(x int, y int) int {
        return x + y
    }

    func sub(x, y int) int {
        return x - y
    }
    ```

    add() 和 sub() 都是 calculation 类型，都可以赋值给 calculation 类型变量

    ```go
    var c calculation
    c = add

    res := c(10, 5)
    fmt.Println(res)    // 15

    s := sub
    fmt.Println(s(20, 55))    // -35
    ```

### 函数变量
在 Go 中，函数被看作是第一类值，这意味着函数像变量一样，有类型、有值，普通变量能做的事函数也能做。

例如函数类型的例子，即使不定义函数类型，也可以直接将函数赋值给变量
```go
func add(x, y int) int {
    return x + y
}

func main() {
    a := add
    fmt.Println(a(10, 20))    // 30
}
```

- 调用 `nil` 的函数变量会导致 panic
- 函数变量的零值 `nil`，所以函数变量可以跟 `nil` 比较，但函数变量之间不能比较。


```go
package main

import "fmt"

type processFunc func(int) bool // 声明一个函数类型

func isOdd(i int) bool { // 定义这种类型的函数
    return i%2 != 0
}

func isEven(i int) bool { // 定义这种类型的函数
    return i%2 == 0
}

func filter(slice []int, f processFunc) []int { // 接收这种类型的函数
    var res []int
    for _, v := range slice {
        if f(v) {
            res = append(res, v)
        }
    }
    return res
}

func main() {
    s := []int{1, 2, 3, 4, 5, 7}
    fmt.Println("s = ", s) // s =  [1 2 3 4 5 7]

    odd := filter(s, isOdd)   // 调用这种类型的函数
    even := filter(s, isEven) // 调用这种类型的函数

    fmt.Println("Odd slice", odd)   // Odd slice [1 3 5 7]
    fmt.Println("Even slice", even) // Even slice [2 4]
}
```


### 匿名函数
没有名字的函数就叫匿名函数
基本语法：
```go
func (函数) (返回值) {
    函数体
}
```

匿名函数没有函数名，所以需要变量来保存，或者作为立即执行函数

```go
func main() {
    // 将匿名函数保存到变量
    add := func (x, y int) int {
        return x + y
    }
    add(10, 20)    // 通过变量调用匿名函数
}
```

```go
func main() {
    // 自执行函数：匿名函数定义完加()直接执行
    func (x, y int) {
        fmt.Println(x + y)
    }(10, 20)
}
```

### 高阶函数
高阶函数分为 `函数作参数` 和 `函数作返回值`

#### 函数作为参数
```go

// 定义一个函数，使用函数作为参数
// @param x       int                 操作数1
// @param y       int                 操作数2
// @param operate func(int, int) int  操作函数
// @return        int                 经过操作函数操作的结果
func calc (x int, y int, operate func(int, int) int) int {
    return operate(x, y)
}

// 定义一个操作函数
// @param  x 操作数1
// @param  y 操作数2
// @return 两个操作数相加的结果
func add(x, y int) int {
    return x + y
}

func main() {
    // 调用 calc 函数，把 add 函数传进去
    res := calc(10, 20, add)
    fmt.Println(res)    // 30
}
```

#### 函数作为返回值
```go
func add(x, y int) int {
    return x + y
}

func sub(x, y int) int {
    return x - y
}

// 定义一个函数，根据传入的操作符返回相应的操作函数
// @param s string 操作符
// @return func(int, int) int, error 操作函数
func do(s string) (func(int, int) int, error) {
    switch s {
    case "+":
        return add, nil
    case "-":
        return sub, nil
    default:
        return nil, errors.New("非法操作符")
    }
}

func main() {
    fn, err := do("+")
    if err == nil {
        fmt.Println(fn(10, 20))    // 20
    }
}
```

#### 闭包

!!! info
    a **closure** is a record storing **a function** together with **an environment**.

    **闭包** 是由 **函数** 和与其相关的引用 **环境** 组合而成的实体 。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201120192526963_12086.png)



闭包能将局部变量带出其作用域

```go
func adder() func() int {
    var x int
    return func() int {
                x++
                return x
            }
}

func main() {
    i := adder()
    fmt.Println(i()) // 1
    fmt.Println(i()) // 2
    fmt.Println(i()) // 3
}
```

相当于：
```go
func adder() func() int {
    var x int
    f := func() int {
        x++
        return x
    }
    return f
}

func main() {
    i := adder()
    i() // 1
    i() // 2
    i() // 3
}
```

等于说，在 `adder()` 中声明的变量`x`，本该在`adder()`调用完就被销毁，却被闭包带出了`x`的作用域（adder函数内），使得局部变量`x`没有被销毁。

即局部变量`x`逃逸了，它的生命周期没有随着它的作用域结束而结束。

### defer 语句

**defer** 就是延迟的意思，被 `defer` 关键字修饰的函数或方法会延迟执行。

```go
func main() {
    fmt.Println("start")
    defer fmt.Println(1)
    fmt.Println("end")
}

// ---------------------
// Output:
start
end
1
```
程序先执行第一句打印了 start，往下遇到 `defer`，于是把第二句先压入栈，然后继续执行第三句打印了 end。
再继续往下已经没有，函数准备结束；但是在结束之前需先出栈，也就执行了第二句，打印了 1。最后结束。

这就是 `defer` 语句。

!!! info 
    由于 `defer` 语句延迟调用的特性，所以 `defer` 语句能非常方便的处理资源释放的问题。比如：关闭文件、关闭数据库连接、资源清理、释放锁、记录时间等等。

    eg ：
    ```go
    func fn() {
        f, err := os.Open("./main.go")    // 打开文件
        defer f.Close()                   // 函数退出之前关闭文件

        if err != nil {
            fmr.Println(err)
            return
        }
    }
    ```

#### 多个 defer 的执行顺序
当有多个 `defer`时，按照栈的方式执行。即先`defer`的后执行。
eg：
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

#### defer 执行时机
Go 中的 `return`语句并不是原子操作，它分为 **给返回值赋值** 和 **RET指令** 两步。
而 `defer` 语句执行的时机在返回值赋值之后，RET指令执行之前。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201122103128567_27058.png)