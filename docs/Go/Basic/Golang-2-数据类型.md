# 2-数据类型

## 整型
整型分为两大类：`有符号`和`无符号`

按长度分为：`int8`、`int16`、`int32`、`int64`；`uint8`、`uint16`、`uint32`、`uint64`

| 类型   | 范围                                      | 描述          |     对应C语言      |
| :----- | :--------------------------------------- | :----------- | -------------- |
| int8   | -128~127                                 | 有符号8位整型  | byte           |
| int16  | -32768~32767                             | 有符号16位整型 | short          |
| int32  | -2147483648~2147483647                   | 有符号32位整型 | int            |
| int64  | -9223372036854775808~9223372036854775807 | 有符号64位整型 | long           |
| uint8  | 0~255                                    | 无符号8位整型  | unsigned byte  |
| uint16 | 0·65535                                  | 无符号16位整型 | unsigned short |
| uint32 | 0~4294967295                             | 无符号32位整型 | unsigned int   |
| uint64 | 0~18446744073709551615                   | 无符号64位整型 | unsigned long  |

### 特殊整型
| 类型    | 描述                                  |
| :------ | :----------------------------------- |
| uint    | 32位OS上就是uint32，64位OS上就是uint64 |
| int     | 32位OS上就是int32，64位OS上就是int64   |
| uintptr | 无符号整型，用于存放一个指针            |


!!! warning "注意"
    在使用 `int` 和 `uint` 类型时，不能假定它是32位或64位的整型，而要考虑`int`和`uint`可能在不同平台上的差异。
    在涉及到 **二进制传输、读写文件的结构描述** 时，为了保持文件的结构不会受到不同编译目标平台字节长度的影响，不要使用`int`和`uint`。



### 数字字面量语法

字面量语法使得开发者可以用 二进制、八进制、十六进制的格式定义数字

- `#!go v := 0b101110`，**0b前缀**；代表二进制的 101110，相当于八进制的56，十进制的46，十六进制的 2E
- `#!go v := 0o56`，**0o前缀**；代表八进制的 56
- `#!go v := 46`，**无前缀**；代表十进制的 46
- `#!go v := 0x2e`，**0x前缀**；代表十六进制的 2E
- `#!go v := 0x1p-2`，代表十六进制的 1 除以 $2^2$，也就是 0.25

还可以用 `_` 来分隔数字。比如 `#!go v := 1_000_000` 表示 v 的值等于 一百万 1000000。

借助 `#!go Printf()` 可以将一个整数以不同进制形式展示
```go
func main() {
    num := 0b101110
    fmt.Printf("%b \t %#b \n", num, num)
    fmt.Printf("%o \t\t %#o \n", num, num)
    fmt.Printf("%d \t\t %#d \n", num, num)
    fmt.Printf("%x \t\t %#x \t %#X \n", num, num, num)
}

// ----------------------------------------
// Output:
101110 	 0b101110
56 		 056
46 		 46
2e 		 0x2e 	 0X2E
```

!!! tip 
    整型支持算术运算和位操作，算术表达式和位操作表达式的结果还是整型。 `#!go var a int = 1000 >> 2`

## 浮点型

Go 语言支持两种浮点型数：`float32`和`float64`。
这两种浮点型数据格式遵循 `IEEE754`标准：

- `float32`的浮点数最大范围约为`3.4e38`，可以使用常量定义 `math.MaxFloat32`
- `float64`的浮点数最大范围约为`1.8e308`，可以使用常量定义 `math.MaxFloat64`

打印浮点数时，可以使用占位符 `%f`:
```go
package main
import "math"
import "fmt"

func main() {
    fmt.Printf("%f \n", math.Pi)
    fmt.Printf("%.2f \n", math.Pi)
}

// ----------------------------------------
// Output:
3.141593
3.14
```

??? tip "提示"
    1. 浮点型字面量会被自动类型推断为 float64 类型
        ```go
        b := 10.00
        fmt.Printf("%T", b)    // float64
        ```
    2. 计算机很难进行浮点数的精确表示和存储，因此两个浮点数之间不应该使用 `==` 或 `!=` 来比较，高精度科学计算应该使用 math 标准库

