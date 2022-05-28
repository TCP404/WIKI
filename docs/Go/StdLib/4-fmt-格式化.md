# 4-fmt

fmt 应该是初学者接触的第一个包了，第一个 Golang 程序 Hello World 也需要借助 fmt 包才能打印出来。

## 导包

```go
import "fmt"
```

## 向外输出

### Print 系列

Print系列函数包括 `fmt.Print()`、`fmt.Printf()`、`fmt.Println()`

- `Print()` 函数直接输出内容
- `Printf()` 函数支持格式化输出字符串
- `Println()` 直接输出内容并会换行，等价 `Print("...\n")`

它们的函数签名如下：

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

!!!example

    ```go
    import "fmt"

    func main() {
        name := "Boii"
        fmt.Print("Hello Boii.")			// Hello Boii.
        fmt.Printf("Hello %s.\n", name)		// Hello Boii.
        fmt.Println("Hello " + name + ".")	// Hello Boii.
    }
    ```

### Fprint 系列

Fprint 系列函数会将内容输出到一个 `io.Writer` 接口类型变量中，通常用这个函数往文件中写入内容。

这个系列也包含三个函数：

- `Fprint()` 函数直接输出内容
- `Fprintf()` 函数支持格式化输出字符串
- `Fprintln()` 直接输出内容并会换行，等价 `Fprint("...\n")`

它们的函数签名如下：

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

!!!example

    ```go
    import (
        "fmt"
        "os"
    )

    func main() {
        // 打开文件
        f, _ := os.OpenFile("./test.txt", os.O_CREATE | os.O_WRONLY | os.O_APPEND, 0644)
        name := "Boii"
        // 向文件写入内容
        fmt.Fprint(f, "Hello Boii.")
        fmt.Fprintf(f, "Hello %s.", boii)
        fmt.Fprintln(f, "Hello " + name + ".")
        
        // 向标准输出写入内容
        fmt.Fprinln(os.Stdout, "Stdout print: Hello Boii.")
    }
    ```



### Sprint 系列

Sprint 系列函数会将传入的数据生成后返回一个字符串

它们同样有三个函数：

- `Sprint()` 函数直接输出内容
- `Sprintf()` 函数支持格式化输出字符串
- `Sprintln()` 直接输出内容并会换行，等价 `Sprint("...\n")`

它们的函数签名如下：

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

!!!example

    ```go
    import "fmt"

    func main() {
        name := "Boii"
        s1 := fmt.Sprint("Hello Boii.")
        s2 := fmt.Sprintf("Hello %s.", name)
        s3 := fmt.Sprintln("Hello " + name + ".")
        fmt.Println(s1, s2, s3)	// Hello Boii.Hello Boii.Hello Boii.
    }
    ```

### Errorf

Errorf 函数根据 format 参数生成格式化字符串并返回一个包含该字符串的错误。

该函数的签名如下：

```go
func Errorf(format string, a ...interface{}) error
```

!!!example

    ```go
    err := fmt.Errorf("这是错误信息。")
    ```





## 接收输入

## Scan 系列

Scan 系列可以在程序运行过程中，从 Stdin 标准输入中获取用户的输入。该系列函数返回成功扫描的数据个数和遇到的任何错误。如果读取的数据个数比提供的参数少，会返回一个错误报告原因。

Scan 系列有三个函数：

- `Scan()` 读取由空白符分隔的值保存到传递给本函数的参数中，换行符视为空白符。
- `Scanf()` 根据format参数指定的格式去读取由空白符分隔的值保存到传递给本函数的参数中。
- `Scanln()` 类似Scan，它在遇到换行时才停止扫描。最后一个数据后面必须有换行或者到达结束位置。

它们的函数签名如下：

```go
func Scan(a ...interface{}) (n int, err error)
func Scanf(format string, a ...interface{}) (n int, err error)
func Scanln(a ...interface{}) (n int, err error)
```

