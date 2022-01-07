# os

os 包提供了与操作系统无关的接口，使用这个包的时候可以不考虑操作系统之间的差异。

提供的方法有很多，很大部分是与文件操作相关，下面是与文件操作相关的函数和方法：

(f 表示一个 FileInfo 对象)
(fd 表示文件描述符，如读、写、创建等 )

| 方法                                        | 说明                                                                                                                                                     |
| :----------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **打开和关闭文件**                            |                                                                                                                                                         |
| Open(name) (\*File, error)                 | 打开 name 文件，文件fd为只读 os.O_RDONLY                                                                                                                    |
| OpenFile(name, flag, perm) (\*File, error) | 打开 name 文件，文件fd由flag指定，文件不存在则创建，创建权限为perm                                                                                                |
| f.Close()                                  | 关闭文件                                                                                                                                                  |
| **创建文件、目录**                            |                                                                                                                                                         |
| Stat(name) (FileInfo, error)               | 用于返回一个文件或目录的文件信息                                                                                                                              |
| Create(name) (\*File, error)               | 创建（文件不存在）文件或清空（文件已存在）文件内容。创建权限0666，fd 为读写 os.O_RDWR                                                                                |
| NewFile(fd, name) \*File                   | 创建文件，fd需指定。                                                                                                                                        |
| Mkdir(name, perm) error                    | 创建目录，权限需指定                                                                                                                                        |
| MkdirAll(path, perm) error                 | 递归式创建目录，权限需指定，并会应用到子目录                                                                                                                    |
| **删除文件、目录，重命名**                      |                                                                                                                                                         |
| Remove(name) error                         | 删除文件或空目录                                                                                                                                           |
| RemoveAll(path) error                      | 递归式删除文件或目录及其子目录                                                                                                                                |
| Rename(oldPath, newPath) error             | 重命名，与 mv 命令相似，可以起到移动文件的作用                                                                                                                  |
| **读、写、裁减文件， 读取目录**                 |                                                                                                                                                         |
| ReadFile(name) ([]byte, error)             | 读取文件内容，没有缓冲区                                                                                                                                     |
| WriteFile(name, data, perm) (error)        | 覆盖写入 data 到 name 文件中，如果文件不存在则创建，权限为 perm，没有缓冲区                                                                                        |
| Truncate(name, size) error                 | 将 name 文件裁减到 size 大小，size=0时则是清空文件内容                                                                                                         |
| ReadDir(name) ([]DirEntry, error)          | 返回目录下所有文件（包括目录），按文件名排序                                                                                                                    |
| f.Read(b) (int, error)                     | 从f中读取最多len(b)字节数据并写入b。返回读取的字节数和可能遇到的任何错误。文件终止标志是读取0个字节且返回值err为io.EOF。                                                   |
| f.ReadAt(b, off) (int, error)              | 从指定的位置（相对于文件开始位置）读取len(b)字节数据并写入b。返回读取的字节数和可能遇到的任何错误。当n<len(b)时，本方法总是会返回错误；如果是因为到达文件结尾，返回值err会是io.EOF。 |
| f.Seek(off, whence) (int64, error)         | 设置f的下一次读/写的位置。offset为相对偏移量，而whence决定相对位置,返回新的偏移量（相对开头）和可能的错误。                                                              |
| f.Write(b) (int, error)                    | 向文件中写入len(b)字节数据。返回写入的字节数和可能遇到的任何错误。如果返回值n!=len(b)，本方法会返回一个非nil的错误。                                                       |
| f.WriteAt(b, off) (int, error)             | 将len(b)字节写入文件，从字节偏移开始。                                                                                                                        |
| f.WriteString(string) (int, error)         | 类似Write，参数为字符串。                                                                                                                                   |
| f.Sync() error                             | 将最近写入的数据同步进硬盘                                                                                                                                   |
| **判断比较**                                 |                                                                                                                                                         |
| IsExist(err) bool                          | 返回文件、目录是否存在                                                                                                                                      |
| IsNotExist(err) bool                       | 返回文还、目录是否不存在                                                                                                                                     |
| SameFile(fi1, fi2) bool                    | 返回 fi1 和 fi2 是否同一个文件                                                                                                                              |
| **更改权限**                                 |                                                                                                                                                         |
| Chmod(name, mode) error                    | 更改 name 文件的权限为 mode                                                                                                                                |
| Chown(name, uid, gid) error                | 更改 name 文件的拥有者                                                                                                                                     |
| Chtimes(name, atime, mtime) error          | 更改 name 文件的访问时间为 atime，修改时间为 mtime                                                                                                            |
| Chdir(dir) error                           | 更改当前工作目录为 dir                                                                                                                                      |

