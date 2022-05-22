# IM3-用户消息群发

群发其实就是广播，实现用户消息的群发很简单，
在 [《IM2-用户上线广播》](./IM2-%E4%B8%8A%E7%BA%BF%E5%B9%BF%E6%92%AD%E5%8A%9F%E8%83%BD.md) 中，我们利用通道来进行广播，我们可以继续使用这个通道来群发。

思路也简单，接收客户端的输入，放到广播里就行了。

## 接收客户端输入

客户端和服务端建立连接之后由 `handler` 处理接下来的事情，

```go
func (s *Server) handler(conn net.Conn) {
	fmt.Println("Connection Successfuly.\n")
	conn.Write([]byte("Connection Successfuly.\n"))

	// 将用户加入 onlinemap
	user := NewUser(conn)
	s.mapLock.Lock()
	s.onlineMap[user.Name] = user
	s.mapLock.Unlock()

	// 用户上线广播
	s.generateMsg(user, "已上线")

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
			s.generateMsg(user, "已离线")
			return
		}

		// 提取用户的消息（去除\n）
		msg := string(buf[:n-1])
		// 将得到的消息进行广播
		s.generateMsg(user, msg)
	}
}
```
在 handler 中我们开启一个 `goroutine` 负责广播客户端发的消息，然后我们在 `massTexting` 这个方法中去处理

在广播之前我们先将消息的换行符去掉，然后交给消息生成器。
消息生成器去生成后会放到消息通道，然后广播器会去读取消息通道中的消息，并进行广播，也就是群发。


## 编译运行

```shell
$ go build -o server server.go main.go user.go

$ ./server
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/4622913776016.png)

## 完整代码

??? example

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

            // 将用户加入 onlinemap
            user := NewUser(conn)
            s.mapLock.Lock()
            s.onlineMap[user.Name] = user
            s.mapLock.Unlock()

            // 用户上线广播
            s.generateMsg(user, "已上线")

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
                    s.generateMsg(user, "已离线")
                    return
                }
                // 提取用户的消息（去除\n）
                msg := string(buf[:n-1])
                // 将得到的消息进行广播
                s.generateMsg(user, msg)
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
    === "user.go"

        ```go
        // user.go
        package main

        import (
            "net"
            "strconv"
        )

        type User struct {
            Name string
            Addr string
            C    chan string
            conn net.Conn
        }

        var count int

        func NewUser(conn net.Conn) *User {
            count++
            user := &User{
                Name: "user" + strconv.Itoa(count),
                Addr: conn.RemoteAddr().String(),
                C:    make(chan string),
                conn: conn,
            }
            go user.listenMessage()    // 启动监听好友上线的 goroutine
            return user
        }

        // 监听当前User channel 的方法，一旦有消息就直接发送给对端客户端
        func (u *User) listenMessage() {
            for {
                msg := <-u.C
                u.conn.Write([]byte(msg + "\n"))
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