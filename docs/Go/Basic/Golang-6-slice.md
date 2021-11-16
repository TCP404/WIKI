---
title: 'Golang [基础] 6-slice'
seotitle: 'Golang [基础] 6-slice'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: 6642a673
date: 2021-07-17 11:16:55
updated: 2021-07-17 11:16:55
---


<!--more-->

# 6-切片
> 切片是一个拥有相同数据类型元素的可变长度的序列。

数组是**固定**长度，切片是**可变**长度
数组是**值类型**，切片是**引用类型**

数组有很多局限性，切片非常灵活，支持自动扩容

切片内部结构包含 `地址`、`长度`、`容量`，一般用于快速操作一块数据集合。

## 创建切片

> **注意：切片是引用类型！！！**

```go
// 切片
var identifier []type
var identifier = []type{initial value}
identifier := []type{initial value}

// 数组
var identifier [len]type
var identifier = [...]type{initial value}
```
`identifier`：切片名、`type`：切片数据类型

> 区别于数组，切片在定义时不用填写 len。
> 它和初始化数组时省略 len 不同，数组省略 len 时要写 `...`，切片啥也不用写。


```go
func main() {
    var a []string				 // 声明一个字符串切片
    var b = []int{1, 2, 3, 4, 5} // 声明一个整型切片,并初始化
    c := []bool{false, true}	 // 声明一个布尔型切片,并初始化
    fmt.Println(a)				 // []
    fmt.Println(b)				 // [1 2 3 4 5]
    fmt.Println(c)				 // [false, true]
}
```

Slice 的创建方式有三种：

1. 通过下标的方式获得数组或切片的一部分
2. 使用字面量初始化新的切片
3. 使用关键字 `make` 创建切片

```go
arr := [8]int{1, 2, 3, 4, 5, 6, 7, 8}

s1 := arr[2:6]				// 1. 通过下标基于数组或切片创建
s2 := []int{11, 22, 33}		// 2. 通过字面量创建
s3 := make([]int, 10, 20)	// 3. 通过关键字 make 创建
```



### 基于数组创建

切片底层是数组，当底层数组不够的时候，切片就会扩容。

上面的定义是创建一个匿名数组，让切片指向这个匿名数组，下面是基于数组定义切片
```go
identifier := array[start_with:end:max]
```

```go
func main() {
    // 基于数组定义切片
    arr := [8]int{55, 56, 57, 58, 59, 60, 61, 62}
    b := arr[1:5]          // 其范围用数学表示为：[1,5) 从arr[1]取到arr[4]不包含arr[5]
    fmt.Println(b)         // [55 56 57 58 59]
    fmt.Printf("%T \n", b) // []int

    // 切片再次切片
    c := b[0:len(b)]       // len(b)为5，所以取了b[0]、b[1]、b[2]、b[3]、b[4]，相当于复制一整个切片
    fmt.Println(c)         // [55 56 57 58 59]
    fmt.Printf("%T \n", c) // []int
}
```

**直接定义切片** 和 **指定数组定义切片** 的区别在于：

1. 直接定义切片会引用一个匿名数组，指定数组定义切片会引用指定数组
2. 直接定义切片`长度=容量`，指定数组定义切片长度和容量视具体情况

直接定义切片 ↓
![直接定义切片](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201117165416502_24200.png)

指定数组定义切片 ↓
![指定数组定义切片](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201117165435737_31154.png)

![指定数组定义切片](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201117165353576_15789.png)

{% noteblock yellow %}
容量指的是从接片第一个元素到底层数组的最后一个元素
例如上面第三张图中：

```go
arr := [8]int{55, 56, 57, 58, 59, 60, 61, 62}
c := arr[3:6]

len(c)    // 3
cap(c)    // 5
```
{% endnoteblock %}


### 使用make()创建

如果需要**动态**的创建一个切片，可以使用内置的`make()`函数
基本语法为：

```go
make([]type, len, cap)

var identifier []type = make([]type, len, cap)
identifier := make([]type, len, cap)
```
`type`：切片数据类型、`len`：长度、`cap`：容量
cap 可以不填，默认和 len 相同

