---
title: 'Golang [基础] 9-指针'
seotitle: 'Golang [基础] 9-指针'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: 237b56c2
date: 2021-07-17 11:28:14
updated: 2021-07-17 11:28:14
---

指针的创建与使用

<!--more-->

# 9-指针

区别于C/C++中的指针，Go中的指针不能进行偏移和运算，是安全指针，

任何程序数据载入内存后，在内存都有他们的地址，这就是指针。
为了保存一个数据在内存中的地址，我们就需要指针。

Go 中的指针操作非常简单：`&`取地址符、`*`根据地址取值。

## 定义指针、指针取地址

{% note warning, **注意：指针是值类型！！！** %}

```go
var identifier *T
var identifier *T = &variable
identifier := &variable
```
`identifier`：指针名、`T`指针基类型、`*T`某类型指针、`variable`：变量、`&variable`：取变量地址

Go 中的值类型（int、float、bool、string、数组、struct结构体）都有对应类型指针，如 `*int、*float、*string、*bool、*[5]int`。

```go
func main() {
    i := 1
    var p *int = &i

    fmt.Printf("i:     %d \n", i)	   // i:    1 
    fmt.Printf("iaddr: %p \n", &i)     // addr: 0xc000012090 
    fmt.Printf("p:     %p \n", p)      // p:    0xc000012090
    fmt.Printf("paddr: %p \n", &p)     // addr: 0xc000006028 
    fmt.Printf("pint Type: %T \n", p)  // pint Type: *int
}
```

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201126192904446_12246.png)

## 指针取值

定义指针之后，就可以对指针进行操作了。
指针的操作有两种：`&`取地址、`*`根据地址取值

```go
func main() {

    // 指针取值
    a := 10
    b := &a
    fmt.Println(b)  // b 中保存着 a 的地址
    fmt.Println(&a) // 取 a 的地址
    fmt.Println(a)  // a 的值
    fmt.Println(*b) // 取 b
}

// ------------------------------------
// Output:
0xc000012090
0xc000012090
10
10
```
取地址操作符`&`和取值操作符`*`是一对互补操作符，`&`取出地址，`*`根据地址取出地址指向的值。

变量、指针地址、指针变量、取地址、取值的相互关系和特性如下：

- 对变量进行取地址（&）操作，可以获得这个变量的指针变量。
- 指针变量的值是指针地址。
- 对指针变量进行取值（*）操作，可以获得指针变量指向的原变量的值。

## 结构体指针

结构体指针访问结构体字段 同 结构体变量访问结构体字段一样，使用 `.` 点操作符，Go 没有 `->` 操作符。

eg:

```go
type User struct {    // 定义结构体
    name string
    age  int
}

Boii := User {    // 定义结构体变量
    name: "Boii",
    age:  18
}

p := &Boii    // 定义结构体指针

fmt.Println(p.name)    // 用 . 点操作符访问成员 Boii
```


## 指针做形参

```go
func fn1(x int) {
    x = 100
}

func fn2(x *int) {
    *x = 100
}

func main() {
    var a int = 10
    fn1(a)
    fmt.Println(a) // 10

    var b *int = &a
    fn2(b)
    fmt.Println(a) // 100
}
```

## 指针做返回值
Go 允许使用“栈逃逸”机制将局部变量的空间分配在堆上。
使用这种机制，需要返回指针

eg：
```go
func add(a, b int) *int {
    sum := a + b
    return &sum
}

func main() {
    psum := add(10, 20)
    fmt.Println(psum)  // 0xc000012090
    fmt.Println(*psum) // 30
}
```

## new 和 make

### new
new 是一个内置函数，它的函数签名如下：
```go
func new(Type) *Type
```
`Type` 表示类型，new函数只接收一个参数，这个参数只能是类型
`*Type` 表示类型指针，new 函数返回一个指向该类型内存地址的指针

new函数可以接受所有类型，使用new函数可以得到类型的指针，并且这个指针指向的内存地址中存着该类型的零值。
```go
func main() {
    a := new(int)
    b := new([5]int)
    c := new([]string)
    d := new(map[int]string)
    fmt.Printf("a type: %T \n", a) // a type: *int
    fmt.Printf("b type: %T \n", b) // b type: *[5]int
    fmt.Printf("c type: %T \n", c) // c type: *[]string
    fmt.Printf("d type: %T \n", d) // d type: *map[int]string

    fmt.Println(*a) // 0
    fmt.Println(*b) // [0 0 0 0 0]
    fmt.Println(*c) // []
    fmt.Println(*d) // map[]
}
```
new 函数会开辟一块空间，然后把这块空间的地址返回出去，这样外面的指针变量（a、b、c、d）就可以操作了。
如果仅仅只是声明一个指针变量，而没有开辟内存空间，这个指针变量是无法使用的。

- 以下为错误示例：

```go
func main() {
    var i *int
    fmt.Println(i)
    *i = 10    // 这里会引发 panic
}

// ------------------------------------
// Output:
<nil>
panic: runtime error: invalid memory address or nil pointer dereference
[signal 0xc0000005 code=0x1 addr=0x0 pc=0x97581b]

goroutine 1 [running]:
main.main()
    e:/---CODE/GO/src/Hello/main.go:23 +0x7b
exit status 2
```
从上面例子可以看到，声明了一个指针，默认值为 `nil`。
在没有分配内存的情况下去使用指针，会导致 `panic`。


- 以下为正确示例：

```go
func main() {
    var i *int 		    // 声明一个指针变量
    fmt.Println(i)	    // <nil> | 默认值为 nil
    i = new(int)	    // 给这个指针分配内存
    *i = 10			    // 存个值进去
    fmt.Println(i)	    // 0xc000012098 | 已经有内存空间了
    fmt.Println(*i)     // 10 | 正常~~
}
```

### make
make 也是用于分配内存，但是只能用于`slice切片、map字典、chan通道`的内存创建，而且它返回的类型 就是这三个类型本身，因为这三种类型就是引用类型，所以没必要返回他们的指针了。

make 的函数签名如下：
```go
func make(t Type, size ...IntegerType) Type
```

在使用 `slice、map、chan`的时候，都需要使用 make 进行初始化，然后才可以对它们进行操作

```go
func main() {
    c := new([]string)
    s := make([]string, 10)
    d := new(map[int]string)
    e := make(map[int]string)

    fmt.Printf("c type: %T \n", c) // c type: *[]string    | string切片类型的指针
    fmt.Printf("s type: %T \n", s) // s type: []string     | string切片
    fmt.Printf("d type: %T \n", d) // d type: *map[int]string | map类型的指针
    fmt.Printf("e type: %T \n", e) // e type: map[int]string  | map
}
```

### new 和 make 的区别
- new 返回的是**指针**，make返回的是**类型本身**
- new 可以给所有类型的**指针开辟空间**，make 只能用于 `slice、map、chan`的**初始化**
