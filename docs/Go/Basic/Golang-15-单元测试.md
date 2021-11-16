---
title: 'Golang [基础] 15-单元测试'
seotitle: 'Golang [基础] 15-单元测试'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: d0d1ce
date: 2021-07-17 11:52:53
updated: 2021-07-17 11:52:53
---

单元测试的创建与使用

<!--more-->

# 单元测试

Golang 的单元测试依赖 `go test` 命令，编写测试代码和编写普通代码一样，只需要遵循一定的规则即可。

`go test` 命令是一个按照一定约定和组织的测试代码的驱动程序。

> 所有以 `_text.go` 为后缀的源代码文件都是 `go test` 测试的一部分，不会被 `go build` 编译到可执行文件中；

测试文件 `xxx_test.go` 中有三种类型函数：`单元测试函数`、`基准测试函数`、`示例函数`.

| 类型         | 前缀      | 格式                        | 作用                       |
|------------|-----------|-----------------------------|--------------------------|
| 单元测试函数 | Test      | TestTTT(t \*testing.T)      | 测试程序的逻辑性为是否正确 |
| 基准测试函数 | Benchmark | BenchmarkBBB(b \*testing.B) | 测试程序的性能             |
| 示例函数     | Example   | ExampleEEE()                | 为文档提供示例             |

在 `go test` 命令执行的过程中，会遍历所有 `_text.go` 文件中所有带有上述前缀的函数，生成一个临时 `main` 包用于调用相应的测试函数，然后编译、运行、报告测试结果、清理临时文件。

## go test 命令

基本命令是 `go test`，有很多参数可选，

```bash
go test [build/flags] [packages] [build/test flags & test binary flags]
```

- flags:
    `-v`: 打印测试过程
    `-cover`: 打印测试覆盖率
    `-bench`: 执行基准测试
    ...

- packages: packages 也就是要执行测试的文件，有两种模式：`当前目录模式（local directory mode）` 和 `包列表模式（package list mode）`

    - **当前目录模式**：没有填写具体的包名时就是这个模式。

        比如执行命令 `go test` 或 `go test -v` 时，就是这种模式。

        这种模式下会执行当前目录下的 `*_test.go` 文件的测试用例。

        默认不开启缓存，也就是不会缓存上次测试的结果。

    - **包列表模式**：填写了具体的包名时就是这个模式。

        比如执行命令 `go test .`、`go test ./.../...` 或 `go test package_name`

        这种模式下会执行指定的包中的测试文件。

        默认开启缓存，也就是会缓存上次测试的结果。可以通过 `-count=1` 关闭缓存。

## 单元测试

### 基本格式

1. 测试函数必须以 `Test` 开头
2. 测试函数参数必须包含一个形参 `*testing.T`
3. 测试函数没有返回值

```go
func TestFib(t *testing.T) {
    // ...
}

func TestSiztFmt(t *testing.T, n int) {
    // ...
}
```

参数 `t` 用于报告测试成功、失败、日志等信息，其包含如下方法：

```go
func (c *T) Error(args ...interface{})
func (c *T) Errorf(format string, args ...interface{})
func (c *T) Fail()
func (c *T) FailNow()
func (c *T) Failed() bool
func (c *T) Fatal(args ...interface{})
func (c *T) Fatalf(format string, args ...interface{})
func (c *T) Log(args ...interface{})
func (c *T) Logf(format string, args ...interface{})
func (c *T) Name() string
func (t *T) Parallel()
func (t *T) Run(name string, f func(t *T)) bool
func (c *T) Skip(args ...interface{})
func (c *T) SkipNow()
func (c *T) Skipf(format string, args ...interface{})
func (c *T) Skipped() bool
```



### 最简单的单元测试

编写一个斐波那契数列，然后编写单元测试。

```go
// fib.go
package fib

func Fib(n int) int {
    if n < 2 {
        return n
    }
    return Fib(n-1) + Fib(n-2)
}
```

```go
// fib_test.go
package fib

import "testing"

// 单元测试函数，以 Test开头
func TestFib(t *testing.T) {
    var (
        input    = 7	// 输入数据
        expected = 13	// 期望结果
    )
    actual := Fib(input)	// 程序执行结果
       // 如果与期望结果不一致，则输出错误提示。
    if actual != expected {
        t.Errorf("Fib(%d) = %d; expected %d", input, actual, expected)
    }
}
```

