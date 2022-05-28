# 6-bufio 缓冲IO

`bufio`是通过缓冲来提高效率的。

io操作本身效率并不低，低的是频繁访问本地磁盘的文件。所以 `bufio` 就提供了缓冲区（分配一块内存），读写都是先在缓冲区中，最后再读写进文件，从而降低访问本地磁盘的次数，提高效率。

- 第一次读取的时候，会从本地磁盘将数据读取到缓冲区中，之后第二次第三次读取就会直接从缓冲区读取，避免了每次都去访问本地磁盘。

- 第一次写入的时候，会将数据先写入到缓冲区，等到缓冲区写满了或者手动刷新，才会把数据写入到本地磁盘上的文件中。

这种缓冲区的设计非常适合 `I/O密集型` 的程序。

## 导包

```go
import "bufio"
```

## bufio 原理

在读取的时候，如果用于存放数据的 `[]byte` 大小超过缓冲区大小，实际上缓冲区是不起作用的。写入同理。

```go
func main() {
    f, err := os.Open("abc.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()

    buf1 := bufio.NewReaderSize(f, 4096)    // buf 区大小 4096 byte
    p := make([]byte, 10000)                // []byte 大小 10000 byte
    buf1.Read(p)                            // 此时buf区实际上不起作用
}
```

![bufio 原理](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/IMG/20210210115116792_10702.png)


## bufio 使用步骤
1. 打开文件
2. `defer` 关闭文件
3. 创建 `buf` 对象
4. 使用 `buf` 读写

```go
func main() {
// 读取
    f1, err := os.Open("abc.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer f1.Close()

    bufR := bufio.NewReader(f1)    // 创建 buf 对象
    p1 := make([]byte, 1024)
    bufR.Read(p1)                  // 使用 buf 读取

// 写入
    f2, err := os.OpenFile("abc2.txt", os.O_CREATE|os.O_WDONLY, 0777)
    if err != nil {
        log.Fatal(err)
    }
    defer f2.Close()

    bufW := bufio.NewWriter(f2)      // 创建 buf 对象
    bufW.Write([]byte("Hello World")) // 使用 buf 写入
    bufW.Flush()    // 刷新缓冲区，也就是手动写入文件。
}
```

## 读取

方法签名：
```go
func NewReader(rd io.Reader) *Reader {}                // 创建一个 bufio.Reader 对象，默认缓冲区大小 4096 byte
func NewReaderSize(rd io.Reader, size int) *Reader {}  // 创建一个 bufio.Reader 对象，指定缓冲区大小 size byte

func (b *Reader) Read(p []byte) (n int, err error) {}           // 读取数据到 p 中
func (b *Reader) ReadByte() (byte, error) {}                    // 读取一个 byte 的数据
func (b *Reader) ReadRune() (r rune, size int, err error) {}    // 读取一个 rune 的数据
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error) {} // 读取一行数据,不建议使用
func (b *Reader) ReadBytes(delim byte) ([]byte, error) {}              // 遇到指定的字符 delim 停止，返回 []byte 格式
func (b *Reader) ReadString(delim byte) (string, error) {}             // 遇到指定的字符 delim 停止，返回 string 格式
func (b *Reader) ReadSlice(delim byte) (line []byte, err error) {}
```

!!!example

    ```go
    func main() {
        f, err := os.Open("abc.txt")
        if err != nil {
            log.Fatal(err)
        }
        defer f.Close()

        buf := bufio.NewReader(f)
        s, err, := buf.ReadString("\n")    // 遇到第一个换行符时停止读取
        fmt.Println(s)    // Hello World
    }
    ```

## 写入

方法签名：

```go
func NewWriter(w io.Writer) *Writer {}                // 创建一个 bufio.Writer 对象，默认缓冲区大小 4096 byte
func NewWriterSize(w io.Writer, size int) *Writer {}  // 创建一个 bufio.Writer 对象，指定缓冲区大小 size byte

func (b *Writer) Write(p []byte) (nn int, err error) {}        // 写入 p 中的数据，返回写入的字节数和错误
func (b *Writer) WriteByte(c byte) error {}                    // 写入一个 byte 数据，写入失败返回错误，写入成功返回 nil
func (b *Writer) WriteRune(r rune) (size int, err error) {}    // 写入一个 rune 数据，返回写入的字节数和错误
func (b *Writer) WriteString(s string) (int, error) {}         // 写入一个字符串，返回写入的字节数和错误
```

!!!example

    ```go
    func main() {
        f, err := os.OpenFile("abc.txt", os.O_WRONLY, 0777)
        if err != nil {
            log.Fatal(err)
        }
        defer f.Close()

        buf := bufio.NewWriter(f)
        size, err := buf.WriteRune('牛')
        fmt.Println(size)    // 3
    }
    ```

`bufio.Writer` 还有一些工具方法：

```go
func (b *Writer) Size() int {}            // 返回缓冲区的大小
func (b *Writer) Available() int {}       // 返回缓冲区当前 可用 的大小
func (b *Writer) Buffered() int {}        // 返回缓冲区当前 已用 的大小
func (b *Writer) Reset(w io.Writer) {}    // 丢弃缓冲区的数据，并把缓冲区清空
func (b *Writer) Flush() error {}         // 将缓冲区数据写入文件，并把缓冲区清空
```