## 打开和关闭文件

`func Open(name string) (*File, error)`：打开 name 文件，文件fd为只读 os.O_RDONLY
`OpenFile(name, flag, perm) (*File, error)`： 打开 name 文件，文件fd由flag指定，文件不存在则创建，创建权限为perm
`f.Close()`：关闭文件

在打开文件时一定要养成一个好习惯：顺手延迟关闭。

```go
package main

import (
    "os"
    "log"
)

func main() {
    f, err := os.Open("/home/boii/a.txt")    // 打开文件，默认只读模式
    if err != nil {
        log.Fatalln(err)
    }
    defer f.Close()

    log.Println("文件已打开。")
}
```

```go
package main

import (
    "os"
    "log"
)

func main() {
    // 打开文件，可读可追加写，文件不存在时创建，权限为0511
    f, err := os.OpenFile("/home/boii/a.txt", os.O_CREATE|os.O_RDWR|os.O_APPEND, os.ModePerm)
    if err != nil {
        log.Fatalln(err)
    }
    defer f.Close()

    log.Println("文件已打开。")
}
```

## 创建文件、目录

`Stat(name) (FileInfo, error)`：用于返回一个文件或目录的文件信息
`Create(name) (*File, error)`：创建（文件不存在）文件或清空（文件已存在）文件内容。创建权限 0666，fd 为读写 os.O_RDWR
`Mkdir(name, perm) error`：创建目录，权限需指定
`MkdirAll(path, perm) error`：递归式创建目录，权限需指定，并会应用到子目录


- `Stat(name) (FileInfo, error)`：用于返回一个文件或目录的文件信息
    ```go
    func main() {
        fileInfo, err := os.Stat("/home/boii/a.txt")
        if err != nil {
            log.Fatalln("Err:", err)
        }
        fmt.Println(fileInfo.Name())    // abc.txt
        fmt.Println(fileInfo.Size())    // 0
        fmt.Println(fileInfo.Mode())    // -rw-rw-rw-
        fmt.Println(fileInfo.ModTime()) // 2021-02-08 21:29:31.4224976 +0800 CST
        fmt.Println(fileInfo.IsDir())   // false
        fmt.Println(fileInfo.Sys())     // &{32 {1857742672 30866974} {1857742672 30866974} {1857742672 30866974} 0 0}
    }
    ```

- `Create(name) (*File, error)`：
    - 如果文件不存在：创建，权限0666，fd 为读写 os.O_RDWR
    - 如果文件已存在：清空文件内容

    ```go
    func main() {
        f, err := os.Create("/home/boii/a.txt")    // 文件不存在则创建，权限0666，返回的 f 的模式为读写
        if err != nil {
            log.Fatalln(err)
        }
        f.Write([]byte("abc"))
    }
    ```

- `Mkdir(name, perm) error`：创建目录，权限需指定。如果创建目录为 /a/b/c，而 b 不存在，则会创建失败。
    ```go
    func main() {
        // 目录 /home/boii 存在，所以目录 abc 会创建成功
        err := os.Mkdir("/home/boii/abc", os.ModePerm)
        if err != nil {
            log.Fatalln(err)
        }
    }
    ```

    ```go
    func main() {
        // 目录 /home/boii/xyz 不存在，所以目录 abc 会创建失败
        err := os.Mkdir("/home/boii/xyz/abc", os.ModePerm)
        if err != nil {
            log.Fatalln(err)
        }
    }
    ```