```shell
$ go test
PASS
ok      goTest  0.002s

$ pwd
~/---FILE---/---CODE/GoProject/playground/goTest

$ tree -L 2 .
goTest
├── fib.go
├── fib_test.go
└── go.mod
```

这是测试通过的结果，如果我们的 `Fib()` 写错了，则会输出与期望值不同的结果，并报告错误。

现在将 `Fib()` 稍微修改一下，测试文件不动，再次执行 `go test`

```go
// fib.go
package fib

func Fib(n int) int {
    if n < 2 {
        return n
    }
    // return Fib(n-1) + Fib(n-2)
    return Fib(n-1) + Fib(n-1)	// 修改一下
}
```

```shell
$ go test
--- FAIL: TestFib (0.00s)
    fib_test.go:12: Fib(7) = 64; expected 13
FAIL
exit status 1
FAIL    goTest  0.002s
```

### 测试组

单单测试一个例子不够，我们需要多测试几种情况，才能更好的保证我们程序的健壮性。

在单元测试函数里，我们可以按照测试用例定义一个结构体，然后创建一个结构体切片，将测试用例都放在里面。

遍历这个切片就可以测试这一整组测试用例。

```go
// fib_test.go
package fib

import "testing"

func TestFib(t *testing.T) {
    type test struct { // 定义一个测试用例结构体
        in   int
        want int
    }
    // 创建一个结构体切片存放所有测试用例
    tests := []test{
        {in: 7, want: 13},
        {in: 10, want: 55},
        {in: 1, want: 1},
        {in: 2, want: 1},
        {in: -1, want: -1},
    }
    // 遍历结构体切片，逐一测试每个测试用例
    for _, tc := range tests {
        if got := Fib(tc.in); got != tc.want {
            t.Errorf("Fib(%d) = %d; expected %d", tc.in, got, tc.want)
        }
    }
}
```

1. 定义测试用例结构体
2. 创建测试用例结构体切片，放置所有测试用例
3. 遍历切片逐一测试每个测试用例

```shell
# 测试成功
$ go test
PASS
ok      goTest  0.002s
```

接着修改 `Fib()` 让其出错，再进行测试

```shell
# 测试失败
$ go test
--- FAIL: TestFib (0.00s)
    fib_test.go:21: Fib(7) = 64; expected 13
    fib_test.go:21: Fib(10) = 512; expected 55
    fib_test.go:21: Fib(2) = 2; expected 1
FAIL
exit status 1
FAIL    goTest  0.002s
```

### 子测试

测试组在测试用例少的时候还比较好处理，但是用例多起来的时候就没法直接看出是哪些用例不通过了，所以我们可以使用官方提供的子测试。

还是一样定义一个测试用例结构体，但接着我们不是创建结构体切片，而是创建结构体 map 存放所有测试用例，遍历结构体 map，逐一调用 `t.Run()` 来测试每一个测试用例。

```go
// fib_test.go
package fib

import "testing"

func TestFib(t *testing.T) {
    type test struct { // 定义一个测试用例结构体
        in   int
        want int
    }
    // 创建一个结构体切片存放所有测试用例
    tests := map[string]test{
        "case7":         {in: 7, want: 13},
        "case10":        {in: 10, want: 55},
        "case1":         {in: 1, want: 1},
        "case2":         {in: 2, want: 1},
        "case_negative": {in: -1, want: -1},
    }

    for name, tc := range tests {
        t.Run(name, func(t *testing.T) {
            if got := Fib(tc.in); got != tc.want {
                t.Errorf("Fib(%d) = %d; expected %d", tc.in, got, tc.want)
            }
        })
    }
}
```



`t.Run()` 接受两个参数，一个是测试用例的名字，也就是 map 中的 key，另一个是一个 `func(t *testing.T)` 函数，在函数中写上具体的测试代码。

执行时带上 `-v` 参数，可以很直观看出哪些用例通过了 (PASS) 哪些没通过 (FAIL)

```shell
$ go test -v
=== RUN   TestFib
=== RUN   TestFib/case7
    fib_test.go:22: Fib(7) = 64; expected 13
=== RUN   TestFib/case10
    fib_test.go:22: Fib(10) = 512; expected 55
=== RUN   TestFib/case1
=== RUN   TestFib/case2
    fib_test.go:22: Fib(2) = 2; expected 1
=== RUN   TestFib/case_negative
--- FAIL: TestFib (0.00s)
    --- FAIL: TestFib/case7 (0.00s)
    --- FAIL: TestFib/case10 (0.00s)
    --- PASS: TestFib/case1 (0.00s)
    --- FAIL: TestFib/case2 (0.00s)
    --- PASS: TestFib/case_negative (0.00s)
FAIL
exit status 1
FAIL    goTest  0.002s
```

