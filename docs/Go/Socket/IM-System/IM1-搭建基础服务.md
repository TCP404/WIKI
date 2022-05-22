# IM1

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/2566840859.png)

## 前期准备

IM 即时通讯，就是一端开放一个端口，并启动一个程序一直监听这个端口，当有人发送请求过来的时候接受然后处理；而另一端对那台机器的端口发送一个请求。

接受并处理的这一端我们通常称为服务端，发送的这一端我们通常成为客户端。
（此处表述并不严谨，有个了解即可）


## 客户端

接下来我们先专注于服务端，客户端暂时用 `nc` 命令模拟。

- **Linux**
    Linux 下默认装有 `nc`

    没有的话在终端执行命令：
    ```shell
    $ sudo apt install netcat
    ```

    Manjaro 或 Arch Linux 发行版请使用命令 `ncat` 或 `netcat`；
    如果没有，请自行安装好 `yay` 第三方包管理工具，然后执行下面的命令：
    ```shell
    $ yay -S nmap-netcat
    ```

- **Windows**
    Windows 用户先下载 [netcat 工具](https://eternallybored.org/misc/netcat/netcat-win32-1.12.zip)，放在 `C:\User` 下或者自己知道的地方，
    然后添加到环境变量，记得先关掉杀毒软件。

## Server 端

首先构造一个 Server 结构体，装有 IP 和 Port 字段，并给他一个工厂函数用于返回 Server 实例。
```go
type Server struct {
    IP      string
    Port int
}

func NewServer (IP string, Port int) *Server {
    return &Server{IP, Port}
}
```

然后我们提供一个 Start 方法供实例调用，在方法中主要做4件事情：

1. 创建一个监听对象，负责监听一个端口号
2. 对发送来的请求进行接受，建立连接
3. 开启一个 `goroutine` 进行处理
4. 关闭监听对象


```go
func (s *Server) Start() {
    address := fmt.Sprintf("%s:%d", s.IP, s.Port)
    listener, err := net.Listen("tcp", address)    // 创建一个监听对象
    if err != nil {
        fmt.Println("Listen err:", err)
        return
    }
    defer listener.Close()    // 延迟关闭监听对象

    fmt.Println("=> Server listen on :", address)

    for {
        conn, err := listener.Accept()    // 对发送来的请求进行接收，建立连接
        if err != nil {
        fmt.Println("Conn Err:", err)
    }
    go s.handler(conn)    // 开启一个 goroutine 处理
    }
}
```

```go
func (s *Server) handler(conn net.Conn) {
    fmt.Println("Connection Successfuly.")                   // 服务端打印连接成功信息
    conn.Write([]byte("Connection Successfuly."))    // 客户端打印连接成功信息
}
```

Server 结构体的逻辑写完后，就可以到 `main()` 函数中创建 server 对象了。

```go
func main() {
    server := NewServer("127.0.0.1", 8088)    // 实例化一个 server 对象
    server.Start()    // 调用 Start 方法开启服务
}
```

## 编译运行


执行命令：
```go
$ go build -o server main.go server.go
$ ./server
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/233006238492.png)

左侧为服务端，运行 `./server` 后显示了服务监听着 `127.0.0.1：8088` 端口;
右侧为客户端，使用 `nc` 命令模拟，发送出请求后，服务端返回了 `Connection Successfuly.` 的字样，而服务端也打印出同样的字符串。

## 完整代码

??? example

    === "server.go"

        ```go
        // server.go
        package main

        import (
            "fmt"
            "net"
        )

        type Server struct {
            IP   string
            Port int
        }

        func NewServer(IP string, port int) *Server {
            return &Server{IP, port}
        }

        func (s *Server) Start() {
            address := fmt.Sprintf("%s:%d", s.IP, s.Port)
            // 1. listen
            listener, err := net.Listen("tcp", address)
            if err != nil {
                fmt.Println("Listen err:", err)
                return
            }
            // 4. close
            defer listener.Close()

            fmt.Println("=> Server listen on :", address)

            for {
                // 2. accept
                conn, err := listener.Accept()
                if err != nil {
                    fmt.Println("Conn Err:", err)
                }
                // 3. handle
                go s.handler(conn)
            }
        }

        func (s *Server) handler(conn net.Conn) {
            fmt.Println("Connection Successfuly.")
            conn.Write([]byte("Connection Successfuly."))
        }
        ```

    === "main.go"

        ```go
        // main.go
        package main

        func main() {
            server := NewServer("127.0.0.1", 8088)
            server.Start()
        }
        ```
