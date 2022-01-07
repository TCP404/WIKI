# 1-random

## 导包

```go
import "math/rand"
```

## 设定随机种子

生成随机数需要设定 **随机种子**，如果不设置的话会导致运行多少次生成的随机数都一样。

设置随机种子需要调用 `Seed()` 函数。

函数签名：

```go
func Seed(seed int64) {}
```

随机种子可以设置一个固定的数，但那样更没设定一样。所以最好传递一个时时刻刻不断变化的参数，如 时间、股价等等

!!!example

    ```go
    rand.Seed(time.Now().Unix())    // 使用时间戳
    ```

## 生成随机数
生成随机数可以有多种类型，通过不同的函数获得。

函数签名：

```go
func Int() int             // 生成一个 int    表示范围内的 有符号随机整数
func Int31() int32         // 生成一个 int32  表示范围内的 有符号随机整数
func Int63() int64         // 生成一个 int64  表示范围内的 有符号随机整数
func Uint32() uint32       // 生成一个 uint32 表示范围内的 无符号随机整数
func Uint64() uint64       // 生成一个 uint64 表示范围内的 无符号随机整数
func Float32() float32     // 生成 [0.0,1.0) 的 float32 型随机浮点数
func Float64() float64     // 生成 [0.0,1.0) 的 float64 型随机浮点数

func Intn(n int) int       // 生成 [0,n) 的 int 型随机整数
func Int31n(n int32) int32 // 生成 [0,n) 的 int 型随机整数
func Int63n(n int64) int64 // 生成 [0,n) 的 int 型随机整数

// 生成一个 [-math.MaxFloat64, +math.MaxFloat64] 的 随机浮点数
// 其生成结果呈 标准正态分布
// 通过 NormFloat64() * desiredStdDev + desiredMean 可以修改正态分布的样本
func NormFloat64() float64

// 生成一个 (0, +math.MaxFloat64] 的随机浮点数
// 其生成结果呈 指数分布（速率为1）
// 通过 ExpFloat64() / desiredRateParameter 可以修改指数速率
func ExpFloat64() float64 
```

!!! example

    ```go
    rand.Seed(time.Now().Unix())    // 设置随机种子

    fmt.Println(rand.Int())         // 1971404210255330026
    fmt.Println(rand.Int31())       // 1005921906
    fmt.Println(rand.Int63())       // 8146236132046518471
    fmt.Println(rand.Uint32())      // 2116138919
    fmt.Println(rand.Uint64())      // 182541428944395698
    fmt.Println(rand.Float32())     // 0.2112303
    fmt.Println(rand.Float64())     // 0.7190299088870225

    fmt.Println(rand.Intn(10))      // 6
    fmt.Println(rand.Int31n(10))    // 1
    fmt.Println(rand.Int63n(10))    // 3

    fmt.Println(rand.NormFloat64()) // -0.3986721754620751
    fmt.Println(rand.ExpFloat64())  // 2.4778555651486647
    ```

## 调整范围
可控范围的 `Intn(n)、Int31n(n)、Int63n(n)`只能生成 0 到 n 范围内，如果起始范围有所不同，则需要稍作修改。

生成范围在 `[0, 9)` 时：
```go
x := rand.Intn(10)
```

生成范围在 `[5, 18)` 时：
```go
x := rand.Intn(13)+5    // [0+5, 13+5)
```

生成范围在 `[a, b)` 时：
```go
x := rand.Intn(b-a) + a    // [0+a, b-a+a)
```

生成范围在 `[a, b]` 时：
```go
x := rand.Intn(b-a+1) + a    // [0+a, b+1)
```

