---
title: 'Golang [基础] 4-流程控制'
seotitle: 'Golang [基础] 4-流程控制'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/cover/go2.png'
thumbnail: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/thumbnail/golang.png'
abbrlink: 7ebc0a4
date: 2021-07-17 10:27:41
updated: 2021-07-17 10:27:41
---

基本流程控制

<!--more-->

# 4-流程控制

## 分支结构

### if 基本写法
```go
if 表达式1 {
    分支1
} else if 表达式2 {
    分支2
} else{
    分支3
}
```

eg：
```go
func main() {
    score := 65

    if score >= 90 {
        fmt.Println("A")
    } else if score > 75 {
        fmt.Println("B")
    } else {
        fmt.Println("C")
    }
}
```

### if 特殊写法
在 if 表达式之前添加一个执行语句，再根据变量值进行判断
```go
func main() {

    if score := 65; score >= 90 {
        fmt.Println("A")
    } else if score > 75 {
        fmt.Println("B")
    } else {
        fmt.Println("C")
    }
}
```
注意上述代码中的 score 是在 if 语句中创建并初始化的，其作用域仅限于 if 语句块（包括同属的 else if 语句、else 语句）

但是如果声明是在 if 语句外，则其作用域不限于 if 语句中
```go
func main() {
    var score int

    if score := 65; score >= 90 {
        fmt.Println("A")
    } else if score > 75 {
        fmt.Println("B")
    } else {
        fmt.Println("C")
    }

    // score 在if外部声明，所以出了 if 依然有效
    fmt.Println(score)
}
```

### switch case
```go
switch [判断变量] {
    case 常量1 或 表达式1:
        执行语句1
    case 常量2 或 表达式2:
        执行语句2
    case 常量3 或 表达式3:
        执行语句3
    case 常量4 或 表达式4:
        执行语句4
    default:
        执行语句N
}
```
一个switch 可有多个分支，但只能有一个`default 分支`，且 defalut 分支不是必须的。

eg：
```go
func main() {
    num := 2

    switch num {
    case 1:
        fmt.Println("It is 1.")
    case 2:
        fmt.Println("It is 2.")
    case 3:
        fmt.Println("It is 3.")
    default:
        fmt.Println("It is more than 3.")
    }
}

// ----------------------------------------
// Output:
It is 2.
```

一个分支可以有多个值，每个值之间用逗号分开

eg：
```go
func main() {
    switch n := 7; n {
    case 1, 3, 5, 7, 9:
        fmt.Println("Odd")
    case 2, 4, 6, 8, 10:
        fmt.Println("Even")
    default
    }
}
```

分支也可以用表达式，这时 switch 后面不需要跟判断变量

eg：
```go
func main() {
    age := 25
    switch {    // switch 后不带判断变量，则case 只能判断 true 或 false
    case age < 25:
        fmt.Println("好好学习！")
    case age > 25 && age < 35:
        fmt.Println("好好工作！")
    case age > 60:
        fmt.Println("好好享受！")
    default:
        fmt.Println("好好活着！")
    }
}
```
或者
```go
func main() {

    switch age := 25; {
    case age < 25:
        fmt.Println("好好学习！")
    case age > 25 && age < 35:
        fmt.Println("好好工作！")
    case age > 60:
        fmt.Println("好好享受！")
    default:
        fmt.Println("好好活着！")
    }
}
```

switch 中的 case 默认不会穿透，执行完这个 case 之后就跳出switch，但是可以使用 `fallthrough` 击穿 case。

`fallthrough` 语句可以执行满足条件的 case 的下一个 case，这是为了兼容 C 语言中的 case 设计的。

```go
func switchDemo5() {
    s := "a"
    switch {
    case s == "a":
        fmt.Println("a")
        fallthrough
    case s == "b":
        fmt.Println("b")
    case s == "c":
        fmt.Println("c")
    default:
        fmt.Println("...")
    }
}
// ----------------------------------------
// Output:
a
b
```

