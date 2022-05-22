# IM9-客户端框架

之前我们都是用 `netcap` 来模拟客户端，现在服务端已经实现的差不多了，接下来我们就手动实现一个客户端。

## 搭建客户端框架

之前我们用来模拟客户端的命令是这样的：`nc 127.0.0.1 8088`；搭建一个客户端，首先就要模拟这个 `nc` 命令。

### Client.go
第一步我们需要一个 `Client` 结构体，带有 IP地址 和 端口，然后通过一个工厂函数返回一个 `client` 实例。

```go
// client.go
package main

import (
	"fmt"
	"log"
	"net"
)

// Client 客户端句柄
type Client struct {
	Name       string
	ServerIP   string
	ServerPort int
	conn       net.Conn
}

// NewClient 返回一个 Client 实例
func NewClient(serverIP string, serverPort int) *Client {

	conn, err := net.Dial("tcp", fmt.Sprintf("%s:%d", serverIP, serverPort))
	if err != nil {
		log.Println("net.Dial Err: ", err)
		return nil
	}

	return &Client{
		ServerIP:   serverIP,
		ServerPort: serverPort,
		conn:       conn,
	}
}
```
上面的代码中，我们不仅为 `Client` 结构体定义了 IP和端口字段，还定义了 Name 和 conn 字段。

在工厂函数中，调用了 `net.Dial()` 函数来执行发送请求的任务。

`net.Dial()` 是 `net` 包下一个用来发送请求的函数，其函数签名为：
```go
func Dial(network , address string) (Conn, error)
```

它接受两个参数，一个是网络类型，一个是地址。

网络类型可以是：`tcp、tcp4、tcp6、udp、udp4、udp6、ip、ip4、ip6、unix、unixgram、unixpacket`。

请求成功后会返回一个连接对象 `conn`。

## Main.go
现在我们到 `main.go` 中的 `main()` 函数里创建一个 `client` 实例：
```go
func main() {
    client := NewClient("127.0.0.1", 8088)
    if client == nil {
		log.Println(">>>>> 连接服务器失败...")
		return
	}
	log.Println(">>>>> 连接服务器成功...")

	// 启动客户端业务
	select {}
}
```

## 编译运行

```bash
$ go build -o client ./main.go ./client.go

$ ./client
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/1119756198577.png)


## 完整代码

??? example

    === "client.go"

        ```go
        // client.go
        package main

        import (
            "fmt"
            "log"
            "net"
        )

        // Client 客户端句柄
        type Client struct {
            Name       string
            ServerIP   string
            ServerPort int
            conn       net.Conn
        }

        // NewClient 返回一个 Client 实例
        func NewClient(serverIP string, serverPort int) *Client {

            conn, err := net.Dial("tcp", fmt.Sprintf("%s:%d", serverIP, serverPort))
            if err != nil {
                log.Println("net.Dial Err: ", err)
                return nil
            }

            return &Client{
                ServerIP:   serverIP,
                ServerPort: serverPort,
                conn:       conn,
            }
        }
        ```

    === "main.go"

        ```go
        // main.go
        package main

        import "log"

        func init() {
            log.SetFlags(log.LstdFlags | log.Lshortfile)
        }

        func main() {

            client := NewClient("127.0.0.1", 8088)
            if client == nil {
                log.Println(">>>>> 连接服务器失败...")
                return
            }
            log.Println(">>>>> 连接服务器成功...")

            // 启动客户端业务
            select {}
        }
        ```
