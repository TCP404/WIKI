# 3-log

Golang 内置的 `log` 包实现了简单的日志服务。

## 导包
```go
import "log"
```

## 配置 logger
### 配置 Flag 选项
logger 默认只会提供日志的时间信息，如果需要更多的信息，如记录行号、文件名等等，可以手动设置。


```go
func Flags() int
func SetFlags(flag int)
```
`Flags` 函数会返回标准 logger 的输出配置；
`SetFlags` 函数用来设置 logger 的输出配置。

**Flag 选项**

```go
const (
	Ldate         = 1 << iota     // 日期: 2009/01/23
	Ltime                         // 时间: 01:23:23
	Lmicroseconds                 // 微秒级时间: 01:23:23.123123.  用于增强 Ltime 位
	Llongfile                     // 完整 文件名+行号: /a/b/c/d.go:23
	Lshortfile                    // 文件名+行号: d.go:23. 会覆盖掉 Llongfile
	LUTC                          // 使用 UTC时间
	Lmsgprefix                    // 将前缀从行的开头移至消息的开头
	LstdFlags     = Ldate | Ltime // 标准 logger 的初始值
)
```


默认配置：
```go
log.Println("This is a message.")

// Output: 
2021/03/01 07:59:36 This is a message.
```


配置后：
```go
log.SetFlags(log.Lshortfile | log.Lmsgprefix | log.Ltime | log.Ldate)
log.Println("This is a message.")

// Output: 
2021/03/01 08:42:49 main.go:17: This is a message.
```

!!! tip

    SetFlags 只控制输出的信息，不控制输出的顺序和格式。

### 配置日志前缀
`log` 库还提供了日志前缀相关的两个方法。

```go
func Prefix() string
func SetPrefix(prefix string)
```
`Prefix` 函数返回前缀，没有设置前缀时会返回空字符串
`SetPrefix`函数用于设置输出前缀。

!!! example

    ```go
    log.SetPrefix("[Boii] ")

    log.SetFlags(log.Lshortfile | log.Lmsgprefix | log.Ltime | log.Ldate)
    log.Println("This is a message.") 
    // Output:
    // 2021/03/01 08:54:23 main.go:20: [Boii] This is a message.

    log.SetFlags(log.Lshortfile | log.Ltime | log.Ldate)
    log.Println("This is a message.") 
    // Output:
    // [Boii] 2021/03/01 08:54:23 main.go:22: This is a message.
    ```

### 配置日志输出位置
日志默认是输出到控制台，通过 `SetOutput` 函数可以设置标准 logger 的输出目的地，比如设置输出到日志文件中。

```go
func SetOutput(w io.Writer)
```
`SetOutput` 函数接受一个 Writer对象，我们可以通过打开文件得到这个 Writer 对象。

```go
logF, err := os.OpenFile("./run.log", os.O_CREATE | os.O_WRONLY | os.O_APPEND, 0644)
if err != nil {
    fmt.Println("Open Log file failed, Err: ", err)
    return
}

log.SetOutput(logF)
```


### 配置在 init()

通常做法会把日志的配置信息放在 `init()` 函数中，这样可以在程序运行之前就已经配置好。

```go
func init() {
	logFile, err := os.OpenFile("./run.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
	if err != nil {
		fmt.Println("Open log file failed, Err:", err)
		return
	}

	log.SetFlags(log.Lshortfile | log.Lmsgprefix | log.Ltime | log.Ldate)
	log.SetPrefix("[Boii] ")
	log.SetOutput(logFile)
	log.Println("Log configured Successfully.")
}
```

## 创建 logger
你可以通过 `New()` 构造函数创建一个 logger 对象。

```go
func New(out io.Writer, prefix string, flag int) *Logger
```
`New`构造函数需要三个参数：输出位置、前缀、Flag。

!!! example

    ```go
    logger := log.New(os.Stdout, "[Boii] ", log.Lshortfile | log.Lmsgprefix | log.Ltime | log.Ldate)
    logger.Println("logger Println.")

    // Output: 
    2021/03/01 09:54:13 main.go:34: [Boii] logger Println.
    ```

## 使用 logger

使用日志可以通过调用 `log` 库的静态函数，或者创建 `logger` 对象，通过对象去调用各种方法。输出日志有 `Print 系列`、`Fatal系列`、`Panic系列`。

- **Print 系列**：输出日志后继续执行
    - log.Print
    - log.Printf
    - log.Println
    - logger.Print
    - logger.Printf
    - logger.Println
- **Fatal 系列**：输出日志后调用 `os.Exit(1)`
    - log.Fatal
    - log.Fatalf
    - log.Fatalln
    - logger.Fatal
    - logger.Fatalf
    - logger.Fatalln
- **Panic 系列**：输出日志后引发 `panic`
    - log.Panic
    - log.Panicf
    - log.Panicln
    - logger.Panic
    - logger.Panicf
    - logger.Panicln

```go
func main() {
	log.SetFlags(log.Lshortfile | log.Lmsgprefix | log.Ltime | log.Ldate)
	log.SetPrefix("[Boii] ")

	log.Print("log print")
	log.Printf("log %s", "printf")
	log.Println("log println")
	// 2021/03/01 10:06:12 main.go:23: [Boii] log print
	// 2021/03/01 10:06:12 main.go:24: [Boii] log printf
	// 2021/03/01 10:06:12 main.go:25: [Boii] log println

	log.Fatal("log Fatal, like Print")
	// 2021/03/01 10:06:12 main.go:27: [Boii] log Fatal, like Print
	// exit status 1
	log.Fatalf("log %s, like %s", "Fatalf", "Printf")
	log.Fatalln("log Fatalln, like Println")

	log.Panic("log Panic will panic after write down log.\n Also has Panicf / Panicln.")
	// 2021/03/01 10:07:06 main.go:36: [Boii] log Panic will panic after write down log.
	//  Also has Panicf / Panicln.
	// panic: log Panic will panic after write down log.
	//  Also has Panicf / Panicln.

	// goroutine 1 [running]:
	// log.Panic(0xc00008df68, 0x1, 0x1)
	//         /usr/local/go/src/log/log.go:354 +0xae
	// main.main()
	//         /home/boii/---FILE---/---CODE/GoProject/pkg/log_pkg/main.go:36 +0x13a
	// exit status 2
}
```

## 总结

Go内置的log库功能有限，例如无法满足记录不同级别日志的情况，我们在实际的项目中根据自己的需要选择使用第三方的日志库，如[logrus](https://github.com/sirupsen/logrus)、[zap](https://github.com/uber-go/zap)等。