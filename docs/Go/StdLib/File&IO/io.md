# IO操作

IO操作基本三步：

1. 打开文件
2. 读写数据
3. 关闭文件

打开文件，其实是让程序和文件之间建立连接。关闭文件，其实就是断开连接。这两者缺一不可。

IO操作，无外乎 **读（Read）** 和 **写（Write）**。

我们写的程序是运行在内存中的，站在程序的立场上，

- 数据从磁盘进入内存，称之为 **读取（Read）、输入（Input）**；
- 数据从内存出去磁盘，称之为 **写入（Write）、输出（Output）**；

![IO](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210209103021253_12593.png)

```go
func main() {
    // // 1. 打开文件
    f, err := os.Open("trytrytry/abc.txt")
    if err != nil {
        fmt.Println(err)
    }
    // 3. 关闭文件
    defer f.Close()

    // 2. 读取数据
    bs := make([]byte, 4, 4)
    n, err := f.Read(bs)
    fmt.Println(err)
    fmt.Println(n)
    fmt.Println(bs)
}
```

## 读取
读取操作是一组 `File` 类型的方法

方法签名如下：
```go
func (f *File) Read(b []byte) (n int, err error) {}
func (f *File) ReadAt(b []byte, off int64) (n int, err error) {}
func (f *File) ReadFrom(r io.Reader) (n int64, err error) {}
func (f *File) Readdir(n int) ([]FileInfo, error) {}
func (f *File) Readdirnames(n int) (names []string, err error) {}
```

**`Read(b []byte)(n int, err error)`**：从文件头开始读取数据存放到 `b` 中，读取成功 返回 `n` 读取的字节数，`err` 为 `nil`；读取到文件末尾时 `err` 返回 0，即 `io.EOF`。

**`ReadAt(b []byte, off int64) (n int, err error)`**：从文件头开始第 `off` 个 字节读取数据存放到 `b` 中，读取成功 返回 `n` 读取的字节数，`err` 为 `nil`；读取到文件末尾时 `err` 返回 0，即 `io.EOF`。

**`ReadFrom(r io.Reader) (n int64, err error)`**：`io.ReaderFrom`的实现。

```go
func main() {
    // 1. 打开文件
    f, err := os.Open("abc.txt")
    if err != nil {
        log.Fatal(err)
        return
    }

    // 3. 关闭文件
    defer f.Close()

    // 2. 按字节读取
    b := make([]byte, 1024)
    for {
        n, err := f.Read(b)
        if n == 0 || err == io.EOF { break }
        fmt.Println(b[:n])
    }
}
```




## 写入
写入操作是一组 `File` 类型的方法

方法签名如下：
```go
func (f *File) Write(b []byte) (n int, err error) {}
func (f *File) WriteAt(b []byte, off int64) (n int, err error) {}
func (f *File) WriteString(s string) (n int, err error) {}
```

**`Write(b []byte) (n int, err error)`**：从文件头开始写入 `b` 中的数据，写入成功 返回 `n` 写入的字节数，`err` 为 `nil`。

**`WriteAt(b []byte, off int64) (n int, err error)`**：从文件头开始第 `off` 个 字节开始写入 `b` 中的数据，写入成功 返回 `n` 写入的字节数，`err` 为 `nil`。

**`WriteString(s string) (n int, err error)`**：写入一个字符串 `s` 到文件中，写入成功 返回 `n` 写入的字节数，`err` 为 `nil`。

!!!example "例1"

    ```go
    func main() {
        // 1. 打开文件，若文件不存在则创建，权限0777，并以只写模式打开
        f, err := os.OpenFile("abc.txt", os.O_CREATE | os.O_WRONLY, os.ModePerm)
        if err != nil {
            log.Fatal(err)
            return
        }
        // 3. 关闭文件
        defer f.Close()

        // 2. 写入数据
        b := []byte{65, 66, 67, 68, 69}    // A, B, C, D, E
        n, err := f.Write(b)
        if err != nil {
            log.Fatal(err)
            return
        }
        fmt.Println(n)
    }
    ```

!!!example "例2"

    ```go
    func main() {
        // 1. 打开文件，若文件不存在则创建，权限0777，并以只写模式打开
        f, err := os.OpenFile("abc.txt", os.O_CREATE | os.O_WRONLY, os.ModePerm)
        if err != nil {
            log.Fatal(err)
            return
        }
        // 3. 关闭文件
        defer f.Close()

        // 2. 写入数据
        b := []byte{65, 66, 67, 68, 69}    // A, B, C, D, E
        n, err := f.Write(b[:3])    // 只写入3个字节
        if err != nil {
            log.Fatal(err)
            return
        }
        fmt.Println(n)
    }
    ```