```go
func main() {
    // make函数构造切片
    // make([]type, len, cap)

    d := make([]int, 4, 10) // 构造一个整型切片，填充5个元素，最大容量10
    fmt.Println(d)         // [0 0 0 0]
    fmt.Printf("%T \n", d) // []int
    fmt.Println(len(d))    // 4
    fmt.Println(cap(d))    // 10
}
```

动态就动态在于它能使用变量哈哈哈
```go
func fn(a int, b int) []int {
    return make([]int, a, b)
}
```

## 切片是引用类型
### 判空
检查切片是否为空，不能用 `s == nil`，而是应该使用 `len(s) == 0`
> 切片是一种引用类型，当它被声明的时候，没有指向任何数组，包括匿名数组也没有，此时切片中指针为 `nil`
```go
var s []int    // s == nil
```

当切片被初始化的时候，它就指向了一个数组，这时切片中的指针不为 `nil`。
```go
var s = []int{}    // s != nil
```

### 切片不能直接比较
切片是一种引用类型，我们不能用 `==` 操作符来判断两个切片是否含有全部相等元素。
切片唯一合法的比较操作是和 `nil` 比较。

 一个`nil`值的切片并没有底层数组，一个`nil`值的切片的长度和容量都是0。
 但是我们不能说一个长度和容量都是0的切片一定是`nil`

```go
var s1 []int         //len(s1)=0; cap(s1)=0; s1==nil
s2 := []int{}        //len(s2)=0; cap(s2)=0; s2!=nil
s3 := make([]int, 0) //len(s3)=0; cap(s3)=0; s3!=nil
```
所以要判断一个切片是否是空的，要是用`len(s) == 0`来判断，不应该使用`s == nil`来判断。

### 切片的拷贝赋值
下面的代码中演示了拷贝前后两个变量共享底层数组，对一个切片的修改会影响另一个切片的内容，这点需要特别注意。
```go
func main() {
    s1 := make([]int, 3) //[0 0 0]
    s2 := s1             //将s1直接赋值给s2，s1和s2共用一个底层数组
    s2[0] = 100
    fmt.Println(s1) //[100 0 0]
    fmt.Println(s2) //[100 0 0]
}
```

## 切片遍历
切片的遍历方式和数组是一致的，支持索引遍历和`for range`遍历。
```go
func main() {
    s := []int{1, 3, 5}

    for i := 0; i < len(s); i++ {
        fmt.Println(i, s[i])
    }

    for index, value := range s {
        fmt.Println(index, value)
    }
}
```

## append()
Go 中的内建函数 `append()` 可以为切片动态添加元素，可以一次添加一个或多个元素。
```go
append(slice, elem_arr_or_slice)
```

eg：
```go
func main() {
    s := []int{1, 2, 3, 4, 5}   // [1 2 3 4 5]
    s = append(s, 11)			// [1 2 3 4 5 11]
    s = append(s, 12, 13, 14)   // [1 2 3 4 5 11 12 13 14]
}
```

> `append()` 第二个参数也可以是另一个切片，不过记得加上 `...`
```go
func main() {
    s2 := []int{55, 56, 57}
    s = append(s, s2...)
}
```

**注意：** 通过var声明的零值切片可以在 `append()` 函数直接使用，无需初始化。
```go
// 正确
var s []int
s = append(s, 1, 2, 3)

// X 没必要
var s = []int{}
s = append(s, 1, 2, 3)
// X 没必要
var s = make([]int)
s = append(s, 1, 2, 3)
```

## copy()

切片是引用类型，如果直接 `s1 = s2` ，其实是将 s2 中的数组地址赋值给 s1 的指针，s1 修改的时候 s2 也会受影响。
要实现真正的复制，需要使用内建函数 `copy()` 进行复制。
```go
copy(destSlice, srcSlice)
```
`destSlice`：目标切片、`srcSlice`：数据来源切片

