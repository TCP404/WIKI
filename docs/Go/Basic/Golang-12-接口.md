---
title: 'Golang [基础] 12-接口'
seotitle: 'Golang [基础] 12-接口'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png
thumbnail: https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png
abbrlink: 19a13983
date: 2021-07-17 11:39:55
updated: 2021-07-17 11:39:55
---

接口的创建与使用

<!--more-->


# 12-接口

Go 中的接口 `interface `是一种类型，一种抽象的类型。

`interface` 是一组方法的集合，是 `dack-type programming` 的一种体现。
接口做的事情就像是定义一个协议（规则），只要一台机器具有洗衣服和甩干的功能，我就称其为洗衣机。
接口不关心属性（数据），只关心行为（方法）。

**只要一个结构体 X 实现了接口 A 中所有的方法，就称这个结构体 X 为接口 A 的实现类，称结构体 X 实现了接口 A。还可称结构体 X 是 A 类型。**

{% note warning, **为了保护你的Go语言职业生涯，请牢记接口（interface）是一种类型。** %}

## 定义接口

基本语法如下：

```go
type identifier interface {
    methodName1([param])[(return)]
    methodName2([param])[(return)]
    methodName3([param])[(return)]
    ...
}
```
`identifier`：接口名，命名时按照规范应该加上 `er`，如 有字符串功能的 `Stringer`、有读取功能的 `Reader`。
`methodName`：方法名

eg：
```go
type USBer interface {
    start()
    end()
}
```

## 实现接口
实现接口：
```go
// 定义一个接口
type USBer interface {
    start()
    end()
}
// 定义一个结构体
type Mouse struct {
    name  string
}

// 实现两个接口中的方法，接收者为结构体 Mouse，这样 Mouse 就是一个实现类
func (m Mouse) start() {
    fmt.Println(m.name, "starts working.")
}

func (m Mouse) end() {
    fmt.Println(m.name, "end working.")
}
```

结构体 `Mouse` 实现了 `USBer` 的所有方法 `start()` 和 `end()`，所以 `Mouse` 就是 `USBer` 的一个实现类。

## 使用接口

实现了接口的结构体（实现类）和没实现接口的结构体，区别在于
当一个函数需要传递一个接口类型的参数时，可以传递一个接口变量进去，也可以传一个实现类对象进去。

```go
func test(usb USBer) {    // 需要传递一个接口类型的参数
    usb.start()
    usb.end()
}

func main() {
    m1 := Mouse{"罗技"}
    test(m1)    // 传递一个实现类对象

    var u1 USBer = m1
    test(u1)    // 传递一个接口变量
}
```

{% noteblock warning %}
注意，
接口变量使用之前，需要先赋一个实现类对象。
接口变量可以调用接口中的方法，但不可以调用实现类的属性和方法
{% endnoteblock %}

```go
package main

import "fmt"

// 定义一个接口
type iUSBer interface {
    start()
    end()
}
// 定义一个结构体
type Mouse struct {
    name  string
}

// 实现两个接口中的方法，接收者为结构体 Mouse，这样 Mouse 就是一个实现类
func (m Mouse) start() {
    fmt.Println(m.name, "starts working.")
}

func (m Mouse) end() {
    fmt.Println(m.name, "end working.")
}

// 定义结构体 Mouse 自己的方法
func (m Mouse) selfMethod() {
    fmt.Println(m.name, "selfMethod")
}

// 定义一个测试方法，需要接收一个 USB 接口变量。可以传递 USB 变量，也可以传递 USB 实现类的对象
func test(usb iUSBer) {    // 需要传递一个接口类型的参数
    usb.start()
    usb.end()
}

func main() {
    m1 := Mouse{"罗技"}
    // Mouse是一个实现类，可以传递实现类对象给 test()
    test(m1)

    // 接口变量使用之前要先赋一个实现类对象
    var u1 iUSBer = m1
    // 传递一个接口变量给 test()
    test(u1)

    m1.start()           // 罗技 starts working.
    m1.end()             // 罗技 end working.
    m1.selfMethod()      // 罗技 selfMethod

    u1.start()           // 罗技 starts working.
    u1.end()             // 罗技 end working.

    // u1.selfMethod()   // 错误，接口变量不可以调用实现类的方法
    // u1.name           // 错误，接口变量不可以调用实现类的属性
}
```

## 空接口
不包含任何方法的接口，称为空接口。
我们可以认为 任意类型都实现了空接口

```go
interface {}

type T interface {}
```