## 循环结构
### for 循环
```go
for 初始化语句; 条件语句; 迭代语句 {
    ...
}
```
或
```go
初始化语句
for 条件语句 {
    ...
    迭代语句
}
```

eg：
```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```
或
```go
i := 0
for i < 5 {
    fmt.Println(i)
    i++
}
```

### 无限循环
```go
for {
    循环体
}
```
for 循环可以通过 `break、continue、return、panic`语句强制退出循环。

### for range 键值循环
使用 `for range` 可以遍历数组、切片、字符串、map 和通道channel。

通过 `for range` 遍历的返回值有以下规律：

1. 数组、切片、字符串 返回 **索引** 和 **值**
2. map 返回 **键** 和 **值**
3. 通道 channel 只返回 **通道内的值**

基本语法为：
```go
for idx, val := range iter {
    ...
}
```

eg：
```go
func main() {
    numbers := [6]int{1, 2, 3, 5}

    for i, x := range numbers {
        fmt.Printf("第 %d 位 x 的值为 %d \n",i ,x)
    }
}

// ----------------------------------------
// Output:
第 0 位 x 的值为 1
第 1 位 x 的值为 2
第 2 位 x 的值为 3
第 3 位 x 的值为 5
第 4 位 x 的值为 0
第 5 位 x 的值为 0
```

!!! warning "注意"
    `val` 始终为`iter`中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值。如果 `val` 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值。

    可以这么理解：
    ```go
    for i, val := range iter {
        ...
    }

    // 等价于
    var val someType
    for i, val = range iter {
        ...
    }

    // 等价于
    var val someType
    for i := 0; i < len(iter); i++ {
        val...
    }
    ```

    eg：
    ```go
    func main() {
        a, b, c := 1, 2, 3
        d, e, f := 4, 5, 6

        pointer := [3]*int{&a, &b, &c}
        valuer := [3]int{d, e, f}

        fmt.Println(a, b, c) // 1 2 3
        fmt.Println(d, e, f) // 4 5 6

        for _, p := range pointer {
            *p = *p + 10
        }

        for _, v := range valuer {
            v = v + 1
        }

        fmt.Println(a, b, c) // 11 12 13
        fmt.Println(d, e, f) // 4 5 6
    }
    ```



### goto 跳转
`goto语句` 通过标签进行代码间的无条件跳转。
标签即某一行第一个一冒号（`:`）结尾的单词,为了提升可读性，一般建议标签名称使用全大写。

`goto语句`可以在快速跳出循环、避免重复退出上有一定的帮助。
Go 语言中使用 goto 语句能简化一些代码的实现过程。

```go
func main() {
    for i := 0; i < 10; i++ {
        if i == 6 {
            goto GOTOTAG
        }
        fmt.Println(i)
    }

GOTOTAG:
    fmt.Println("I'm break.")
}

// ----------------------------------------
// Output:
0
1
2
3
4
5
I'm break.
```

---

使用 goto 语句也可以实现循环，但是不建议这么做，因为滥用 goto 语句会写出意大利面条代码
```go
func main() {
    a := 1
L:
    if a < 10 {
        fmt.Println(a)
        a++
        goto L
    }
}
```

如果您必须使用 goto，应当只使用正序的标签（**标签位于 goto 语句之后**），但注意标签和 goto 语句之间不能出现定义新变量的语句，否则会导致编译失败。

当然如果使用逆序的标签（**标签位于 goto 语句之前**）也是可以的，编译器并没有限制，例如 Gin 框架的路由中的实现就是逆序的。

### break、continue

**break**：跳出整个循环
**continue**：跳出本次循环

break 和 continue 语句后面都可以添加标签。**不过标签必须位于 `for、switch、select`代码块之上。**
eg：
```go
func main() {
breakTag:
    for i := 0; i < 10; i++ {
        if i == 5 {
            break breakTag
        }
        fmt.Println(i)
    }
}

// ----------------------------------------
// Output:
0
1
2
3
4
```