!!!example "例3"

    ```go
    func main() {
        // 1. 打开文件，若文件不存在则创建，权限0777，并以只写模式打开
        f, err := os.OpenFile("abc.txt", os.O_CREATE | os.O_WRONLY, os.ModePerm)
        if err != nil {
            log.Fatal(err)
            return
        }
        // 3. 关闭文件
        defer f.Close()

        // 2. 写入数据
        n, err := f.WriteString("Hello World")
        if err != nil {
            log.Fatal(err)
        }
        fmt.Println(n)
    }
    ```

## 复制文件

复制文件其实就是打开一个源文件，打开一个目标文件，然后从源文件读取，写入到目标文件。

Golang 中有三种方式完成。

-   方式一：使用按字节的方式循环复制。

```go
func copyFile1(destPath, srcPath string) (int, error) {
    f1, err := os.Open(srcPath)
    if err != nil {
        log.Fatal(err)
    }

    f2, err := os.OpenFile(destPath, os.O_CREATE|os.O_WRONLY, os.ModePerm)
    if err != nil {
        log.Fatal(err)
    }

    defer f1.Close()
    defer f2.Close()

    b := make([]byte, 1024)
    total := 0
    for {
        n, err := f1.Read(b)
        if err != nil {
            log.Fatal(err)
        }

        if n == 0 || err == io.EOF {
            fmt.Println("读取结束！")
            break
        } else if err != nil {
            log.Fatal(err)
            return total, err
        }

        total += n
        f2.Write(b[:n])
    }
    return total, nil
}
```

-   方式二：使用 `io.Copy()` 复制

```go
func copyFile2(destPath, srcPath string) (int64, error) {
    f1, err := os.Open(srcPath)
    if err != nil {
        return 0, err
    }
    f2, err := os.OpenFile(destPath, os.O_CREATE|os.O_WRONLY, os.ModePerm)
    if err != nil {
        return 0, err
    }
    defer f1.Close()
    defer f2.Close()

    return io.Copy(f1, f2)
}
```

-   方式三：使用 `ReadFile` 和 `WriteFile` 复制。但这种方式针对大文件时会出问题。因为它是一次性全部读取到内存中，如果文件太大会爆内存。

```go
func copyFile3(destPath, srcPath string) (int, error) {

    bs, err := ioutil.ReadFile(srcPath)
    if err != nil {
        return 0, err
    }

    err = ioutil.WriteFile(destPath, bs, 0777)
    if err != nil {
        return 0, err
    }

    return len(bs), nil
}
```

## 断点续传
我们在下载文件或者复制文件的时候，如果中途断电，或者暂停，那么重新传输的时候是否要一切从头开始？
答案是否定的。

下载文件等操作本质就是复制，想要实现断点续传，需要借助一个临时文件，在传输的时候不断把已传输完成的字节数写入临时文件。
如果因为某些原因突然断开传输，下一次重新传输的时候会先读取临时文件中的字节数，然后设置文件的读写位置，接着复制（传输）。
传输完成后，将临时文件删除即可。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
    "strconv"
)

func main() {
    src := "trytrytry/3.jpg"
    dest := "trytrytry/33.jpg"
    temp := "trytrytry/33jpg.temp.txt"
    breakPoint(dest, src, temp)
}

func breakPoint(dest, src, temp string) {

    srcf, err := os.Open(src)
    if err != nil {
        log.Fatal(err)
    }
    destf, err := os.OpenFile(dest, os.O_CREATE|os.O_WRONLY, 0777)
    if err != nil {
        log.Fatal(err)
    }
    tempf, err := os.OpenFile(temp, os.O_CREATE|os.O_RDWR, 0777)
    if err != nil {
        log.Fatal(err)
    }

    defer destf.Close()
    defer srcf.Close()
    defer tempf.Close()

    // 1. 读取临时文件中的数据
    bs := make([]byte, 100)
    tempf.Read(bs)
    count, err := strconv.ParseInt(string(bs), 10, 64)

    // 2. 设置读写位置
    srcf.Seek(count, io.SeekStart)
    destf.Seek(count, io.SeekStart)
    data := make([]byte, 8*1024)
    n2 := -1
    n3 := -1
    total := int(count) // 写入总量

    // 3. 复制文件
    for {
        n2, err = srcf.Read(data)
        if err == io.EOF || n2 == 0 {
            fmt.Println("文件复制完毕...", total)
            tempf.Close()
            os.Remove(temp)
            break
        }
        n3, err = destf.Write(data[:n2])
        total += n3
        // 将复制的总量，存储到临时文件中
        tempf.Seek(0, io.SeekStart)
        tempf.WriteString(strconv.Itoa(total))

        // if total > 100000 {
        // 	panic("模拟断电")
        // }
    }
}
```


## io.ioutil


### 导包
```go
import "io/ioutil"
```


### 函数集合

```go
func ReadAll(r io.Reader) ([]byte, error) {}
func ReadDir(dirname string) ([]os.FileInfo, error) {}
func ReadFile(filename string) ([]byte, error) {}
func WriteFile(filename string, data []byte, perm os.FileMode) error {}
func TempFile(dir, pattern string) (f *os.File, err error) {}
func TempDir(dir, pattern string) (name string, err error) {}
func NopCloser(r io.Reader) io.ReadCloser {}

