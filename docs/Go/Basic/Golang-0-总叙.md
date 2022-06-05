# Hello Go

## 特性

- 简化问题，易于学习
- 内存管理，简洁语法，易于使用
- 快速编译，高效开发
- 高效执行
- 并发支持，轻松驾驭
- 静态类型
- 标准类库，规范统一
- 易于部署
- 文档全面
- 免费开源

## 环境变量

- **GOROOT**：Go的安装位置，默认 `#!shell $HOME/go`；
- **GOPATH**：工作目录，在这下面写代码；
- **GOBIN**：编译器和链接器的安装位置，默认 `$GOROOT/bin`；
- **GOOS**：运行Go的操作系统；
    - 取值：[darwin | freebsd | linux | windows]
- **GOARCH**：运行Go的处理器架构；
    - 取值：[386 | amd64 | arm]
- **GOARM**：专门针对基于 arm 架构的处理器，默认为 6；
    - 取值：[5 | 6]

## Hello World

```go
// src/Hello/main.go

package main    // (1) 

import "fmt"    // (2)

func main() {    // (3)
    fmt.Println("Hello World!")    // (4)
}
```

1. `package main` 定义了包名。必须在源文件中非注释的第一行指明这个文件属于哪个包，如：package main。
   package main 表示一个可独立执行的程序，**每个 Go 应用程序都包含一个名为 main 的包**。
2. `import "fmt"` 告诉 Go 编译器这个程序需要使用 fmt 包（的函数，或其他元素），fmt 包实现了格式化 IO（输入/输出）的函数。
3. `func main()` 是程序开始执行的函数。main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 init() 函数则会先执行 init() 函数）。
4. `fmt.Println(...)` 可以将字符串输出到控制台，并在最后自动增加换行字符 \n。使用 fmt.Print("hello, world\n") 可以得到相同的结果。

注意：

- 在 Go 中，左大括号不能单独一行，这会编译不了
- 对缩进没有要求，可以用 `go fmt 文件名.go` 命令将文件格式化
- 每个语句独占一行，不需要加分号
- 大部分语句只能写在 {} 内， {} 外只能做一些声明变量、函数、结构体的工作。
- 变量声明了不使用也是 **不允许** 的



### 运行

在文件目录下，可以运行 `go build` 命令，编译之后会在目录下生成一个可执行文件 `Hello.exe`，输入文件名即可以运行。

```shell
$ go build
$ ./Hello

Hello World!
```

如果不在文件目录下，也可以使用 `go build 项目名称` 命令，编译之后会在所在目录下生成一个可执行文件 `Hello.exe`。

```shell
$ cd ..
$ go build Hello
$ ./Hello

Hello World!
```

也可以使用 -o 参数来指定可执行文件的名字

```shell
$ go build -o haha.exe
$ ./haha

Hello World!
```

还可以使用 `go run 文件名.go` 命令来执行，但不会生成可执行文件。

```shell
$ cd src/Hello
$ go run main.go

Hello World!
```

## init 函数

每个源文件都只能包含一个 init 函数。
它不能被人为调用，而是在每个包完成初始化后自动执行，并且执行优先级比 main 函数高。
可以将数据检验或修复的工作放在 init函数中，这样程序开始执行之前就会先修复或检验数据，保证程序状态正确性。

```go
package main

import (
    "fmt"
    "math"
)

var Pi float64

func init() {
    Pi = 4 * math.Atan(1)
    fmt.Println("init()")
}

func main() {
    fmt.Println("main()")
}
```

输出：

```shell
$ go run main.go
init()
main()

$
```
