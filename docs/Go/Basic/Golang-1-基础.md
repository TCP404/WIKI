---
title: 'Golang [基础] 1-基础'
seotitle: 'Golang [基础] 1-基础'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: 7b9344b7
date: 2021-07-17 10:20:39
updated: 2021-07-17 10:20:39
---

基础语法

<!--more-->

# 1-基础

## 标识符

- [ _ | A-Z | a-z | 0-9 ]
- 不能数字开头
- 不能与关键字和保留字冲突

## 关键字

Go 中有25个关键字

```go
    break        default      func         interface    select
    case         defer        go           map          struct
    chan         else         goto         package      switch
    const        fallthrough  if           range        type
    continue     for          import       return       var
```

37个保留字

```go
Constants:  true  false  iota  nil

    Types:  int  int8  int16  int32  int64
            uint  uint8  uint16  uint32  uint64  uintptr
            float32  float64  complex128  complex64
            bool  byte  rune  string  error

Functions:  make  len  cap  new  append  copy  close  delete
                complex  real  imag
                panic  recover
```

## 变量

### 声明

```go
var 变量名 变量类型
```

变量声明以关键字 `var` 开头，类型放在后面，无需分号

```go
var name string
var age int
var isEmpty bool
```

### 批量声明

每个变量写一个 var 太麻烦，go 支持批量声明

```go
var (
    name string
    age int
    isEmpty bool
)

var num1, num2, num3 int
```

### 初始化

> go 声明变量时，会自动对变量对应的内存区域进行初始化，每个变量都会被初始化成默认值。

eg：整型和浮点型变量默认值是 `0`，字符串变量默认值是 `空字符串`，布尔类型变量默认值是`false`，切片、函数、指针变量默认是 `nil`。

同时，我们可以在声明变量时为其指定初始值，格式如下：

```go
var 变量名 类型 = 表达式
```

eg：

```go
var name string = "Boii"
var age  int    = 18
```

或者一次初始化多个变量。eg：

```go
var name, age = "Boii", 18
```

> Go 语言具有的这种特性，使得交换两个变量非常方便，可以简单地使用 `a, b = b, a` 完成交换两个变量的值。

### 类型推导

类型推导指的是**没有显示的写明类型，编译器会通过等号右边的值来推导出类型**，然后完成初始化工作。

```go
var name = "Boii"
var age = 18
height := 180
```

### 交换变量

```go
a, b := 10, 15
a, b = b, a
fmt.Println(a, b)    // 15 10
```

### 短变量声明

在函数内部，可以用更简略的方式 `:=` 声明并初始化变量，但这种方式**只能声明局部变量**

```go
package main

import "fmt"

var m = 100    // 声明全局变量 m

func main() {
    n := 10    // 声明局部变量 n
    m := 200   // 声明局部变量 m
    fmt.Println(m ,n)
}

// ----------------------------------------
// Output:
200 10
```


### 匿名变量

使用多重赋值时，想要忽略某个值，可以使用 匿名变量，也就是一个下划线 `_` 表示。eg：

```go
func foo() (int, string) {
    return 18, "Boii"
}

func main() {
    x, _ := foo()
    fmt.Println("x =", x)
}
// ----------------------------------------
// Output:
18
```

匿名变量不占用命名空间，不会分配内存，所以匿名变量之间不存在重复声明，且匿名变量无法拿来使用，只能用来承接不想要的值。

### 总结

1. 函数外的每个语句都必须以关键字开始（var、const、func等）
2. `:=` 不能在函数外使用
3. `_` 多用于占位，表示忽略值

## 常量

常量是程序运行期间不会改变的那些值。
变量声明是用 `var`，常量声明是用 `const`
**常量在定义的时候必须赋值**

```go
const PI = 3.14156
const E  = 2.7182
```

批量声明

```go
const (
    PI = 3.14156
    E  = 2.7182
)
```

批量声明时，如果省略了赋值则表示与上面一行的值相同。eg：

```go
const (
    a = 100
    b
    c = 3.33
    d
    e
)
```

例子中，常量 `a` 和 `b` 的值都是 100，`c`、`d`、`e` 的值都是 3.33

### iota

`iota` 是go语言中的**常量计数器**，只能在常量的表达式中使用。

`iota` 在const 关键字出现时将被重置为 **0**。
const 中每新增一行常量声明，将使 `iota +1`
iota 可理解为const 块中的行索引
使用iota能简化定义，在枚举时很方便

```go
const (
    n1 = iota    // 0
    n2           // 1
    n3           // 2
    n4           // 3
)

func main() {
    fmt.Println(n1, n2, n3, n4)
}

// ----------------------------------------
// Output:
0 1 2 3
```

还有几个比较经典的例子:

- 匿名变量插队

    ```go
    const (
        n1 = iota // 0
        n2        // 1
        _
        n4        // 3
    )
    ```

- 声明中间插队

    ```go
    const (
        n1 = iota    // 0
        n2 = 100     // 100
        n3 = iota    // 2
        n4           // 3
        n5           // 4
    )
    const n6 = iota  // 0
    ```

- 定义数量级

    ```go
    const (
        _  = iota
        KB = 1 << (10 * iota)    // 10*1 == 10
        MB = 1 << (10 * iota)    // 10*2 == 20
        GB = 1 << (10 * iota)    // 10*3 == 30
        TB = 1 << (10 * iota)    // 10*4 == 40
        PB = 1 << (10 * iota)    // 10*5 == 50
    )
    ```

- 多个 `iota` 定义在一行

```go
const (
    a, b = iota + 1, iota + 2 //1,2
    c, d                      //2,3
    e, f                      //3,4
)

func main() {
    fmt.Println(a, b, c, d, e, f)
}

// ----------------------------------------
// Output:
1 2 2 3 3 4
```

- iota 只有在新起一行才会自增
- iota 只有遇到新的 const 才会重置
- iota 是从 0 开始的。 

## 值类型和引用类型

**值类型**：`int float bool string 指针`，使用这些类型的变量直接指向存在内存中的值，存储在栈中。
j = i
![ j = i ](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201115104832899_3842.png)

**引用类型**：`slices、maps、channel`，被引用的变量会存储在堆中，以便垃圾回收。
r1 = r2
![ r1 = r2 ](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201115104853017_32412.png)