!!! example "Scan"

    ```go
    import "fmt"

    func main() {
        var name string
        var age  int
        
        fmt.Scan(&name, &age)
        fmt.Printf("输入内容 name： %s， age： %d\n", name, age)
    }
    ```

    输入时，通过空格分隔。

    ```bash
    $ ./demo
    Boii 18
    输入内容 name： Boii， age： 18
    ```



!!! example "Scanf"

    ```go
    import "fmt"

    func main() {
        var name string
        var age  int
        
        fmt.Scanf("1: %s; 2: %d", &name, &age)
        fmt.Printf("输入内容 name： %s， age： %d\n", name, age)
    }
    ```

    输入时，需要按照既定格式输入。

    ```bash
    $ ./demo
    1: Boii; 2: 18
    输入内容 name： Boii， age： 18
    ```



!!!example "Scanln"

    ```go
    import "fmt"

    func main() {
        var name string
        var age  int
        
        fmt.Scanln(&name, &age)
        fmt.Printf("输入内容 name： %s， age： %d\n", name, age)
    }
    ```

    输入时，通过空格分隔，遇到回车结束扫描。

    ```bash
    $ ./demo
    Boii 18
    输入内容 name： Boii， age： 18
    ```

## Fscan 系列

Fscan 系列类似于 Fprint 系列，只不过它们不是从标准输入中读取数据，而是从 `io.Reader` 中读取数据。

它们的函数签名如下：

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

!!!example

    ```go
    import (
        "fmt"
        "os"
        "strings"
    )
    
    // Calling main
    func main() {
    
        // Declaring some type of variables
        var (
            i int
            b bool
            s string
            f float32
        )
        
        // Calling the NewReader() function to
        // specify some type of texts.
        // variable "r" contains the scanned texts
        r := strings.NewReader("46 true 3.4 GeeksforGeeks")
        
        // Calling the Fscan() function to receive 
        // the scanned texts
        n, err := fmt.Fscan(r, &i, &b, &f, &s)
        
        // If the above function returns an error then
        // below statement will be executed
        if err != nil {
            fmt.Fprintf(os.Stderr, "Fscanf: %v\n", err)
        }
        
        // Printing each type of scanned texts
        fmt.Println(i, b, f, s)
        
        // It returns the number of items 
        // successfully scanned
        fmt.Println(n)
    }
    ```

    输出：

    ```bash
    $ ./demo
    46 true 3.4 GeeksforGeeks
    4
    ```



## Sscan 系列

Sscan 系列类似于 Sprint 系列，只不过它们不是从标准输入中读取数据，而是从指定字符串中读取数据。

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

!!!example

    ```go
    // Importing fmt
    import (
        "fmt"
    )
    
    // Calling main
    func main() {
    
        // Declaring two variables
        var name string
        var alphabet_count int
    
        // Calling the Sscan() function which
        // returns the number of elements
        // successfully scanned and error if
        // it persists
        n, _ := fmt.Sscan("GFG 3", &name, &alphabet_count)
    
        // Printing the number of elements and each elements also
        fmt.Printf("%d: %s, %d\n", n, name, alphabet_count)
    }
    ```

    输出：

    ```bash
    $ ./demo
    2: GFG, 3
    ```

## 格式化占位符