- `MkdirAll(path, perm) error`：递归式创建目录，权限需指定，并会应用到子目录
    ```go
    func main() {
        // 目录 /home/boii/xyz 不存在，会先创建目录 xyz，然后创建目录 abc
        err := os.MkdirAll("/home/boii/xyz/abc", os.ModePerm)
        if err != nil {
            log.Fatalln(err)
        }
    }
    ```

## 删除文件、目录，重命名
`Remove(name) error`：删除文件或空目录
`RemoveAll(path) error`：递归式删除文件或目录及其子目录
`Rename(oldPath, newPath) error`：重命名，与 mv 命令相似，可以起到移动文件的作用

```go
func main() {
    if err := os.Remove("/home/boii/a.txt"); err != nil {
        lof.Fatal(err)
    }
}
```

```go
func main() {
    // 这种具体到文件的方式只会删除文件
    if err := os.RemoveAll("/home/boii/abc/a.txt"); err != nil {
        lof.Fatal(err)
    }

    // 注意：这种具体到目录的方式，目录 abc 下所有文件连同目录 abc 本身都会被删除
    if err := os.RemoveAll("/home/boii/abc/"); err != nil {
        log.Fatal(err)
    }
```

`Rename(oldPath, newPath) error`：重命名，与 mv 命令相似，可以起到移动文件的作用
```go
    if err := os.Rename("/home/boii/abc", "/home/boii/xyz"); err != nil {
        log.Fatal(err)
    }
}
```

## 读、写、裁减文件， 读取目录
`f.Seek(off, whence) (int64, error)`：设置读写位置
`f.Read(b) (int, error)`：读取文件内容
`f.Write(b) (int, error)`：内容写入文件
`f.WriteString(string) (int, error)`：写入字符串到文件

`ReadFile(name) ([]byte, error)`：读取文件内容，没有缓冲区
`WriteFile(name, data, perm) (error)`：覆盖写入 data 到 name 文件中，如果文件不存在则创建，权限为 perm，没有缓冲区
`Truncate(name, size) error`：将 name 文件裁减到 size 大小，size=0时则是清空文件内容
`ReadDir(name) ([]DirEntry, error)`：返回目录下所有文件（包括目录），按文件名排序

- `f.Read(b) (int, error)`：读取文件内容
这个方法从文件中最多读取`len(b)`字节。返回读取的字节数和遇到的任何错误。如果读取位置在文件末尾，Read返回`0，io.EOF`

```go
func main() {
    // 1. 打开文件
    f, err := os.Open("/home/boii/a.txt")
    if err != nil {
        log.Fatal(err)
    }

    // 3. 关闭文件
    defer f.Close()

    // 2. 按字节读取
    b := make([]byte, 1024)
    for {
        n, err := f.Read(b)
        if n == 0 || err == io.EOF {
            break
        }
        log.Println(b[:n])
    }
}
```
- `f.Write(b) (int, error)`：内容写入文件
    ```go
    func main() {
        // 1. 打开文件，若文件不存在则创建，权限0511，并以覆盖只写模式打开
        f, err := os.OpenFile("/home/boii/a.txt", os.O_CREATE | os.O_WRONLY, os.ModePerm)
        if err != nil {
            log.Fatal(err)
        }
        // 3. 关闭文件
        defer f.Close()
    
        // 2. 写入数据
        b := []byte{65, 66, 67, 68, 69}    // A, B, C, D, E
        n, err := f.Write(b)
        if err != nil {
            log.Fatal(err)
        }
        log.Println(n)
    }
    ```



- `f.Write(b) (int, error)`：内容写入文件

## 判断比较
`IsExist(err) bool`：返回文件、目录是否存在
`IsNotExist(err) bool`：返回文还、目录是否不存在
`SameFile(fi1, fi2) bool`：返回 fi1 和 fi2 是否同一个文件

