# w2-flag

flag 包专门用来解析命令行.

命令行最简单的组成，例如著名的系统提速命令：`#!bash rm -rf /*`，
这条命令行由 **命令**、**flag，或称 opetion**、**参数** 组成。

![系统提速命令](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/1623225138578.png)


我们写出来的命令行程序，在编译后会生成一个可执行文件，这个可执行文件就是命令，如：
```go
$ go build -o server main.go

$ ls
-rwxr--r--  2 boii boii 4.0K  3月  3 17:22 server
-rw-r--r--  2 boii boii 4.0K  3月  3 13:39 main.go

$ ./server
...
```
其中 server 就是一个命令，我们可以输入 `./server` 运行。

要编写一个命令行程序，就得提供一些参数方面的处理。那么首先就得先把参数解析出来。


解析参数有两种方式：

1. 通过 `os.Args` 获得所有参数，在通过下标得到每一个参数
2. 通过 `flag` 包获得


## os.Args

os.Args 是一个字符串切片 `[]string`，用法非常简单：

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    for i, arg := range os.Args {
        fmt.Printf("args[%d] is %v \n", i, arg)
    }
}
```

运行：
```bash
$ go build -o echo_args

$ ./echo_args 1 2 B o i i

args[0] is ./echo_args
args[1] is 1
args[2] is 2
args[3] is B
args[4] is o
args[5] is i
args[6] is i
```

## flag

flag 包提供了 `T()`系列、`TVar()`系列。

- `T()`系列基本格式：`#!go flag.T(参数名, 默认值, 帮助信息) *T`
- 具体有以下几种：
    - **flag.Bool()**
    - **flag.String()**
    - **flag.Int()**
    - **flag.Int64()**
    - **flag.Duration()**
    - **flag.Float64()**
    - **flag.Uint()**
    - **flag.Uint64()**
- `TVar()`系列基本格式：`#!go flag.StringVar(&T, 参数名, 默认值, 帮助信息)`
- 具体有以下几种：
    - **flag.BoolVar()**
    - **flag.StringVar()**
    - **flag.IntVar()**
    - **flag.Int64Var()**
    - **flag.DurationVar()**
    - **flag.Float64Var()**
    - **flag.UintVar()**
    - **flag.Uint64Var()**
- 区别：
    - `T()`系列返回一个 T型指针，需要一个变量去承接
    - `TVar()`系列不返回，变量是作为参数传进去的。

!!! example

    ```go
    func main() {
        // T() 系列
        name := flag.String("n", "Boii", "请输入名字")

        // TVar() 系列
        var sex string
        flag.StringVar(&sex, "s", "男", "请输入性别")

        flag.Parse()    // 得调用 Parse 才能解析命令行参数

        fmt.Println("name is", *name)
        fmt.Println("sex is", sex)
    }
    ```

    运行：

    ```bash
    $ go build -o flag_args

    $ ./flag_args -n Eva -s=女

    name is Eva
    sex is 女
    ```

### flag.Parse
`T()` 系列函数 和 `TVar()` 系列函数的作用的将参数和变量绑定起来（即注册），通过变量可以获取参数的值，真正的解析是 `Prase()` 函数。

`flag.Parse()` 从`os.Args[1:]`中解析注册的flag。必须在所有flag都注册好而未访问其值时执行。未注册却使用flag -help时，会返回ErrHelp。


### 参数形式
通过 flag 包这两个系列接收的命令行参数，在命令中可以使用以下4种方式：

1. `-flag xxx`
2. `-flag=xxx`（等号两边不能有空格，布尔类型参数必须用等号的方式）
3. `--flag xxx`
4. `--flag=xxx`（等号两边不能有空格，布尔类型参数必须用等号的方式）

```bash
$ ./flag_args -n Eva --s 女

$ ./flag_args -n=Eva --s=女
```

### 参数值
flag 包支持的命令行参数类型有 `bool、int、int64、uint、uint64、float、float64、string、time.duration`，对应的有效值如下

|        **flag**         | **有效值**                                                  |
| :---------------------: | :--------------------------------------------------------- |
|          bool           | 1,0,t,f,T,F,true,flast,TRUE,FALSE,True,False               |
| int、int64、uint、uint64 | 0b1011、1234、0755、0x12H等，可以是负数                        |
|      float、float64      | 合法浮点数                                                   |
|         string          | 合法字符串                                                   |
|      time.duration      | 合法时间段字符串；合法单位：ns、us、µs、ms、s、m、h。eg：3h32m、4d |


### 帮助信息

通过 `-h、--h` 或 `-help、--help` 可以获得帮助信息。

```bash
$ ./flag_args --help

Usage of ./flag_args:
  -n string
        请输入名字 (default "Boii")
  -s string
        请输入性别 (default "男")

$ ./flag_args -h

Usage of ./flag_args:
  -n string
        请输入名字 (default "Boii")
  -s string
        请输入性别 (default "男")
```

### 其他函数

```go
// Args returns the non-flag command-line arguments. 
func flag.Args() []string {}    // (1)

// NFlag returns the number of command-line flags that have been set.
func NFlag() int {} //(2)

// NArg is the number of arguments remaining after flags have been processed.
func flag.NArg() int {} //(3)
```

1. 返回不是使用 flag 的参数切片
2. 返回使用 flag 的参数的数量
3. 返回使用 flag 参数外剩下参数的数量，即 NArg = 参数总数 - NFlag

!!! example

    ```go
    // ./flag_args nn ss na nf nc
    // ./flag_args -n nn ss na nf nc
    // ./flag_args -n nn -s=ss na nf nc
    fmt.Println(flag.Arg(0))  // nn					ss				na
    fmt.Println(flag.Args())  // [nn ss na nf nc]  [ss na nf nc]   [na nf]	返回命令行参数后的其他参数
    fmt.Println(flag.NArg())  // 5					4				3		返回命令行参数后的其他参数个数
    fmt.Println(flag.NFlag()) // 0					1				2		返回使用的命令行参数个数
    ```

## 其他命令解析库

`flag` 是官方的库，出自Google，但是并不是最好的，有些局限性，而同门师弟 `pflag` 却做的更好，且兼容 `flag`，也被应用的更加广泛。

第三方库的 `spf13/cobra` 也是不错的解析库，Docker、Kubernetes、Hugo、Etcd 等程序都用 cobra 构建命令。