执行时带上 `-run` 参数还可以指定运行哪一个测试用例

```shell
$ go test . -v -run=TestFib/case7
=== RUN   TestFib
=== RUN   TestFib/case7
    fib_test.go:22: Fib(7) = 64; expected 13
--- FAIL: TestFib (0.00s)
    --- FAIL: TestFib/case7 (0.00s)
FAIL
exit status 1
FAIL    goTest  0.002s

$ go test . -v -run=Fib/case_negative
=== RUN   TestFib
=== RUN   TestFib/case_negative
--- PASS: TestFib (0.00s)
    --- PASS: TestFib/case_negative (0.00s)
PASS
ok      goTest  0.002s
```

`-run=RegExp`， `-run` 参数后带的是一个正则表达式，但是一般都像这样使用：`-run=X/Y`，其中：

- X 是 `TestAAA` 中的 `AAA`，或者直接写全: `TestAAA` ，

- Y 是测试用例结构体 map 的 key。如上面例子中 `-run=Fib/case_negative` 的 `case_negative` 就是 key。

### 覆盖率

覆盖率就是你的代码被测试用例覆盖的百分比。通常这个覆盖指 [语句覆盖](https://en.wikipedia.org/wiki/Code_coverage)。

输出覆盖率有两种方式：`-cover`、`-coverprofile=outfile`

- 使用 `-cover` 可以打印代码覆盖率。
- 使用 `-coverprofile` 可以将更详细的覆盖信息输出到指定文件中。



**-cover**

```shell
$ go test -cover
PASS
coverage: 100.0% of statements
ok      goTest  0.002s
```



**-coverprofile**

```shell
$ go test -coverprofile=fib.cover
$ tree -L 2 .
goTest
├── fib.cover
├── fib.go
├── fib_test.go
└── go.mod

$ go tool cover -html=fib.cover
```

使用 `go tool cover -html=制定文件` 命令会打开默认浏览器显示具体的覆盖率信息。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210623230022.png)

如上图，绿色表示被覆盖的语句，红色表示未被覆盖的语句，灰色表示未追踪的语句。



## 基准测试

### 基本格式

1. 测试函数必须以 `Benchmark` 开头
2. 测试函数参数只能包含一个形参 `*testing.B`
3. 测试函数没有返回值

```go
func BenchmarkFib(b *testing.B) {
    // ...
}

func BenchmarkSiztFmt(b *testing.B) {
    // ...
}
```

参数 `t` 用于报告测试成功、失败、日志等信息，其包含如下方法：

```go
func (c *B) Error(args ...interface{})
func (c *B) Errorf(format string, args ...interface{})
func (c *B) Fail()
func (c *B) FailNow()
func (c *B) Failed() bool
func (c *B) Fatal(args ...interface{})
func (c *B) Fatalf(format string, args ...interface{})
func (c *B) Log(args ...interface{})
func (c *B) Logf(format string, args ...interface{})
func (c *B) Name() string
func (b *B) ReportAllocs()
func (b *B) ResetTimer()
func (b *B) Run(name string, f func(b *B)) bool
func (b *B) RunParallel(body func(*PB))
func (b *B) SetBytes(n int64)
func (b *B) SetParallelism(p int)
func (c *B) Skip(args ...interface{})
func (c *B) SkipNow()
func (c *B) Skipf(format string, args ...interface{})
func (c *B) Skipped() bool
func (b *B) StartTimer()
func (b *B) StopTimer()
```

### 最简单的基准测试

编写一个容量格式化函数，然后编写对应的基准测试函数。

```go
// sizefmt.go
package sizefmt

import "strconv"

func sizeFmt(bit int64) string {
    const (	// 定义几个数量级
        _        = iota
        KB int64 = 1 << (10*iota + 3)
        MB
        GB
    )
    sizeFloat := float64(bit)
    unit := "b"
    switch {	// 根据 bit 参数的大小格式化不同的单位
    case bit < KB:
        return strconv.FormatInt(bit, 10) + unit
    case bit >= KB && bit < MB:
        sizeFloat /= 1 << 10
        unit = "K"
    case bit >= MB && bit < GB:
        sizeFloat /= 1 << 20
        unit = "M"
    case bit >= GB:
        sizeFloat /= 1 << 30
        unit = "G"
    }
    return strconv.FormatFloat(sizeFloat, 'f', 2, 64) + unit
}
```