type devNull int

func (devNull) Write(p []byte) (int, error) {}
func (devNull) WriteString(s string) (int, error) {}
func (devNull) ReadFrom(r io.Reader) (n int64, err error) {}
```

#### ReadAll

```go
func ReadAll(r io.Reader) ([]byte, error)
```

- 功能：ReadAll从r读取数据直到EOF或遇到error
- 返回值：
    - 返回读取的数据和遇到的错误
    - 成功的调用返回的err为nil而非EOF
- 因为本函数定义为读取r直到EOF，**它不会将读取返回的EOF视为应报告的错误**
- 底层实现：阅读该函数的源码发现，它是通过 **bytes.Buffer中的ReadFrom** 来实现读取所有数据的

#### ReadFile

```go
func ReadFile(filename string) ([]byte, error)
```

- 功能：ReadFile从filename指定的文件中 **读取数据并返回文件的内容**
- 返回值：成功返回的 err 为 nil 而非 EOF
- 因为本函数定义为读取整个文件，它 **不会将读取返回的EOF视为应报告的错误**
- ReadFile的是实现和ReadAll类似，不过，ReadFile会先判断文件的大小，给bytes.Buffer一个预定义容量，**避免额外分配内存**
- ReadFile源码中 **先获取了文件的大小**，当大小<1e9时，才会用到文件的大小。按源码中注释的说法是FileInfo不会很精确地得到文件大小

#### WriteFile

```go
func WriteFile(filename string, data []byte, perm os.FileMode) error
```
- 功能：它将data写入filename文件中，当文件不存在时会创建一个（文件权限由perm指定）；否则会先清空文件内容再写入

#### ReadDir

```go
func ReadDir(dirname string) ([]os.FileInfo, error)
```

- 功能：它读取目录并返回排好序的文件和子目录名（[]os.FileInfo）

#### TempDir

```go
func TempDir(dir, prefix string) (name string, err error)
```

- 功能：在dir目录里创建一个新的、使用prfix作为前缀的临时文件夹，并返回文件夹的路径
- **如果dir是空字符串**，表明在系统默认的临时目录中创建临时目录（参见os.TempDir）
- 不同程序同时调用该函数会创建不同的临时目录，调用本函数的程序有责任在不需要临时文件夹时删除它。代码如下所示

    ```go
    defer func() {
        f.Close()
        os.Remove(f.Name())
    }()
    ```

#### TempFile

```go
func TempFile(dir, prefix string) (f *os.File, err error)
```

- 功能：在dir目录下创建一个新的、使用prefix为前缀的临时文件，以读写模式打开该文件并返回os.File指针
- **如果dir是空字符串**，表明在系统默认的临时目录中创建临时文件（参见os.TempDir）
- 不同程序同时调用该函数会创建不同的临时文件，调用本函数的程序有责任在不需要临时文件时摧毁它。代码如下所示

    ```go
    defer func() {
        f.Close()
        os.Remove(f.Name())
    }()
    ```

#### NopCloser

```go
func NopCloser(r io.Reader) io.ReadCloser
```

- 功能：NopCloser用一个无操作的Close方法包装r返回一个ReadCloser接口
- 有时候我们需要传递一个io.ReadCloser的实例，而我们现在有一个io.Reader的实例，比如：strings.Reader，这个时候NopCloser就派上用场了。**它包装一个io.Reader，返回一个io.ReadCloser**，而相应的Close方法啥也不做，只是返回nil

!!!example

    在标准库net/http包中的NewRequest，接收一个io.Reader的body，而实际上，Request的Body的类型是io.ReadCloser，因此，代码内部进行了判断，如果传递的io.Reader也实现了io.ReadCloser接口，则转换，否则通过ioutil.NopCloser包装转换一下。相关代码如下：

    ```go
    rc, ok := body.(io.ReadCloser)
    if !ok && body != nil {
        rc = ioutil.NopCloser(body)
    }
    ```
