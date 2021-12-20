# 11-包
在工程化的Go语言开发项目中，Go语言的源码复用是建立在包（package）基础之上的。本文介绍了Go语言中如何定义包、如何导出包的内容及如何导入其他包。

包（package）是多个 Go 源码的集合，是一种高级的代码复用方案，例如 Go 就提供了很多内置包，如`fmt`、`os`、`io`等。

## 定义包

一个包可以简单理解为一个存放 `.go` 文件的目录。
该目录下面所有的go文件都要再代码第一行添加 `package 包名`，声明该文件归属的包。

!!! warning "注意"
    1. 一个目录下面直接包含的文件只能归属一个 `package`
    2. 同一个 `package` 的文件不能放在多个目录下
    3. 包名可以不和目录名一样，但不能包含 `-` 符号
    4. `main 包` 为程序的入口包，这种包编译后会得到一个可执行文件
    5. 编译不含 `main 包` 的源代码不会得到可执行文件

```
demo
├── utils
│   ├── calc.go     // package utils
│   └── data.go     // package utils
├── log
│   ├── debug.go    // package log
│   └── info.go     // package log
├── main.go    // package main
└── test.go    // package main
```

## 可见性
在一个包中引用另外一个包里的标识符（变量、函数、类型等），该标识符必须是对外可见的，即首字母大写的。

```go
package model

import "fmt"

var a = 100             // 首字母小写，外部包不可见，只能在当前包内使用

const PI = 3.14         // 首字母大写，外部包可见，可在其他包中使用

type person struct {    // 首字母小写，外部包不可见，只能在当前包内使用
    Name string         // 首字母大写，外部包可见，可在其他包中使用
    age  int8           // 仅限包内访问的字段
}

type Payer interface {  // 首字母大写，外部包可见，可在其他包中使用
    init()              // 仅限包内访问的方法
    Pay()               // 可在保外访问的方法
}

func Add(x, y int) int { // 首字母大写，外部包可见，可在其他包中使用
    return x + y
}

func age() {             // 首字母小写，外部包不可见，只能在当前包内使用
    var Age = 18         // 函数局部变量，外部包不可见，只能在当前函数内使用
    fmt.Println(age)
}
```

## 导入包

```go
import "包的路径"
```
!!! warning "注意"
    1. import 语句通常放在文件开头、包声明语句下面。
    2. 包的路径需要双引号括起来
    3. 包名是从 `$GOPATH/src/` 后面开始计算的，使用 `/` 分割路径，这是绝对路径；也可以使用相对路径
    4. 禁止互相导包

### 单行导入
```go
import "包1"
import "包2"
```

### 多行导入
```go
import (
    "包1"
    "包2"
)
```

### GO MODULE 模式下导入自己的包
使用 Go Module 模式管理包时，如果想导入自己写的包，路径应为 `module名/路径...`

eg：

```
├── go.mod     // module Stack
├── main.go    // package main
└── seqStack
    └── SeqStack.go    // package seqStack
```

从项目根目录下开始，有一个 `main.go` 文件，还有一个子包 `seqStack`。
现在我想在 `main.go` 中使用 `SeqStack.go` 中的函数，需要在 `main.go` 中导入 `seqStack` 这个包。

分为以下三步：

1. 在 `go.mod` 中写下：

```go
module Stack    // 声明 module 名

go 1.16

...
```
2. 在 `SeqStack.go` 中声明：

```go
package seqStack    // 声明子包名

...
```

3. 在 `main.go` 中导入：

```go
package main    // 声明 main 包

import "Stack/seqStack"    // 正确导入

...
```


## 别名
导包的时候，可以给包设置别名。常用于处理包名太长或包名冲突的情况。
```go
import 别名 "包的路径"
```

### 单行
```go
import "fmt"
import t "tcp404.com/study/go/test"

func main() {
    fmt.Println(t.Add(10, 20))
}
```

### 多行
```go
import (
    . "fmt"    // 使用 . 作为别名，则使用时可以省略包名
    t "tcp404.com/study/go/test"
)

func main() {
    Println(t.Add(10, 20))
}
```

### 省略包名
在给包起别名的时候，别名为 `.` ，在后面使用时可以直接使用包中的变量和方法等，不需要加包名。

```go
import . "fmt"

func main() {
    Println("Hello!")
}
```


## init() 初始化函数


!!! note
    - Go 在执行导包之前，会自动触发包内的 `init()` 函数的调用。
    - `init()` 没有参数也没有返回值，自动被调用，不能主动调用，`main()` 也是。
    - 一个包内的`init()` 可以有多个，`main()` 只能有一个。
    - 一个包可以被多个包导入，但是 `init()` 只会执行一次。
    - 包内多个 `init()` 的执行顺序不固定，目前（Go 1.16.5）是按照文件名升序，a最先执行，z最后执行。

每个 `.go` 文件都可以有自己的 `init()` ，

```go
package main

import "fmt"

// 全局变量
var age int8 = 18
const E = 2.78

// init函数
func init() {
    fmt.Println(E)
}

// main函数
func main() {
    fmt.Println("Hello World")
}
```



!!! note "一个包中的执行顺序为："
    **1. 全局声明(const -> var) -> 2. init() -> 3. main()**

多个包有导入关系时，init() 执行顺序：从main 包开始，按照 main 包的 **import 的顺序 广度优先** 一层层往下找
![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201125164650876_22585.png)


## 匿名导包
有时候只想导入一个包执行一下它的 `init()` 函数，并不使用包里的东西，可以使用匿名导包。
```go
import _ "包的路径"
```
匿名导入的包与其他方式导入的包一样都会被编译到可执行文件中。