eg：
```go
type T interface{}

type person struct {
    name string
}

var t1 T = person{"Boii"}
var t2 T = "str"
var t3 T = 100
var t4 T = []int{1, 2, 3}

fmt.Println(t1)    // {Boii}
fmt.Println(t2)    // str
fmt.Println(t3)    // 100
fmt.Println(t4)    // [1 2 3]
```

{% noteblock info %}
空接口的作用在于 **任意类型**。
当我们想要接受一个任意类型的参数，或者定义一个接收任意类型的容器，都可以使用空接口来代替。

```go
func test (a interface{}) {
    ...
}

test("string")
test(123)
test([]int{1, 2, 3, 4})
```

```go
a := []interface{}{1, 3, "str", true}
fmt.Println(a)    // {1 3 str true}

type T interface{}
b := []T{1, 3, "str", true}
fmt.Println(b)    // {1 3 str true}
```
{% endnoteblock %}

## 接口嵌套
接口中不仅可以定义方法签名，还可以定义其他的接口。这种方式我们称为**接口嵌套**。

```go
// 接口A
type A interface {
    method1()
}

// 接口B
type B interface {
    method2()
}

// 接口C，嵌套了接口 A 和 B
type C interface {
    A
    B
    method3()
}

// 结构体
type person struct {
    name string
}

func (p person) method1() { // 使 person 成为 接口A 的实现类
    ...
}

func (p person) method2() { // 使 person 成为 接口B 的实现类
    ...
}

func (p person) method3() { // 使 person 成为 接口C 的实现类，前提是实现了前面两个
    ...
}
```
嵌套了接口，不仅要实现接口本身的方法，还要实现接口中的嵌套接口的方法，才能认为实现了最外侧的接口。

如上面例子，person 要成为 C 的实现类，不仅要实现 C 的 `method3()` 方法，还得实现 A 的 `method1()` 和 B 的`method2()`。

```go
func main() {
    p := person{"Boii"}
    p.method1()
    p.method2()
    p.method3()

    var a1 A = p
    a1.method1()
    // a1.method2() // !报错
    // a1.method3() // !报错

    var b1 B = p
    b1.method2()
    // b1.method1() // !报错
    // b1.method3() // !报错

    var c1 C = p
    c1.method1() // 正确
    c1.method2() // 正确
    c1.method3() // 正确

    var a2 A = c1   // 接口C变量看成是接口A类型，这是可以的
    a2.method1()    // 但是只能调用接口A的 method1()
    // a2.method2() // !报错
    // a2.method3() // !报错

    var c2 C = a1   // !报错，接口A变量看成是接口C类型，这是错误的
}
```

## 接口断言

设 定义了一个接口A，还有两个结构体 X、Y，两个结构体都实现了接口A，当一个函数或方法想要接收 X 或 Y 两种类型时，可以将形参设置为 接口A 类型，这样传递的时候不管是 X 的对象还是 Y 的对象都可以传递进来。

【Q】：那如果在方法或函数里，我需要确定到底是 X 还是 Y 怎么办？

```go
type A interface {
    aMethod()
}

type X struct {
    name string
    age  int
}

type Y struct {
    name string
    sex  string
}

// 实现接口A
func (x X) aMethod() {
    fmt.Println("I am ", x)
}

func (y Y) aMethod() {
    fmt.Println("I am", y)
}

// 定义一个接收 X 和 Y 对象的函数
func test(a A) {
    a.aMethod()
}
```

【A】：这时候我们需要用到接口类型断言来确定传进来的到底是 x 还是 y。

- 方式1：
    1. `instance := 接口对象.(实际类型)`，这种不安全，会引发 panic()
    2. `instance, ok := 接口对象.(实际类型)`，这种就安全。接口对象是实际类型时，ok 为 true
- 方式2：switch
    ```go
    switch instance := 接口对象.(type) {
    case 实际类型1: ...
    case 实际类型2: ...
    ...
    }
    ```

于是在 `test()` 函数中我们可以这样写：
```go
func test (a A) {
    if ins, ok := a.(X); ok {
        fmt.Println(ins.age)
    } else if ins, ok := a.(Y); ok {
        fmt.Println(ins.sex)
    }
}
```

或者这样写：
```go
// 定义一个接收 X 和 Y 对象的函数
func test(a A) {
    switch ins := a.(type) {
    case X:
        fmt.Println(ins.age)
    case Y:
        fmt.Println(ins.sex)
    }
}
```

## 值类型实现 VS 指针类型实现

我们先定义一个接口，和两个结构体

```go
type Aer interface {
    method()
}

type X struct {}
type Y struct {}
```

然后分别使用 值类型 和 指针类型 实现接口

