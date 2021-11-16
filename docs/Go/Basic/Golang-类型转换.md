---
title: 'Golang [nil] 类型转换'
seotitle: 'Golang [nil] 类型转换'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: 2c39f343
date: 2021-07-19 23:40:51
updated: 2021-07-19 23:40:51
---

类型转换的三种方式

<!--more-->

# 类型转换

**Go 不会对数据进行隐式转换，只能显式的手动转换**

## 简单的转换

```go
T(expression)
```

eg：
```go
int(time.Now().Weekday())    // 星期转int
int(time.now().Month())      // 月份转int

var a float32
a = 3.14
b := int(a)     // float32 转 int
```
注意：这种方式不是所有数据类型都能转换的。
1. 例如 string 类型的 `"Boii"` 转 int 就会失败
2. 低精度转高精度时是安全的，高精度转低精度则会丢失精度，例如 float64转float32
3. 不能对 int、float 同 string 互转，跨大类型转换，可以使用 `strconv 包` 提供的函数

## strconv 包
strconv 包提供了基本数据类型之间的类型转换功能。

### string -> int : Atoi()

将 string 转换为 int：Atoi()

Atoi() 函数签名：
```go
func Atoi(s string) (int, error)
```
由于 string 有可能无法转换为 int，所以提供了两个返回值
- 第一个是转换成 int 的值
- 第二个是返回是否转换成功

eg：
```go
// success
i, err := strconv.Atoi("3")
fmt.Print(i + 5)    // 8

// fail
i, err := strconv.Atoi("a")
if err != nil {
    fmt.Print("Converted FAILED!")
}
```

### int -> string : Itoa()

```go
func Itoa(i int) string
```

eg：
```go
a := strconv.Itoa(18)
fmt.Println(a)         // 18
fmt.Printf("%T \n", a) // string
```
### string -> T : ParseT()

Parse 类函数用于将字符转转换为给定类型的值

#### ParseBool(str string)
eg：
```go
b, err := strconv.ParseBool("true")
if err == nil {
	fmt.Println(b) // true
}
```
ParseBool 的内部实现非常简单

```go
func ParseBool(str string) (bool, error) {
	switch str {
	case "1", "t", "T", "true", "TRUE", "True":
		return true, nil
	case "0", "f", "F", "false", "FALSE", "False":
		return false, nil
	}
	return false, syntaxError("ParseBool", str)
}
```

#### ParseInt(s string, base int, bitSize int)
- s：要解析 int 的字符串
- base：指定字符串中数字的进制（2到36），如果为0，会从字符串前置判断：
    - `0b`代表2进制
    - `0o`代表8进制
    - `0x`代表16进制
    - 否则是10进制。
- bitSize：指定结果必须能无溢出赋值的整数类型，0、8、16、32、64 分别代表 int、int8、int16、int32、int64

eg：
```go
i, err := strconv.ParseInt("-C", 16, 0)
if err == nil {
	fmt.Println(i)         // -12
	fmt.Printf("%T \n", i) // int64
}
```
十六进制中，C代表12，例子中传递的是 -C ，也就是-12


#### ParseUint(s string, base int, bitSize int)
和 Parseuint 一样，不过是针对 无符号整型的，如果传递一个有符号的数字字符串，会转换失败

eg：
```go
// success
u, err := strconv.ParseUint("45", 7, 0)
if err == nil {
	fmt.Println(u)         // 33
	fmt.Printf("%T \n", u) // uint64
}

// fail
u, err := strconv.ParseUint("-10", 10, 0)
if err != nil {
    fmt.Print("Converted FAILED!")
}
```
7进制的 45 等于 10进制的33


#### ParseFloat(s string, bitSize int)
- s：要解析成 float 的字符串，
- bitSize：指定了期望的接收类型，32是float32（返回值可以不改变精确值的赋值给float32），64是float64。

eg：
```go
f, err := strconv.ParseFloat("3.14", 64)
if err == nil {
	fmt.Println(f)         // 3.14
	fmt.Printf("%T \n", f) // float64
}
```
bitSize 表示位数，只能填 64或32，不过 ParseFloat 只能接收64位类型的浮点数，即使 bitSize 你填32，依然会返回一个 float64 类型的数据给你，因为其内部实现是调用了一个 `atof32()` 的函数，然后 `return float64(f)`