printf 系列函数都支持 format 格式化参数，但是占位符非常多种，我这里参照 [李文周博客](https://www.liwenzhou.com/posts/Go/go_fmt/#autoid-1-1-4) 给出的方式分类。

### 通用占位符

| 占位符 | 说明                               |
| ------ | ---------------------------------- |
| %v     | 值的默认格式表示                   |
| %+v    | 类似%v，但输出结构体时会添加字段名 |
| %#v    | 值的Go语法表示                     |
| %T     | 打印值的类型                       |
| %%     | 打印百分号                         |

!!!example

    ```go
    import "fmt"

    func main() {
        type Person struct{ name string }
        s := Person{"Boii"}
        fmt.Printf("%v \n", s)  // 默认格式： {Boii}
        fmt.Printf("%+v \n", s) // 添加字段名： {name:Boii}
        fmt.Printf("%#v \n", s) // Go格式： main.Person{name:"Boii"}
        fmt.Printf("%T \n", s)  // 类型： main.Person
        fmt.Printf("%% \n")     // 百分号： %
    }
    ```



### 布尔型

| 占位符 | 说明          |
| ------ | ------------- |
| %t     | 打印 true 或 false |

### 整型

| 占位符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| %b     | 打印值的二进制形式                                           |
| %o     | 打印值的八进制形式                                           |
| %d     | 打印值的十进制形式                                           |
| %x     | 打印值的十六进制形式，使用 a-f                               |
| %X     | 打印值的十六进制形式，使用A-F                                |
| %c     | 打印值的unicode码值                                          |
| %U     | 打印值的Unicode格式                                          |
| %q     | 打印值的对应字面值，用单括号因起来，必要时会采用安全的转义表示 |

!!! example

    ```go
    import (
        "fmt"
    )

    func main() {
        n := 75
        fmt.Printf("%b \n", n) // 1001011
        fmt.Printf("%o \n", n) // 113
        fmt.Printf("%d \n", n) // 75
        fmt.Printf("%x \n", n) // 4b
        fmt.Printf("%X \n", n) // 4B
        fmt.Printf("%c \n", n) // K
        fmt.Printf("%U \n", n) // U+004B
        fmt.Printf("%q \n", n) // 'k'
    }
    ```



### 浮点数与复数

| 占位符 | 说明                                                   |
| ------ | ------------------------------------------------------ |
| %b     | 无校书部分、二进制指数的科学计数法                     |
| %e     | 科学计数法，如-123.456e+78                             |
| %E     | 科学计数法，如-123.456E+78                             |
| %f     | 有小数部分但无指数                                     |
| %F     | 等价%f                                                 |
| %g     | 根据实际情况采用%e或%f格式（以获得更简洁、准确的输出） |
| %G     | 根据实际情况采用%E或%F格式（以获得更简洁、准确的输出） |

!!!example

    ```go
    import "fmt"

    func main() {
        f := 123456789123.456789123456789
        fmt.Printf("%b \n", f) // 8090864131994864p-16
        fmt.Printf("%e \n", f) // 1.234568e+11
        fmt.Printf("%E \n", f) // 1.234568E+11
        fmt.Printf("%f \n", f) // 123456789123.456787
        fmt.Printf("%F \n", f) // 1123456789123.456787
        fmt.Printf("%g \n", f) // 1.2345678912345679e+11
        fmt.Printf("%G \n", f) // 1.2345678912345679E+11
    }
    ```



### 字符串和 []byte

| 占位符 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| %s     | 直接输出字符串或[]byte                                       |
| %q     | 打印值的对应字面值，用双括号因起来，必要时会采用安全的转义表示 |
| %x     | 每个字节用两字符十六进制数表示（使用a-f）                    |
| %X     | 每个字节用两字符十六进制数表示（使用A-F）                    |

!!!example

    ```go
    import "fmt"

    func main() {
        s := "你好 Boii"
        fmt.Printf("%s \n", s) // 你好 Boii
        fmt.Printf("%q \n", s) // "你好 Boii"
        fmt.Printf("%x \n", s) // e4bda0e5a5bd20426f6969
        fmt.Printf("%X \n", s) // E4 BD A0 E5 A5 BD 20 42 6F 69 69
                               //    你   |   好    | | B  o  i  i
    }
    ```



### 指针

| 占位符 | 说明                           |
| ------ | ------------------------------ |
| %p     | 表示为十六进制，并加上前导的0x |

!!!example

    ```go
    import "fmt"

    func main() {
        a := "Boii"
        fmt.Printf("%p \n", &a)  // 0xc00009e220
        fmt.Printf("%#p \n", &a) // c00009e220
    }
    ```



### 宽度

宽度标识符通过一个紧跟在百分号后面的十进制数指定。

- 宽度：即整个值占多少位。如未指定宽度，则表示值时除必须之外不做填充。

- 精度：小数点后的位数，即小数占多少位，点号后的十进制数指定。如未指定精度，则使用默认精度。如点号后没有数字，则精度为0。



| 占位符 | 说明               |
| ------ | ------------------ |
| %f     | 默认宽度，默认精度 |
| %9f    | 宽度9，默认精度    |
| %.2f   | 默认宽度，精度2    |
| %9.2f  | 宽度9，精度2       |
| %9.f   | 宽度9，精度0       |

!!!example

    ```go
    import "fmt"

    func main() {
        f := 123456789123.456789123456789
        fmt.Printf("%f \n", f)    // 123456789123.456787 
        fmt.Printf("%9f \n", f)   // 123456789123.456787
        fmt.Printf("%9.f \n", f)  // 123456789123 
        fmt.Printf("%.2f \n", f)  // 123456789123.46
        fmt.Printf("%9.2f \n", f) // 123456789123.46
        
        f = 75.445	
        fmt.Printf("%f \n", f)    // 75.445000
        fmt.Printf("%9f \n", f)   // 75.445000
        fmt.Printf("%9.f \n", f)  //        75
        fmt.Printf("%.2f \n", f)  // 75.44
        fmt.Printf("%9.2f \n", f) //     75.44
    }
    ```

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/IMG/20210608150153.png)