```go
func main() {
    s1 := []int{1, 2, 3, 4}
    s2 := []int{12, 13, 14}
    copy(s1, s2)
    fmt.Println(s1)    // [12 13 14 4]
    fmt.Println(s2)    // [12 13 14]

    s3 := []int{1, 2, 3, 4}
    s4 := []int{12, 13, 14}
    copy(s4, s3)
    fmt.Println(s3)    // [1 2 3 4]
    fmt.Println(s4)    // [1 2 3]

    s5 := []int{1, 2, 3, 4}
    s6 := make([]int, 4)
    copy(s5, s6)
    fmt.Println(s5)    // [0 0 0 0]
    fmt.Println(s6)    // [0 0 0 0]

    s7 := []int{1, 2, 3, 4}
    s8 := make([]int, 4)
    copy(s8, s7)
    fmt.Println(s7)    // [1 2 3 4]
    fmt.Println(s8)    // [1 2 3 4]
    s8[0] = 100
    fmt.Println(s7)    // [1 2 3 4]
    fmt.Println(s8)    // [100 2 3 4]
}
```

## [:] 语法

这里需要说明一下，无论任何语言的 `[:]` 语法都是左闭右开的，用数学表达就是 $[start, end)$。

在 Python 中，``[:]``的语法规则是 `[low:high:step]`；

而在 Golang 中则是 `[low:high:max]`，其规则是：

`0 <= low <= len(arr) <= high <= max <= cap(arr)`。

- `low `的取值在 `0 至 底层结构长度`，取底层结构长度时为空切片

- `high`的取值在`low 至 max`，取 low 时为空切片
- `max`的取值在`high 至 底层结构容量`，取 high 时新切片 len==cap

举个栗子：

```go
func main() {
    a := [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    s := a[2:6:10]	// [low:high:max]

    fmt.Printf("a: %v\n", a)          // a: [0 1 2 3 4 5 6 7 8 9]
    fmt.Printf("a len: %d\n", len(a)) // a len: 10
    fmt.Printf("a cap: %d\n", cap(a)) // a cap: 10

    fmt.Printf("s: %v\n", s)          // s: [2 3 4 5]
    fmt.Printf("s len: %d\n", len(s)) // s len: 4
    fmt.Printf("s cap: %d\n", cap(s)) // s cap: 8
}
```

总结一下就是

- 新切片的容量 `cap(s) = max - low`
- 新切片的长度 `len(s) = high - low`
- `low 取值在 [0, len(arr)]`
- `high 取值在 [low, max]`
- `max 取值在 [hign, cap(arr)]`



## 删除

Go 并没有提供删除切片元素的方法，我们可以利用其本身的特性来删除元素
切片可以取自切片，那我们就将被删除元素之前的元素切下来，再把被删除元素之后的元素切下来，然后用`append()`拼接

```go
func main() {
    a := []int{30, 31, 32, 33, 34, 35, 36, 37}
    // 删除下标为2的元素
    a = append(a[:2], a[3:]...)
    fmt.Println(a)    // [30, 31, 33, 34, 35, 36, 37]
}
```
简单说就是 `a = append(a[:index], a[index+1:]...)`



## 其他操作

```go
// append slice
slice = append(slice, slice2...)
 
// copy
dest := make([]int, len(src))
copy(dest, src)

// 删除多个连续的元素
a = append(a[:i], a[j:]...)

// 删除一个元素 i
a = append(a[:i], a[i+1:]...)

// 扩展 j 个元素
a = append(a, make([]T, j)...)

// 插入元素 x 到 i 的位置上
a = append(a[:i], append([]T{x}, a[i:]...)...)

// push，追加到尾部
a = append(a, x)

// pop，尾部弹出
x, a = a[len(a)-1], a[:len(a)-1]

// dequeue，队头出队
x, a = a[0], a[1:]

// enqueue，队尾入队
a = append(a, x)


// shift，头部取出
x, a = a[0], a[1:]

// unshift，头部加入
a = append([]T{x}, a...)
```



## 总结

**切片和数组**之间有那么点像数据库中的 **基本表和视图**。
基本表是存储数据定义和数据的，视图只存储数据定义。所以基本表改变时视图也改变。
数组是存储定义和数据的，切片只存储定义。所以数组元素改变时切片也改变。

这个例子可能不是很贴切，但联系一下这句话：数组是值类型，切片是引用类型。
也就是说，数组是实实在在存数据的地方，切片只是对某个数组的引用，自己并没有数据。