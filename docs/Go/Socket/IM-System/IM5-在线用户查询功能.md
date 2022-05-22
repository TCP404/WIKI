# IM5-在线用户查询

在线用户查询，思路是这样的：
首先肯定得用户发送一条什么指令，然后服务端在接收到这条指令之后作出对应的处理。

在 [《IM4-用户业务封装》](./IM4-%E4%B8%9A%E5%8A%A1%E5%B0%81%E8%A3%85.md) 中，我们在 服务端的 `handler()` 中启动一个 `goroutine` 去执行 `massTexting()` 接收客户端发送的消息，
然后在 `massTexting()` 中把消息内容交给 `user.HandleMessage(msg)` 处理。

那么接下来我们要做的，就是在 `user.HandleMessage()` 中根据内容的不同进行不同的处理。


先看下原本的 `HandleMessage()`
```go
// DoMessage 用户处理消息功能
func (u *User) HandleMessage(msg string) {
	u.server.generateMsg(u, msg)
}
```

现在我们需要根据 msg 内容的不同进行不同的处理，我们可以使用 `switch`。
```go
// HandleMessage 用户处理消息功能
func (u *User) HandleMessage(msg string) {
	switch msg {
	case "alluser":    	// 查询当前在线用户
		u.showOnlineUser()
	default:
		u.server.generateMsg(u, msg)
	}
}
```

这样如果用户输入的是 `alluser` 就会去调用 `showOnlineUser()` 这个方法。

展示也简单，就是遍历 onlineMap，然后打印出来即可。

```go
// 打印所有在线用户
func (u *User) showOnlineUser() {
    // 查询逻辑，遍历在线用户列表
	u.server.mapLock.Lock()
	userName := ""
	for _, user := range u.server.onlineMap {
		userName += user.Name + "\n"
	}
	u.server.mapLock.Unlock()
	u.conn.Write([]byte(userName))
}
```

## 编译运行
```shell
$ go build -o server server.go main.go user.go

$ ./server
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/5907610258493.png)

## 完整代码

??? example

    === "user.go"

        ```go
        // user.go
        package main

        import (
            "net"
            "strconv"
        )

        type User struct {
            Name   string
            Addr   string
            C      chan string
            conn   net.Conn
            server *Server
        }

        var count int

        func NewUser(conn net.Conn, server *Server) *User {
            count++
            user := &User{
                Name:   "user" + strconv.Itoa(count),
                Addr:   conn.RemoteAddr().String(),
                C:      make(chan string),
                conn:   conn,
                server: server,
            }
            go user.listenMessage()
            return user
        }

        // 监听当前User channel 的方法，一旦有消息就直接发送给对端客户端
        func (u *User) listenMessage() {
            for {
                msg := <-u.C
                u.conn.Write([]byte(msg + "\n"))
            }
        }

        // Online 用户上线功能
        func (u *User) Online() {
            // 用户上线，将用户加入 onlinemap
            u.server.mapLock.Lock()
            u.server.onlineMap[u.Name] = u
            u.server.mapLock.Unlock()

            // 用户上线广播
            u.server.generateMsg(u, "已上线")

        }

        // Offline 用户下线功能
        func (u *User) Offline() {
            // 用户下线，将用户从 onlineMap 中移除
            u.server.mapLock.Lock()
            delete(u.server.onlineMap, u.Name)
            u.server.mapLock.Unlock()

            // 用户下线广播
            u.server.generateMsg(u, "已离线")

        }

        // HandleMessage 用户处理消息功能
        func (u *User) HandleMessage(msg string) {
            switch msg {
            case "alluser":		// 查询当前在线用户
                u.showOnlineUser()
            default:
                u.server.generateMsg(u, msg)
            }
        }

        // 打印所有在线用户
        func (u *User) showOnlineUser() {
            u.server.mapLock.Lock()
            userName := ""
            for _, user := range u.server.onlineMap {
                userName += user.Name + "\n"
            }
            u.server.mapLock.Unlock()
            u.conn.Write([]byte(userName))
        }
        ```

    === "server.go"

        ```go
        // server.go
        package main

        import (
            "fmt"
            "io"
            "net"
            "sync"
        )

        type Server struct {
            IP        string
            Port      int
            onlineMap map[string]*User // 在线用户表onlinemap
            mapLock   sync.RWMutex
            message   chan string // 消息广播的 channel
        }

        func NewServer(IP string, port int) *Server {
            return &Server{
                IP:        IP,
                Port:      port,
                onlineMap: make(map[string]*User),
                message:   make(chan string),
            }
        }

        func (s *Server) Start() {
            address := fmt.Sprintf("%s:%d", s.IP, s.Port)
            // new
            listener, err := net.Listen("tcp", address)
            if err != nil {
                fmt.Println("Listen err:", err)
                return
            }
            // close
            defer listener.Close()

            fmt.Println("=> Server is listening on :", address)

            // 启动监听消息生成器的 goroutine
            go s.broadcast()

            for {
                // Accept
                conn, err := listener.Accept()
                if err != nil {
                    fmt.Println("Conn Err:", err)
                }
                // handle
                go s.handler(conn)
            }
        }

        func (s *Server) handler(conn net.Conn) {
            fmt.Println("Connection Successfuly.\n")
            conn.Write([]byte("Connection Successfuly.\n"))

            user := NewUser(conn, s)
            user.Online()
            // 接收客户端发送的消息
            go s.massTexting(conn, user)

            // 当前广播阻塞
            select {}
        }

        func (s *Server) massTexting(conn net.Conn, user *User) {
            buf := make([]byte, 4096)
            for {
                n, err := conn.Read(buf) // 从客户端读取输入
                if err != nil && err != io.EOF {
                    fmt.Println("Conn Read Err:", err)
                    return
                }
                if n == 0 || err == io.EOF {
                    user.Offline()
                    return
                }
                // 提取用户的消息（去除\n）
                msg := string(buf[:n-1])
                // 针对msg 进行处理
                user.HandleMessage(msg)
            }
        }

        // 消息生成器
        func (s *Server) generateMsg(user *User, msg string) {
            sendMsg := "[" + user.Addr + "]" + user.Name + ": " + msg
            s.message <- sendMsg
        }

        // 监听消息通道，然后进行播送
        func (s *Server) broadcast() {
            for {
                msg := <-s.message
                s.mapLock.Lock()
                for _, cli := range s.onlineMap {
                    cli.C <- msg
                }
                s.mapLock.Unlock()
            }
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