```go
// 值类型实现
func (x X) method() {}

// 指针类型实现
func (y *Y) method() {}
```
此时实现接口的是 `X` 类型、`*X` 类型 和 `*Y` 类型。

接着我们来使用一下：
```go
func main() {
	x := X{}
	y := Y{}

	var a1 Aer = x    // a1 可以接收 x 类型
	var a2 Aer = &x   // a2 可以接收 *X 类型

	var a3 Aer = y    // !报错，a3 不可以接收 y 类型
	var a4 Aer = &y   // a4 可以接收 *y 类型
}
```
在使用值类型实现接口之后，不管是 `X 结构体类型` 还是 `*X 结构体指针类型`，都可以赋值给该接口变量。
因为在 Go 语言中有对指针类型变量求值的语法糖，X 指针 `&x` 内部会自动求值。
`var a2 Aer = &x` 会变成 `var a2 Aer = *(&x)`。

而 Y 是用指针类型实现的，所以只能传递指针 `&y` 给 Aer 接口变量。

---

可以这么理解，结构体通过值接收者（Value receiver）实现了方法，编译器默认帮着实现了一个指针接收者（Pointer receiver）。

-   所以当一个结构体“**值实现**”了一个接口时，这个结构体不管是变量还是指针都可以传给这个接口类型。

-   当一个结构体“**指针实现**”了一个接口时，这个结构体只有指针可以传给接口类型。

如果一个结构体在实现一个接口的时候有的“值实现”，有的“指针实现”，那么只有结构体指针（struct pointer）实现了这个接口。例如：

```go
type Ber interface {
	f1()
	f2()
	f3()
}

type B1 struct{}

func (b *B1) f1() {}
func (b *B1) f2() {}
func (b B1)  f3() {}

func inter2(b Ber) {}

func main() {

	b1v := B1{}
	b1p := &B1{}

	inter2(b1p)	// 可以通过编译
	inter2(b1v)	// cannot use b1v (variable of type B1) as Ber value in argument to inter2: missing method f1 (f1 has pointer receiver)

}
```

## 结构体嵌套接口

我们说当一个结构体 `S` 实现了接口 `Ier` 的所有方法时，则结构体 `S` 实现了 `Ier` 接口。实现了接口以后，可以将结构体变量 `s1` 传递给接口变量 `ier1`，也可以调用接口 `Ier` 中的方法，当然具体执行的是 `s1` 实现的内容。

```go
type Ier interface {	// 接口 Ier
    method1()
    method2()
}

type S struct {}		// 结构体 S

func (s S) method1() {	// 结构体 S 实现了 Ier 接口
    fmt.Println("Hello1")
}

func (s S) method2() {	// 结构体 S 实现了 Ier 接口
    fmt.Println("Hello2")
}

func main() {
    s1 := S{}			// 结构体变量 s1
    var ier1 Ier = s1	// 结构体变量 s1 可以传递给接口变量 ier1
    
    s1.method1()		// 结构体变量 s1 可以调用 Ier 中的方法，执行结果打印 Hello1
    s1.method2()		// 结构体变量 s1 可以调用 Ier 中的方法，执行结果打印 Hello2
}
```

那如果在结构体中嵌入一个接口变量呢？

```go
type Ier interface {	// 接口 Ier
    method1()
    method2()
}

type S struct {			// 结构体 S
    Ier			// 嵌入接口变量
    name string
}
```

在结构体中匿名嵌入别的结构体相当于继承了这个结构体，是一种 `is-a` 的关系，上面的写法表明了：

>   结构体 `S` is a `Ier` 类型。

这样写，可以将结构体变量 `s1` 赋值给接口变量 `ier1`，但**无法调用接口中的方法**，因为结构体 `S` 没有实现。当然如果结构体 `S` 实现了方法，还是可以调用的。

```go
func main() {
    s1 := S{}			// 结构体变量 s1
    var ier1 Ier = s1	// 结构体变量 s1 可以传递给接口变量 ier1
    
    s1.method1()		// x 结构体变量 s1 不能调用 Ier 中的方法，因为没有实现
    s1.method2()		// x 结构体变量 s1 不能调用 Ier 中的方法，因为没有实现
}
```

>   由此我们也可以这么理解：
>
>   当一个结构体 `S` 实现了接口 `Ier` 中的所有方法，
>   相当于隐式地在 `S` 中嵌入一个匿名字段 `Ier`，
>   使得 `S is a Ier`，
>   于是结构体变量 `s1` 可以赋值给接口变量 `ier1`，可以调用接口方法。