然后编写基准测试函数。

```go
// sizefmt_test.go
package sizefmt

import "testing"
// 基准测试函数以 Benchmark 开头
func BenchmarkSizefmt(b *testing.B) {
    for i := 0; i < b.N; i++ {	// b.N 不是一个固定的值，它会不断迭代
        sizeFmt(1<<20 + 554)
    }
}
```

在 for 循环中使用到了一个 `b.N`，这个变量 不是一个固定的值，它会从 1 不断的迭代，例如 1、2、5、10、20、50...，保证基准测试函数的执行时间至少超过 1 秒钟。这样得到的数据才比较有参考性。

从 1 开始是保证被测试函数 `sizeFmt` 至少没问题能跑得动。



使用 `-bench=RegExp` 来指定只执行基准测试。

```shell
$ go test . -bench=Sizefmt
goos: linux
goarch: amd64
pkg: goTest
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkSizefmt-4       3589545               330.4 ns/op
PASS
ok      goTest  1.528s
```

- goos: 当前操作系统
- goarch: 当前芯片架构
- pkg: 基准测试函数所在的包
- cpu: 当前机器上 CPU 信息
- `BenchmarkSizefmt-4`: 对 `sizeFmt` 函数进行基准测试，当前 `GOMAXPROCS` 的值，这个对于开发基准测试很重要。
- `3589545`: 执行了3百多万次
- `330.4 ns/op`: 在这3百多万次调用里平均每次调用 `sizeFmt` 函数耗时 330.4 纳秒。
- `PASS`: 用于基准测试的用例执行通过
- `ok      goTest  1.528s`: 共耗时 1.528 秒

使用 `-benchmem` 参数可以获得内存分配的统计数据

```shell
$ go test . -bench=Sizefmt -benchmem
goos: linux
goarch: amd64
pkg: goTest
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkSizefmt-4   3589545	 330.4 ns/op	40 B/op		3 allocs/op
PASS
ok      goTest  1.528s
```

- `40 B/op`: 平均每次调用 `sizeFmt` 函数占用 40 个字节
- `3 allocs/op`: 每次调用 `sizeFmt` 需要申请 3 次内存

### 性能比较函数

上述例子我们得到的是给定例子的绝对耗时，但有些问题，例如斐波那契数列，数量不多的时候耗时还不会太长，但是当计算到 40 左右的时候就已经慢的不行了。

现在我们想要知道，当 `Fib(n)` 的 n 为 1、10、20、40、45 的时候他们的性能。

于是我们可以编写一个基准测试函数，不仅接受 `b *testing.B`，还接受另外一个参数 `n`，然后编写几个基准测试函数来测试。如下：

```go
// fib.go
package fib

func Fib(n int) int {
    if n < 2 {
        return n
    }
    return Fib(n-1) + Fib(n-2)
}
```



```go
// fib_test.go
package fib

import "testing"

// 把原本的基准函数变成小写开头，这样 -bench=Fib 的时候也不会直接匹配到
// 然后带上一个参数 n
func benchmarkFib(b *testing.B, n int) {
    for i := 0; i < b.N; i++ {
        fib(n)
    }
}

// 编写几个不同 n 的基准测试函数，每次去调用原本的基准测试函数
func BenchmarkFib1(b *testing.B)  { benchmarkFib(b, 1) }
func BenchmarkFib10(b *testing.B) { benchmarkFib(b, 10) }
func BenchmarkFib20(b *testing.B) { benchmarkFib(b, 20) }
func BenchmarkFib40(b *testing.B) { benchmarkFib(b, 40) }
func BenchmarkFib45(b *testing.B) { benchmarkFib(b, 45) }
```

然后执行命令：

```shell
$ go test -bench=Fib -benchmem
goos: linux
goarch: amd64
pkg: goTest
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkFibGreedy1-4    457008499         2.601 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy10-4   2589387           462.6 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy20-4   19940             57224 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy40-4   2         		865527961 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy45-4   1            9598796528 ns/op   0 B/op    0 allocs/op
PASS
ok      goTest  17.082s
```

