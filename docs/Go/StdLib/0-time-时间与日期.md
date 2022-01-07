# 0-time

!!! tip
    - 1年 = 365天，day     -> `d`
    - 1天 = 24小时，hour  -> `h`
    - 1小时 = 60分钟，minute  -> `m`
    - 1分钟 = 60秒，second     -> `s`
    - 1秒钟 = 1000毫秒，millisecond     -> `ms`
    - 1毫秒 = 1000微秒，microsecond   -> `μs`
    - 1微秒 = 1000纳秒，nanosecond    -> `ns`
    - 1纳秒 = 1000皮秒，picosecond     -> `ps`


## 导包
```go
import "time"
```
## 获取时间
### 获取当前时间 `#!go time.Now()`
函数签名：
```go
func Now() Time
```

!!! example

    ```go
    t1 := time.Now()
    fmt.Println(t1) //(1)
    ```

    1. 2021-02-08 11:54:51.5596954 +0800 CST m=+0.001962701


### 获取指定时间 `#!go time.Date()`
函数签名：
```go
func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time
```

!!! example

    ```go
    t2 := time.Date(2021, 10, 1, 9, 10, 32, 11, time.Local)
    fmt.Println(t2) // (1)
    ```

    1. 2021-10-01 09:10:32.000000011 +0800 CST


## 获取时间戳

### 1970年时间戳 `#!go t.Unix()、t.UnixNano()`
方法签名：
```go
func (t Time) Unix() int64 {}
func (t Time) UnixNano() int64 {}
```

- 作用：指定的日期距离1970年1月1日 0时0分0秒 的时间差值。
- `Unix()`：返回 秒 的差值；
- `UnixNano()`：返回 纳秒 的差值。

!!! example 

    ```go
    t4 := time.Date(1970, 1, 1, 1, 0, 0, 0, time.UTC)
    fmt.Println(t4.Unix())	    // (1)
    fmt.Println(t4.UnixNano())	// (2)
    ```

    1. 3600，相差1小时，也就是3600秒
    2. 3600 000 000 000，相差1小时，也就是36万亿纳秒

### 当前时间戳 `#!go time.Since()、time.Until()`
函数签名：
```go
func Since(t Time) Duration {}
func Until(t Time) Duration {}
```

- 作用：返回当前时间 和 t 之间的时间差值
- `Since` 相当于 `time.Now().Sub(t)`，返回当前之间减去 t 的差
- `Until` 相当于 `t.Sub(time.Now())`，返回 t 减去当前时间的差

!!! example

    ```go
    t3 := time.Parse("2006/1/2", "2021/2/8")
    fmt.Println(t3)             // (1)
    fmt.Println(time.Since(t3)) // (2)
    fmt.Println(time.Until(t3)) // (3)
    ```

    1. 2021-02-08 00:00:00 +0000 UTC
    2. 8h21m9.0061743s = 当前时间 - t3
    3. -8h21m9.0061743s = t3 - 当前时间


## 时间格式化
### 时间转字符串 `#!go t.Format()`
方法签名：
```go
func (t Time) Format(layout string) string
```

- 作用：调用此方法的时间对象会根据给定的字符串模板来格式化出一个字符串格式的时间。
- `layout`：字符串模板。

!!! warning
    字符串模板的日期时间必须是 **『2006年1月2日 15点04分05秒』**，这是Go语言诞生的时间。（官方彩蛋）
    
    也可以写成 3 点： **『2006年1月2日 03点04分05秒』**。

    记忆方法：6-1-2-3-4-5

!!! example

    ```go
    // time -> string
    // 模板日期必须是 06-1-2-3-4-5
    t1 := time.Now()
    s1 := t1.Format("2006年1月2日 15:04:05")    // (1)
    fmt.Println(s1) // (2)

    s2 := t2.Format("2006-1-2")
    fmt.Println(s2) // (3)
    ```

    1. 模板日期必须是 06-1-2-3-4-5
    2. 2021年2月8日 11:56:27
    3. 2021-10-1
### 字符串转时间 `#!go time.Parse()`
函数签名：
```go
func Parse(layout, value string) (Time, error)
```

- 作用：根据模板将一个字符串转为时间格式
- `layout`：字符串模板。时间必须是 2006年1月2日 15点04分05秒
- `value`：要被解析成时间格式的字符串