### 其他

| 符   | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| ‘+’  | 总是输出数值的正负号；对%q（%+q）会生成全部是ASCII字符的输出（通过转义）； |
| ‘ ’  | 对数值，正数前加空格而负数前加负号；对字符串采用%x或%X时（% x或% X）会给各打印的字节之间加空格 |
| ‘-’  | 在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐）； |
| ‘#’  | 八进制数前加0（%#o），十六进制数前加0x（%#x）或0X（%#X），指针去掉前面的0x（%#p）对%q（%#q），对%U（%#U）会输出空格和单引号括起来的go字面值； |
| ‘0’  | 使用0而不是空格填充，对于数值类型会把填充的0放在正负号后面； |

!!! example

    ```go
    import "fmt"

    func main() {
        s := "Boii"
        fmt.Printf("|%s| \n", s)     // |Boii|
        fmt.Printf("|%5s| \n", s)    // | Boii|
        fmt.Printf("|%-5s| \n", s)   // |Boii |
        fmt.Printf("|%5.7s| \n", s)  // | Boii|
        fmt.Printf("|%-5.7s| \n", s) // |Boii |
        fmt.Printf("|%5.2s| \n", s)  // |   Bo|
        fmt.Printf("|%05s| \n", s)   // |0Boii|
        
        d := -100
        fmt.Printf("|%d| \n", d)     // |-100|
        fmt.Printf("|%5d| \n", d)    // | -100|
        fmt.Printf("|%-5d| \n", d)   // |-100 |
        fmt.Printf("|%5.7d| \n", d)  // |-0000100|
        fmt.Printf("|%-5.7d| \n", d) // |-0000100|
        fmt.Printf("|%5.2d| \n", d)  // | -100|
        fmt.Printf("|%05d| \n", d)   // |-0100|
        
        d = 18
        fmt.Printf("|%d| \n", d)     // |18|
        fmt.Printf("|%5d| \n", d)    // |   18|
        fmt.Printf("|%-5d| \n", d)   // |18   |
        fmt.Printf("|%5.7d| \n", d)  // |0000018|
        fmt.Printf("|%-5.7d| \n", d) // |0000018|
        fmt.Printf("|%5.2d| \n", d)  // |   18|
        fmt.Printf("|%05d| \n", d)   // |00018|
    }
    ```