## 复数

`complex64` 和 `complex128`

```go
var c1 complex64 = 1 + 2i
var c2 complex128 = 2 + 3i

fmt.Println(c1)
fmt.Println(c2)

// ----------------------------------------
// Output:
(1+2i)
(2+3i)
```
复数有实部和虚部
complex64的实部和虚部为32位
complex128的实部和虚部为64位。

!!! tip
    Go 有三个内置函数处理复数：complex、real、imag

    ```go
    var v = complex(2.1, 3) // 构造一个复数 (2.1+3i)
    a := real(v)            // 返回复数实部 2.1
    b := imag(v)            // 返回复数虚部 3
    ```

## 布尔值

Go 语言中以 `bool` 类型进行声明布尔型，布尔型数据只有 `true` 和 `false` 两个值。

1. 布尔类型变量默认值为 false
2. Go 语言中 **不允许将整型强制转换为布尔型（integer !-> bool）**
3. 布尔型无法参与数值运算，无法与其他类型进行转换

## 字符串
**Go 语言中的字符串是原生数据类型**，使用字符串就像使用其他原生数据类型（int、bool、float32等）一样
Go 语言中字符串内部实现使用 `UTF-8` 编码，这使得 go 不需要专门使用 UTF-8 字符集的文本进行编码和解码，可以在Go语言中直接添加非ASCII码字符

字符串是一种值类型，且值不可变；即创建某个文本后你无法再次修改这个文本的内容，即使你改变了，你会发现那已经不是原来的那串字符串了。
更深入的讲，字符串是 **字节的定长数组**。

1. Go 支持以下2种形式的字面值

    - 解释字符串

        该类字符串使用双括号 `""` 引起来，其中转义字符（`\n、\r、 \t、 \\`）会被替换。eg：
        
        ```go
        str1 := "Hello"
        str2 := "Boii"
        ```

    - 非解释字符串

        该类字符串使用反引号括起来  **\`\`**，支持换行。eg：
        
        ```go
        // 下面的 \n 会被原样输出
        str := `This is a
        raw string \n`
        ```

2. **Go 中的字符串是根据长度限定，而非特殊字符 `\0`**。
3. string 类型的零值：长度为 0 的字符串，即空字符串 `""`
4. 可以通过 `len()` 函数来获取字符串字符个数
5. 字符串中的内容（纯字节）可以通过标准索引法来获取
    ```go
    func main() {
        str := "hello Boii"
        c1 := str[0]
        c4 := str[4]
        cEnd := str[len(str)-1]
        fmt.Println(c1)        // 104
        fmt.Println(c4)        // 111
        fmt.Println(cEnd)      // 105
    }
    ```
    输出的是字符的 Ascii 码

6. 字符串类型底层实现是一个二元的数据结构，一个是指向字节数组的起点，另一个是长度。
    eg：
    
    ```go
    // runtime/string.go
    type stringStruct struct {
        str unsafe.Pointer    // 指向底层字节数组的指针
        len int               // 字节数组长度
    }
    ```

7. 基于字符串创建的切片 和 原字符串 指向相同的 底层字符数组，一样不能修改，**对字符串的切片操作返回的仍然是 string，而非 slice**。
    ```go
    a := "Hello World"
    b := a[0:4]
    c := a[1:]
    fmt.Printf("b -> %s, b -> type: %T\n", b, b) // b -> Hell, b -> type: string
    fmt.Printf("c -> %s, c -> type: %T\n", c, c) // c -> ello World, c -> type: string
    ```




### 字符串常用操作

| 方法                                  | 描述                                        | 返回值   |
|---------------------------------------|-------------------------------------------|----------|
| len(str)                              | 求字符串长度                                | int      |
| + 或 fmt.Sprintf("%s %s", str1, str2) | 拼接字符串                                  | string   |
| strings.Split(str, 分隔符)            | 分割字符串                                  | []string |
| strings.Contains(str, 目标内容)       | 判断str是否包含目标子串                     | bool     |
| strings.HasPrefix(str, 目标内容)      | 判断str是否以目标内容开头                   | bool     |
| strings.HasSuffix(str, 目标内容)      | 判断str是否以目标内容结尾                   | bool     |
| strings.Index(str, 目标内容)          | 返回str第一次出现目标内容的首个字符的下标   | int      |
| strings.LastIndex(str, 目标内容)      | 返回str最后一次出现目标内容的首个字符的下标 | int      |
| strings.Join(strArr, 拼接符号)        | 返回用拼接符号拼接strArr的字符串            | string   |

```go
package main