!!! example

    ```go
    // string -> time
    s3 := "2021/2/8"
    t3, err := time.Parse("2006/1/2", s3)
    if err != nil {
        fmt.Println(err) // (1)
    }

    fmt.Println(t3)         // (2)
    fmt.Printf("%T \n", t3) // (3)
    ```

    1. parsing time "2021/2/8" as "2006-1-2": cannot parse "/2/8" as "-"
    2. 2021-02-08 00:00:00 +0000 UTC
    3. time.Time
### 解析具体时间 `#!go t.Func...()`
方法签名：
```go
func (t Time) Date() (year int, month Month, day int) {}
func (t Time) Clock() (hour, min, sec int) {}
func (t Time) Year() int {}
func (t Time) Month() Month {}
func (t Time) Day() int {}
func (t Time) Weekday() Weekday {}
func (t Time) Hour() int {}
func (t Time) Minute() int {}
func (t Time) Second() int {}
func (t Time) Nanosecond() int {}
func (t Time) YearDay() int {}
func (t Time) ISOWeek() (year, week int) {}
```

- `Date()` 返回年月日
- `Clock()` 返回时分秒
- `WeekDay()` 返回星期
- `YearDay()` 返回该日期是那一年的第几天
- `ISOWeek()` 返回该日期是那一年的第几周

!!! example

    ```go
    t2 := time.Date(2021, 10, 1, 9, 10, 32, 11, time.Local)
    fmt.Println(t2) // (1)

    year, month, day := t2.Date()
    fmt.Println(year, month, day) // 2021 October 1

    hour, min, sec := t2.Clock()
    fmt.Println(hour, min, sec)  // 9 10 32

    fmt.Println(t2.Year())       // 2021
    fmt.Println(t2.Month())      // October
    fmt.Println(t2.Day())        // 1
    fmt.Println(t2.Weekday())    // Friday
    fmt.Println(t2.Hour())       // 9
    fmt.Println(t2.Minute())     // 10
    fmt.Println(t2.Second())     // 32
    fmt.Println(t2.Nanosecond()) // 11
    fmt.Println(t2.YearDay())    // 274, 2021-10-1是2021年的第274天
    fmt.Println(t2.ISOWeek())    // 2021 39
    ```

    1. 2021-10-01 09:10:32.000000011 +0800 CST


## 时间计算
### 时间修改 `#!go t.Add()、t.Sub()`
方法签名：
```go
func (t Time) Add(d Duration) Time {}
func (t Time) Sub(u Time) Duration {}
```

- `Add` 用来获得一个增减后的时间，`return t + d`
- `Sub` 用来获得两个时间的差，`return u - t`

!!!example

    ```go
    /* 时间间隔 */
    fmt.Println(t2)                     // 2021-10-01 09:10:32.000000011 +0800 CST
    // 加一分钟
    fmt.Println(t2.Add(time.Minute))    // 2021-10-01 09:11:32.000000011 +0800 CST
    // 减一分钟
    fmt.Println(t2.Add(-time.Minute))   // 2021-10-01 09:09:32.000000011 +0800 CST
    // 加1天
    fmt.Println(t2.Add(24 * time.Hour)) // 2021-10-02 09:10:32.000000011 +0800 CST
    // 加1年1个月零3天
    fmt.Println(t2.AddDate(1, 1, 3))    // 2022-11-04 09:10:32.000000011 +0800 CST

    fmt.Println(t2)            // 2021-10-01 09:10:32.000000011 +0800 CST
    t5 := t2.Add(time.Minute)
    fmt.Println(t5)            // 2021-10-01 09:11:32.000000011 +0800 CST

    fmt.Println(t5.Sub(t2))    // 1m0s, t5 - t2 = 1分钟0秒
    ```

### 时间判断 `#!go t.Equal()、t.Before()、t.After()`
```go
// 判断两个时间是否相同，会考虑时区的影响，因此不同时区标准的时间也可以正确比较。本方法和用t==u不同，这种方法还会比较地点和时区信息。
func (t Time) Equal(u Time) bool {}

// 如果 t 代表的时间点在 u 之前，返回真；否则返回假。
func (t Time) Before(u Time) bool {}

// 如果 t 代表的时间点在 u 之后，返回真；否则返回假。
func (t Time) After(u Time) bool {}
```