```go
func ParseFloat(s string, bitSize int) (float64, error) {
	f, n, err := parseFloatPrefix(s, bitSize)
	if err == nil && n != len(s) {
		return 0, syntaxError(fnParseFloat, s)
	}
	return f, err
}

func parseFloatPrefix(s string, bitSize int) (float64, int, error) {
	if bitSize == 32 {
		f, n, err := atof32(s)
		return float64(f), n, err
	}
	return atof64(s)
}
```

#### ParseComplex(s string, bitSize int)
- s：要解析成 float 的字符串，
- bitSize：指定了期望的接收类型，64是complex64（返回值可以不改变精确值的赋值给float32），128是complex128。

eg：
```go
c, err := strconv.ParseComplex("3+4i", 128)
if err == nil {
	fmt.Println(c)         // (3+4i)
	fmt.Printf("%T \n", c) // complex128
}
```

### T -> string : FormatT()

FormatT 系列函数用于将 T 类型数据转成字符串

#### FormatBool
函数签名：
```go
FormatBool(b bool) string
```

eg：
```go
b := strconv.FormatBool(true)
fmt.Println(b) // true
```

#### FormatInt
函数签名：
```go
FormatInt(i int64, base int) string
```
- i：要转成字符串的数字，必须是 int64 类型
- base：要转成什么进制再转字符串

eg：
```go
i1 := strconv.FormatInt(31, 16)
fmt.Println(i1) // 1f ,将 10进制的31 转成 16进制的1f 然后转成字符串返回

i2 := strconv.FormatInt(-0x1E, 8)
fmt.Println(i2) // -36, 将 16进制的-1E 转成 8进制的-36 然后转成字符串返回
```

#### FormatUint
函数签名：
```go
FormatUint(i uint64, base int) string
```
- i：要转成字符串的数字，必须是 uint64 类型
- base：要转成什么进制再转字符串

eg：
```go
u := strconv.FormatUint(12, 8)
fmt.Println(u) // 14, 将 10进制的12 转成 8进制的14 然后转成字符串返回
```

#### FormatFloat
函数签名：
```go
FormatFloat(f float64, fmt byte, prec, bitSize int) string
```
- f：要转成字符串的浮点数
- fmt：['f', 'b', 'e', 'E', 'g', 'G']；表示格式
    - f：000.000
    - b：000p±000 二进制指数
    - e：0.0000e±00 十进制指数
    - E：0.0000E±00 十进制指数
    - g：指数很大时用e格式，不大时用f格式
    - G：指数很大时用E格式，不大时用f格式
- prec：
    - 对于 f格式、e格式、E格式：控制小数点后位数
    - 对于 g格式、G格式：控制中数字个数
    - 为-1时，代表使用最少数量的、但又必需的数字来表示f
- bitSize：表示浮点数来源类型（32：float32、64：float64）

eg：
```go
v := 4.12345678

ff64 := strconv.FormatFloat(v, 'f', 30, 64)
ff32 := strconv.FormatFloat(v, 'f', 30, 32)
fb := strconv.FormatFloat(v, 'b', 10, 32)
fe := strconv.FormatFloat(v, 'e', -1, 32)
fe10 := strconv.FormatFloat(v, 'e', 10, 32)
fg := strconv.FormatFloat(v, 'g', 10, 32)

fmt.Println(ff64) // 4.123456779999999710639713157434
fmt.Println(ff32) // 4.123456954956054687500000000000
fmt.Println(fb)   // 8647516p-21
fmt.Println(fe)   // 4.123457e+00
fmt.Println(fe10) // 4.1234569550e+00
fmt.Println(fg)   // 4.123456955
```

#### FormatComplex
函数签名：
```go
FormatComplex(c complex128, fmt byte, prec, bitSize int) string
```
- f：要转成字符串的复数
- fmt：['f', 'b', 'e', 'E', 'g', 'G']；表示格式
    - f：000.000
    - b：000p±000 二进制指数
    - e：0.0000e±00 十进制指数
    - E：0.0000E±00 十进制指数
    - g：指数很大时用e格式，不大时用f格式
    - G：指数很大时用E格式，不大时用f格式
- prec：
    - 对于 f格式、e格式、E格式：控制小数点后位数
    - 对于 g格式、G格式：控制中数字个数
    - 为-1时，代表使用最少数量的、但又必需的数字来表示f
- bitSize：表示复数来源类型（64：complex64、128：complex128）

eg：
```go
c := strconv.FormatComplex(3+8i, 'e', 10, 64)
fmt.Println(c)	// (3.0000000000e+00+8.0000000000e+00i)
```

### T -> string & append to slice : AppendT()