import (
    "fmt"
    "strings"
)

func main() {

    // 求字符串长度
    str1 := `hello Boii`
    fmt.Println(len(str1))    // 10
    str2 := "Hello 中国"
    fmt.Println(len(str2))    // 12

    fmt.Println()

    // 拼接字符串
    fmt.Println(str1 + str2)  // hello BoiiHello 中国
    str3 := fmt.Sprintf("%s | %s", str1, str2)
    fmt.Println(str3)         // hello Boii | Hello 中国

    fmt.Println()

    // 字符串分割
    str4 := `How do you do`
    fmt.Println(strings.Split(str4, " "))         // [How do you do]
    fmt.Printf("%T \n", strings.Split(str4, " ")) // []string

    fmt.Println()

    // 判断是否包含
    fmt.Println(strings.Contains(str4, "do"))     // true
    // 判断前缀
    fmt.Println(strings.HasPrefix(str4, "How"))   // true
    // 判断后缀
    fmt.Println(strings.HasSuffix(str4, "How"))   // false

    fmt.Println()

    // 判断子串位置
    fmt.Println(strings.Index(str4, "do"))        // 4
    // 最后子串出现的位置
    fmt.Println(strings.LastIndex(str4, "do"))    // 11

    fmt.Println()

    // join
    str5 := []string{"how", "do", "you", "do"}
    fmt.Println(str5)                            // [how do you do]
    fmt.Println(strings.Join(str5, "-"))         // how-do-you-do

}
```

## 字符
Go 的字符有以下两种：

1. uint8类型，别名 byte，代表一个 ASCII 码字符
2. int32类型，别名 rune，代表一个 UTF-8 字符

**注意：字符用单引号( ' ' )括起来**

```go
func main() {
    var c1 byte = 'a'
    var c2 rune = 'a'
    fmt.Println(c1, c2)
    fmt.Printf("c1 type: %T, c2 type: %T \n", c1, c2)
}

// ----------------------------------------
// Output:
97 97
c1 type: uint8, c2 type: int32
```
在编写代码的时候，使用别名 byte、rune 可以使得代码更加清晰，不过这两个别名在经过编译后还是会还原成 unit8 和 int32。

当需要处理中文、日文或者中英混杂的字符时，则需要用到 rune 类型，因为 byte 型只有8位，只能显示 ASCII 码，而 rune 型有32位。

```go
func main() {
    s := `Hello 中国`

    for i := 0; i < len(s); i++ {
        fmt.Printf("%v[%c]\t", s[i], s[i])
    }

    fmt.Println()

    for _, char := range s {
        fmt.Printf("%v[%c]\t", char, char)
    }

    fmt.Println()
}