!!! example

    ```go
    t1 := time.Date(2021, 2, 13, 17, 14, 05, 132465461, time.Local)
    t2 := time.Date(2021, 2, 14, 17, 14, 05, 132465461, time.Local)
    t3 := time.Date(2021, 2, 15, 17, 14, 05, 132465461, time.Local)

    fmt.Println(t1.Equal(t2))  // false
    fmt.Println(t1.Before(t2)) // true
    fmt.Println(t1.Before(t3)) // true
    fmt.Println(t2.After(t1))  // true
    fmt.Println(t2.After(t3))  // false
    ```

## 时间控制

### 睡眠 `#!go time.Sleep()`
函数签名：
```go
func Sleep(d Duration)
```

!!! example

    ```go
    // 睡眠3秒钟
    time.Sleep(3 * time.Second)
    ```

### 定时器 time.Tick()、time.NewTimer()

- `Ticker` 是按一定时间间隔 **持续** 触发时间事件。
- `Timer` 是 **一次性** 的时间触发事件

#### 持续性——Ticker
```go
func Tick(d Duration) <-chan Time
```
`Tick` 返回的是一个只读通道。

!!! example

    ```go hl_lines="1"
    ticker := time.Tick(time.Second) //定义一个1秒间隔的定时器
    for t := range ticker {
        fmt.Println(t) //每秒都会执行的任务
    }
    ```

    或者

    ```go hl_lines="1"
    ticker := time.Tick(time.Second) //定义一个1秒间隔的定时器
    for {
        fmt.Println(<-ticker) //每秒都会执行的任务
    }
    ```

#### 一次性——Timer
标准库中的 Timer 让用户可以自定义自己的超时逻辑，尤其是在应对 select 处理多个 channel 的超时、单 channel 读写的超时等情形时尤为方便。

常见的创建方式

```go
t := time.NewTimer(d)
t := time.AfterFunc(d, f)
c := time.After(d)
```

虽然创建方式不同，但原理是相同的。

Timer 有3个要素：

- 定时时间： `d`
- 触发动作：`f`
- 时间chan：`c`

下面逐一说明：

- **`NewTimer()`**
    ```go
    func NewTimer(d Duration) *Timer
    ```
    
    !!! example

        ```go hl_lines="2 6"
        func main() {
            timer := time.NewTimer(3 * time.Second)    // 创建一个 timer
            fmt.Println(time.Now()) // (1)
        
            // 启动定时器，阻塞3秒后继续执行
            fmt.Println(<-timer.C)  // (2)
        }
        ```

        1. 2021-02-14 17:58:28.7661805 +0800 CST m=+0.003483101
        2. 2021-02-14 17:58:31.7657332 +0800 CST m=+3.003035801

- **`After()`**
    ```go
    func After(d Duration) <-chan Time {
        return NewTimer(d).C
    }
    ```
    `After()` 其实就是返回 `timer.C`

    !!! example

        ```go hl_lines="2 6"
        func main() {
            ch := time.After(3 * time.Second)   // (1)
        
            fmt.Println(time.Now()) // (2)
            // 启动定时器
            fmt.Println(<-ch) // (3)
        }
        ```

        1. 返回一个通道，存储的是d时间间隔之后的当前时间
        2. 2021-02-14 18:21:23.3943073 +0800 CST m=+0.003990801
        3. 2021-02-14 18:21:26.3929503 +0800 CST m=+3.002633801

- **`AfterFunc()`**
    ```go
    func AfterFunc(d Duration, f func()) *Timer
    ```
    `AfterFunc()` 会在 `d` 时间后，启动一个 `goroutine` 执行 `f`

    !!! example
    
        ```go hl_lines="8"
        func main() {
            fn := func() {
                defer wg.Done()
                fmt.Println("do")
            }
        
            wg.Add(1)
            time.AfterFunc(3*time.Second, fn)
            wg.Wait()
        }
        ```


!!! example "在另外一个`goroutine` 启动定时器"

    ```go
    func main() {
        timer := time.NewTimer(5 * time.Second)    // 创建一个 timer

        wg.Add(1)
        go func() {
            defer wg.Done()
            <-timer.C    // (1)
            fmt.Println("Timer2 is over.")
        }()

        time.Sleep(3 * time.Second) // (2)
        flag := timer.Stop()    // (3)
        if flag {
            fmt.Println("Timer2 停止了")
            wg.Done()
        }
        wg.Wait()
    }
    ```

    1. 启动定时器，定时器结束后（阻塞等待时间结束）打印
    2. 3秒后停止 timer
    3. 提前停止定时器