可以看出，n 比较少的时候，执行的次数比较多，每次耗费的时间也不多。当 n 超过 40 以后，机器有点受不了了， 45 仅仅只执行了一次，还耗时 9 秒多。



主要的原因是采用了递归求斐波那契数列，调用栈很快就爆满了。

得到性能数据信息以后，我们就得知这个 `Fib` 在 n 稍大时效果不理想，应做改进。

递归属于分治法中的一种，我们可以换成动态规划来解决这个问题。

```go
// fib.go
package fib

// 用递归求斐波那契数列
func fibDAC(n int) int {
    if n < 2 {
        return n
    }
    return fibDAC(n-1) + fibDAC(n-2)
}

// 用动态规划求
func fibDP(n int) int {
    if n < 2 {
        return n
    }
    slice := make([]int, n)
    slice[0] = 1
    slice[1] = 1
    for i := 2; i < n; i++ {
        slice[i] = slice[i-1] + slice[i-2]
    }
    return slice[n-1]
}
```

```go
// fib_test.go
package fib

import "testing"

func benchmarkFibDAC(b *testing.B, n int) {
    for i := 0; i < b.N; i++ {
        fibDAC(n)
    }
}

func BenchmarkFibDAC1(b *testing.B)  { benchmarkFibDAC(b, 1) }
func BenchmarkFibDAC10(b *testing.B) { benchmarkFibDAC(b, 10) }
func BenchmarkFibDAC20(b *testing.B) { benchmarkFibDAC(b, 20) }
func BenchmarkFibDAC40(b *testing.B) { benchmarkFibDAC(b, 40) }
func BenchmarkFibDAC45(b *testing.B) { benchmarkFibDAC(b, 45) }

func benchmarkFibDP(b *testing.B, n int) {
    for i := 0; i < b.N; i++ {
        fibDP(n)
    }
}

func BenchmarkFibDP1(b *testing.B)    { benchmarkFibDP(b, 1) }
func BenchmarkFibDP10(b *testing.B)   { benchmarkFibDP(b, 10) }
func BenchmarkFibDP100(b *testing.B)  { benchmarkFibDP(b, 100) }
func BenchmarkFibDP1000(b *testing.B) { benchmarkFibDP(b, 1000) }
```

执行测试：

```shell
$ go test -bench=FibDP -benchmem
goos: linux
goarch: amd64
pkg: goTest
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkFibDP1-4       1000000000   0.4029 ns/op     0 B/op   0 allocs/op
BenchmarkFibDP10-4      20096688      51.87 ns/op    80 B/op   1 allocs/op
BenchmarkFibDP100-4     3030915       380.3 ns/op   896 B/op   1 allocs/op
BenchmarkFibDP1000-4    328071         3656 ns/op  8192 B/op   1 allocs/op
PASS
ok      goTest  5.356s

$ go test -bench=FibDAC -benchmem
goos: linux
goarch: amd64
pkg: goTest
cpu: Intel(R) Core(TM) i5-7300HQ CPU @ 2.50GHz
BenchmarkFibGreedy1-4    457008499         2.601 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy10-4   2589387           462.6 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy20-4   19940             57224 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy40-4   2         		865527961 ns/op   0 B/op    0 allocs/op
BenchmarkFibGreedy45-4   1            9598796528 ns/op   0 B/op    0 allocs/op
PASS
ok      goTest  17.082s
```

上面 FibDP 是对动态规划实现的基准测试，下面 FibDAC 是对递归实现的基准测试。可以看到差距十分巨大。



**注意不要拿 b.N 当参数**

```go
// 错误示范1
func BenchmarkFibWrong(b *testing.B) {
    for n := 0; n < b.N; n++ {
        Fib(n)
    }
}

// 错误示范2
func BenchmarkFibWrong2(b *testing.B) {
    Fib(b.N)
}
```





### 重置时间

`b.ResetTime()` 可以重置计时器，也就是这句语句前面的耗时不计入。所以可以执行一些不参与时间测试的工作，然后 `b.ResetTimer()`.

```go
func BenchmarkFib(b *testing.B) {
    fmt.Println("Start Benchmark test of Fib()")
    time.Sleep(time.Second)

    for i:=0; i<b.N; i++ {
        Fib(10)
    }
}
```



### 并行测试



## TestMain、Setup 和 Teardown

### TestMain 基本格式