// ----------------------------------------
// Output:
72[H]	101[e]	108[l]	108[l]	111[o]	32[ ]	228[ä]	184[¸]	173[­]	229[å]	155[]	189[½]	
72[H]	101[e]	108[l]	108[l]	111[o]	32[ ]	20013[中]	22269[国]
```
遍历一个中英混杂的字符串后我们会发现，中文一个字占好几个字节。
因为 go 中字符串是按照 UTF-8 编码的，而 UTF-8 编码下一个中文汉字由 3~4 个字节组成。
**len(str) 返回的是字符串的字节数，而不是字符串的个数**，所以第一个循环中会出现乱码的情况。

第二种方式 `for...range` 则可以正确的返回字符串中每个字符，因为它是按 rune 类型计算的，所以打印的时候没有乱码。

!!! summary "总结"
    - 字符串的底层是一个 byte 数组，所以字符串可以和 `[]byte` 型互相转换。
    - 字符串是不能修改的
    - 字符串是由 byte 字节组成的
    - 字符串的长度是 byte 字节的长度
    - rune 类型用来表示 utf8 字符
    - 一个字符由一个或多个 byte 组成

## 修改字符串

1. 将字符串转换成 `[]byte` 或 `[]rune` 类型；例如`[]byte(str)` 或 `[]rune(str)`
2. 修改字符串
3. 转回 `string`

!!! warning "注意"
    无论哪种转换，都会重新分配内存，并复制字节数组。

```go
func main() {
    str1 := "big"

    byteS = []byte(str1)    // 转成[]byte切片
    byteS[0] = 'p'          // 修改
    str1 = string(byteS)    // 转回 string
    fmt.Println(str1)

    str2 = "青菜"

    runeS = []rune(str2)    // 转成[]rune切片
    runeS[0] = '白'         // 修改
    str2 = string(runeS)    // 转回 string
    fme.Println(str2)
}

// ----------------------------------------
// Output:
pig
白菜
```



## 类型转换
Go语言中只有强制类型转换，没有隐式类型转换。该语法只能在两个类型之间支持相互转换的时候使用

基本语法如下：
```go
Type(expression)
```
Type 表示要转的类型，expression 表示变量、复杂算式、函数返回值等

```go
var i int = 10
var f float32 = float32(i)
```

详见[类型转换](/wiki/Go/Basic/Golang-类型转换.md)

## 类型大小

!!! tip 
    使用 `unsafe.Sizeof(v)` 可以得到 v 占用内存大小。

|       类型 |            bit数            |
| ---------: | :------------------------: |
|       指针 | 32位 -> 4bit，64位C -> 8bit |
|    uintptr | 32位 -> 4bit，64位C -> 8bit |
|       bool |             8              |
|       byte |             8              |
|       int8 |             8              |
|      uint8 |             8              |
|      int16 |             16             |
|     uint16 |             16             |
|      int32 |             32             |
|       rune |             32             |
|     uint32 |             32             |
|    float32 |             32             |
|      int64 |             64             |
|     uint64 |             64             |
|    float64 |             64             |
|  complex64 |             64             |
| complex128 |            128             |
|        int |        与CPU位数相同        |
|       uint |        与CPU位数相同        |
|     string |  ASCII范围8bit，中文24bit   |



## 类型别名、自定义类型
### 类型别名

为类型起个别名，方便代码编写过程中使用，

```go
type alias = T
```
`type`：关键字、`alias`：类型别名、`T`：类型

例如数据类型中提到的Unicode字符型 `rune` 和ASCII字符型 `byte`就是类型别名
```go
type rune = int32
type byte = uint8
```


### 自定义类型
在 Go 中有一些基本的数据类型，如`string`、`int`、`bool`等数据类型，也可以通过关键字 `type` 来定义自定义类型

自定义类型是定义了一个全新的类型，我们可以基于内置基本类型定义，也可以通过 struct 定义。
eg：
```go
type Status bool
```
通过 `type` 关键字的定义，`Status`就是一种新的类型，它具有 `bool` 的特性


### 区别
从定义上看，类型别名有 `=` ，自定义类型没有。

```go
func main() {
    type abc = string
    type Status bool

    var OK Status
    var ss abc

    fmt.Printf("Type of OK: %T\n", OK)  // Type of OK: main.Status
    fmt.Printf("Type of ss: %T\n", ss)  // Type of ss: string
}
```
OK 的类型是 main.Status，表示在 main 包下定义的`Status`类型
ss 的类型是 string。`abc`类型只会在代码中存在，编译前编译器会将其替换回来。