AppendT 系列函数用于将 T 转成字符串后 append 到一个 切片 slice 中

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // 声明一个slice
    b10 := []byte("int (base 10):")

    // 将转换为10进制的string，追加到slice中
    b10 = strconv.AppendInt(b10, -42, 10)
    fmt.Println(string(b10))    // int (base 10):-42

    b16 := []byte("int (base 16):")
    b16 = strconv.AppendInt(b16, -42, 16)
    fmt.Println(string(b16))    // int (base 16):-2a
}
```

## 类型断言
类型断言分两种形式：`Type Assertion` 和 `Type Switch`。

### Type Assertion
```go
expression.(Type)
```
eg：
```go
t := i.(T)
t, ok := i.(T)
```

Type Assertion 的作用有两个：

1. 检查 `i` 是否为 nil
2. 检查 `i` 的值是否为 T 类型

使用方式有两种:

1. `t := i.(T)`：
    - **说明**：可以断言一个接口对象 `i` 的值不是 *nil*，并且值是 T 类型。
    - **断言成功**：返回 `i` 的值给 `t`
    - **断言失败**：引发 `panic`
2. `t, ok := i.(T)`
    - **说明**：可以断言一个接口对象 `i` 的值不是 *nil*，并且值是 T 类型。
    - **断言成功**：返回 `i` 的值给 `t`，`ok` 的值置为 *true*
    - **断言失败**：返回 `T` 的零值给 `t`，`ok` 的值置为 *false*


eg：

1. `t := i.(T)`
    ```go
    func main() {
    	var i interface{} = 10
    	t1 := i.(int)
    	fmt.Println(t1)    // 10
    
    	fmt.Println("+++++++++++++++++++++")
    
    	t2 := i.(string)    // 这里会引发 panic
    	fmt.Println(t2)
    
    }
    // --------------------------------------
    // Output:
    10
    +++++++++++++++++++++
    panic: interface conversion: interface {} is int, not string
    ```

    接口值为 nil 时，断言会引发 panic
    ```go
    func main() {
    	var i interface{}    // i 的值为 nil
    	t := i.(interface{}) // 这里会引发 panic
    	fmt.Println(t)
    }
    // --------------------------------------
    // Output:
    panic: interface conversion: interface is nil, not interface {}
    ```
2. `t, ok := i.(T)`
    ```go
    func main() {
    	var i1 interface{} = 10
    
    	t1, ok := i1.(interface{})
    	fmt.Printf("t1: %d, %v\n", t1, ok) // t1: 10, true
    	t2, ok := i1.(int)
    	fmt.Printf("t2: %v, %v\n", t2, ok) // t2: 10, true
    	t3, ok := i1.(string)
    	fmt.Printf("t3: %v, %v\n", t3, ok) // t3: , false
    
    	var i2 interface{}
    	t4, ok := i2.(interface{})
    	fmt.Printf("t4: %v, %v\n", t4, ok) // t4: <nil>, false
    
    	i2 = "Boii"
    	t5, ok := i2.(int)
    	fmt.Printf("t5: %v, %v\n", t5, ok) // t5: 0, false
    	t6, ok := i2.(string)
    	fmt.Printf("t6: %v, %v\n", t6, ok) // t6: Boii, true
    }
    // --------------------------------------
    // Output:
    t1: 10, true
    t2: 10, true
    t3: , false
    t4: <nil>, false
    t5: 0, false
    t6: Boii, true
    ```
    虽然 `t3、t4、t5` 断言失败，但没有引发 `panic`


### Type Switch

上面的方式适合 断言指定一种类型，如果需要断言 接口对象 `i` 是多种类型中的一种，则需要用 Type Switch。

```go
switch t := 接口对象.(type) {
    case T1:  ...
    case T2:  ...
    case nil: ...
    default: ...
}
```
`接口对象.(type)` 中的 `.(type)` 是固定格式，不要修改。

```go
func typeSwitch(i interface{}) {
    switch t := i.(type) {
    case int:
        fmt.Println(t, "is int.")
    case string:
        fmt.Println(t, "is string.")
    case float64:
        fmt.Println(t, "is float64")
    case nil:
        fmt.Println(t, "is nil")
    default:
        fmt.Println(t, "啥也不是")
    }
}


func main() {
	var i interface{}
	typeSwitch(i)      // <nil> is nil
	typeSwitch("Boii") // Boii is string.
	typeSwitch(3.14)   // 3.14 is float64
	typeSwitch(true)   // true 啥也不是
}
```
`t` 是什么类型就走什么分支，`t` 是 nil 就走 `case nil` 分支，没有一个满足就走 `default` 分支。
`default` 分支是可选的。
