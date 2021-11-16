---
title: 'Golang [nil] 交叉编译'
seotitle: 'Golang [nil] 交叉编译'
pin: false
tags:
  - Golang
  - 交叉编译
  - 条件编译
categories: [Golang, nil]
headimg: https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png
thumbnail: https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png
abbrlink: c8f1d592
date: 2021-07-27 12:42:33
updated: 2021-07-27 12:42:33
---

编译多平台程序的必备技能

<!--more-->
## 交叉编译

Golang 支持交叉编译，可以在一个平台上生成另一个平台的可执行程序。

Mac 下编译 Linux 和 Windows 64位可执行程序

```shell
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go12
```

Linux 下编译 Mac 和 Windows 64位可执行程序

```shell
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build main.go12
```

Windows 下编译 Mac 和 Linux 64位可执行程序

```cmd
SET CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go

SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```

上面的命令编译 64 位可执行程序，也可以使用 386 编译 32 位可执行程序。



## 条件编译

Golang 虽然能跨平台编译但是也无法完全解决平台之间的差异，不过 Golang 能够选择性编译。

当我们写的程序在不同的平台上调用的东西不同时，比如编译 Linux 平台的可执行文件的时候要编译这个文件，编译 Windows 平台的可执行文件的时候要编译那个文件。这种情况下就可以使用选择性编译。

选择性编译有两种方式： `编译注释（构建约束）` 和 `文件后缀`。

### 编译注释（构建约束）

编译注释长这样：
```go
// +build darwin freebsd

// sort package is used to sort.
package sort
```
**第一行就是编译注释，或者叫构建约束（build constraint）、构建标记（build tag）。** 

格式为：
```go
// +build [$GOOS] [$GOARCH]
```

这是 Go 一开始就有的特性，在 Go 源码中有很多这样的注释行。上面注释行的意思，这个文件只在 darwin 系统或 freebsd 系统下会包含在包中，其他系统会忽略这个文件。

它需要遵循以下几点：

1. 必须在文件顶部附近，它的前面只能有空行或其他注释行；可见包子句也在约束之后
2. 约束可以出现在任何源文件中，比如 `.go`、`.s` 等
3. 约束可以有多行
4. 为了区别约束和包文档，在约束之后必须有空行
5. `//` 后面必须留一个空格
6. `+build` 是关键字，声明这是一个编译注释
7. 逻辑关系:
   1. `!`: 非（感叹号）
   2. `,`: 与（逗号）
   3. `  `: 或（空格）
8. $GOOS 取值：
   1. darwin
   2. dragonfly
   3. freebsd
   4. linux
   5. netbsd
   6. openbsd
   7. plan9
   8. solaris
   9. windows
9. $GOARCH 取值：
   1. 386
   2. amd64
   3. arm

> GOOS 和 GOARCH 的取值可以通过命令 `go tool dist list` 查看

#### 示例
```go
// +build A,B !C,D 

代表编译此文件需符合 (A且B) 或 ((非C)且D) 。
```

```go
// +build !windows,arm
代表此文件在 系统不是windows，并且处理器架构是arm时编译
```

### 文件后缀
通过在文件名后加上 `_$GOOS.go` 的方式也可以实现条件编译。

后缀有3种形式：
1. `_$GOOS.go`
2. `_$GOARCH.go`
3. `_$GOOS_$GOARCH.go`

#### 示例

```
color_windows.go    // 该文件仅在编译 windows 可执行文件时编译
color_linux.go      // 该文件仅在编译 linux 可执行文件时编译
color_darwin.go     // 该文件仅在编译 darwin 可执行文件时编译
color_linux_arm.go  // 该文件仅在编译 linux 可执行文件时，且处理器架构为 arm 时编译
```

### 如何选择
如果是排除某一两个平台的情况
如果是在很多个平台都可用的情况
则使用 `编译注释`

如果是指定某一个平台的情况
如果是指定某一种架构的情况
则使用 `文件后缀`

### 其他文件名

以 `下划线` 或 `点` 开头的文件不会被编译。
```
_file.cfg
.set.vm
```
这两个文件在编译时不会被编译。

## 新版构建约束

自 Go1.17 开始，Golang 调整了构建约束，也就是编译注释。

```go
//go:build [$GOOS] [$GOARCH]
```

### 对比

旧版：

```go
// +build linux,amd64
```

新版：

```go
//go:build linux && amd64
```

主要改变：

1. `// +build` 改成 `//go:build`，双斜杠后**没有**空格！！

2. 一个文件只能有一行构建语句。（旧版可以多行）

3. gofmt 工具会自动根据旧版语法生成对应的新版语法，并且为了兼容两者都保留，新版在支持的环境下会覆盖旧版。

    ```go
    // +build !freebsd, !plan9
    ```

    gofmt 后：

    ```go
    //go:build !freebsd && !plan9
    // +build !freebsd, !plan9
    ```