- `IsExist(err) bool`：返回文件、目录是否存在；`IsNotExist(err) bool`：返回文还、目录是否不存在
    ```go
    func main() {
        f, err := os.Stat("/home/boii/xyz")
        if os.IsNotExist(err) {
            log.Fatal("文件不存在")
        }
    
        log.Println("文件已存在", f.Name())
    }
    ```
- `SameFile(fi1, fi2) bool`：返回 fi1 和 fi2 是否同一个文件
    ```go
    func main() {
        f1, _ := os.Stat("xyz.sh")
        f2, _ := os.Stat("xyz.sh")
    
        if os.SameFile(f1, f2) {
            log.Println("同一个文件。")
        }
    }
    ```
- `f.Seek(off, whence) (int64, error)`：设置读写位置
    whence 可以填3个值：

    - `0 或 os.SEEK_SET`：设置新的读写位置为：第 **off** 字节，`must off > 0`
    - `1 或 os.SEEK_CUR`：设置新的读写位置为：第 **当前读写位置+off** 字节, `off > 0 || off < 0`
    - `2 或 os.SEEK_END`：设置新的读写位置为：第 **文件结尾+off** 字节, `off > 0 || off < 0`

    返回值为新的偏移量，这个偏移量是相对于文件开头而言的。

    几种特殊用法：

    - f.Seek(0, os.SEEK_SET)：将读写位置移到文件头
    - f.Seek(0, os.SEEK_END)：将读写位置移到文件尾，返回值为文件大小
    - f.Seek(0, os.SEEK_CUR)：取得当前文件读写位置

    ```go
    func main() {
        f, err := os.OpenFile("/home/boii/a.txt", os.O_CREATE|os.O_WRONLY, os.ModePerm)
        if err != nil {
            log.Fatal(err)
        }
        // 3. 关闭文件
        defer f.Close()
    
        // 2. 写入数据
        b := []byte{65, 66, 67, 68, 69} // A, B, C, D, E
    
        f.Seek(-3, os.SEEK_END)     // 设置读写位置为文件末尾倒退3个字节
        n, err := f.Write(b)        // 写入数据
        if err != nil {
            log.Fatal(err)
        }
        log.Println(n)
    }
    ```

    ```
    /home/boii/a.txt
    
    写入前：
    abcdefg
    hijklmn
    opqrst
    uvwxyz
    
    写入后：
    abcdefg
    hijklmn
    opqrst
    uvwABCDE
    ```

- `f.WriteAt(b, off) (int, error)`
相当于 `f.Seek(off, os.SEEK_SET)` + `f.Write(b)`，也就是上面的栗子

- `ReadFile(name) ([]byte, error)`：读取文件内容，没有缓冲区
    相当于 `f, err := os.Open(name)` + `f.Read(b)`

    ```go
    func main() {
    
        data, err := os.ReadFile("/home/boii/a.txt")
        if err != nil {
            log.Fatal(err)
        }
        os.Stdout.Write(data)
    }
    ```

- `WriteFile(name, data, perm) (error)`：覆盖写入 data 到 name 文件中，如果文件不存在则创建，权限为 perm，没有缓冲区
    相当于 `f, err := os.Open(name)` + `f.Write(b)`

    ```go
    func main() {
        err := os.WriteFile("/home/boii/a.txt", []byte("Hello, Gophers!"), 0666)
        if err != nil {
            log.Fatal(err)
        }
    }
    ```
- `Truncate(name, size) error`：将 name 文件裁减到 size 大小，size=0时则是清空文件内容

    ```go
    func main() {
        if err := os.Truncate("home/boii/a.txt", 0); err != nil {
            log.Println("文件内容已清空")
        }
    }
    ```

## 更改权限
`Chmod(name, mode) error`：更改 name 文件的权限为 mode
`Chown(name, uid, gid) error`：更改 name 文件的拥有者
`Chtimes(name, atime, mtime) error`：更改 name 文件的访问时间为 atime，修改时间为 mtime
`Chdir(dir) error`：更改当前工作目录为 dir



