1. 函数名为 `TestMain`
2. 函数参数必须包含一个形参 `*testing.M`
3. 函数没有返回值

```go
func TestMain(m *testing.M) {
    // ...
}
```

参数 `m` 用于启动测试，启动方法如下：

```go
func (m *M) Run() (code int)
```



有时候遇到测试之前需要做一些初始化工作或者测试之后需要做一些收尾工作，就可以用 `TestMain()` 。

如果测试文件中包含`TestMain()`，那么生成的测试将调用 `TestMain(m)`，而不是直接其他运行测试。

`m.Run()` 被调用后，将会运行测试文件中所有的测试函数。

`TestMain` 运行在主 goroutine 中 , 可以在调用 `m.Run` 前后做任何 Setup 和 Teardown 。

注意，在 `TestMain` 函数的最后，应该使用 `m.Run` 的返回值作为参数去调用 `os.Exit`。

另外，在调用 `TestMain` 时 , `flag.Parse` 并没有被调用。

所以，如果 `TestMain` 依赖于 command-line 标志（包括 `testing` 包的标志），则应该显式地调用 `flag.Parse`。注意，这里的依赖是指，若 `TestMain` 函数内需要用到 command-line 标志，则必须显式地调用 `flag.Parse`，否则不需要，因为 `m.Run` 中调用 `flag.Parse`。

### 示例

```go
// fib.go
package fib

func Fib(n int) int {
    if n < 2 {
        return n
    }
    return Fib(n-1) + Fib(n-2)
}
```



```go
// fib_test.go
package fib

import "testing"

// 单元测试函数，以 Test开头
func TestFib(t *testing.T) {
    var (
        input    = 7	// 输入数据
        expected = 13	// 期望结果
    )
    actual := Fib(input)	  // 程序执行结果
    if actual != expected {  // 如果与期望结果不一致，则输出错误提示。
        t.Errorf("Fib(%d) = %d; expected %d", input, actual, expected)
    }
}

func setup() {
    fmt.Println("设置一些东西")
}

func teardown() {
    fmt.Println("清理一些东西")
}

func TestMain(m *testing.M) {
    setup()
    code := m.Run()
    teardown()
    os.Exit(code)
}
```



```bash
$ go test
设置一些东西
PASS
清理一些东西
ok      goTest  0.002s
```



### 子测试的 Setup 和 Teardown

有时候不只是全局进行测试的时候需要 Setup 和 Teardown，有可能每个子测试在测试的时候也需要 Setup 和 Teardown。那可以对 Setup 和 Teardown 尝试以下这种优雅的写法：

```go
func setup() func() {
    fmt.Println("Setup")
    return func() {
        fmt.Println("Teardown")
    }
}
```

然后在子测试中这样使用：

```go
func TestFibDP(t *testing.T) {
    // 1 1 2 3 5 8 13 21 34 55
    tests := map[string]test{
        "case1": {1, 1},
        "case2": {2, 1},
        "case3": {3, 2},
        "case4": {7, 13},
        "case9": {9, 34},
    }

    for name, tc := range tests {
        t.Run(name, func(t *testing.T) {
            teardown := setup()
            defer teardown()
            if got := fibDP(tc.in); got != tc.want {
                t.Errorf("want: %v, got: %v", tc.want, got)
            } else {
                fmt.Println("yes!!!")
            }
        })
    }
}
```

效果如下：

```bash
$ go test -run=FibDP -v                                                                                     ──(日,7月11)─┘
=== RUN   TestFibDP
=== RUN   TestFibDP/case1
Setup
yes!!!
Teardown
=== RUN   TestFibDP/case2
Setup
yes!!!
Teardown
=== RUN   TestFibDP/case3
Setup
yes!!!
Teardown
=== RUN   TestFibDP/case4
Setup
yes!!!
Teardown
=== RUN   TestFibDP/case9
Setup
yes!!!
Teardown
--- PASS: TestFibDP (0.00s)
    --- PASS: TestFibDP/case1 (0.00s)
    --- PASS: TestFibDP/case2 (0.00s)
    --- PASS: TestFibDP/case3 (0.00s)
    --- PASS: TestFibDP/case4 (0.00s)
    --- PASS: TestFibDP/case9 (0.00s)
PASS
ok      goTest  0.002s
```



---

参考：

[李文周博客] : <https://www.liwenzhou.com/posts/Go/16_test/#autoid-3-1-0>
