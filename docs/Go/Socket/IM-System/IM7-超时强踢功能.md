# IM7-超时强踢功能

实现超时强踢，首先想到的肯定是得有一个什么来记录是否活跃的状态。

每个用户的存活状态不一样，所以我们可以在 User 结构体 中增加一个字段 `isAlive`，类型为 `chan bool`。

```go
type User struct {
   	Name    string
    Addr    string
	C       chan string
	conn    net.Conn
	server  *Server
	isAlive chan bool    // 新增字段
}
```

接着进一步思考，超时的逻辑应该是，有一个计时器，每次客户端有动作，这个计时器就刷新重置；如果客户端一直没动作，则计时器会倒计时结束，然后执行强踢操作。

服务端和客户端建立连接之后，客户端的操作都是在 `server.go` 中的 `handler()` 中处理，我们看一眼 `handler()`：
```go
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
```

我们发现在建立连接之后，`handler()` 创建了 user 实例之后执行了上线功能，然后开启一个 `goroutine` 去执行 `massTexting()` 接收客户端发送来的消息。
那么我们应该在每次接受到客户端发来的消息后，刷新计时器。

而在 `handler()` 中我们需要一个定时器。

```go
func (s *Server) handler(conn net.Conn) {
	fmt.Println("Connection Successfuly.\n")
	conn.Write([]byte("Connection Successfuly.\n"))
	
	user := NewUser(conn, s)
	user.Online()
	
	// 接收客户端发送的消息
	go s.massTexting(conn, user)

	// 当前广播阻塞
	for {
		select {
		case <-user.isAlive:
			// 当前用户是活跃的，应重置计时器
			// 不做任何事情，为了激活select{}，使计时器重置
		case <-time.After(time.Second * 10):
			// 已经超时，强制踢下线
			user.SendMsg("超时未有动作，您已被强迫下线。")
			user.Offline()
			// 关闭连接
			conn.Close()
		}
	}
}
```
我们通过在 `handler()` 中放置一个 `for语句`，循环的检测是否超时。

如果客户端有发消息的动作，则会向 `user.isAlive` 写一个值，然后第二个 `case` 的计时器会被重置。

如果客户端没有发消息的动作，则时间一到，第二个 `case` 会被选中执行强踢下线的操作。

这样就差最后一步，我们得在 `massTexting()` 中，添加一条刷新活跃状态 `isAlive` 的语句。

```go
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
			conn.Close()
			return
		}
		// 提取用户的消息（去除\n）
		msg := string(buf[:n-1])
		args := strings.Split(msg, " ")
		// 针对msg 进行处理
		user.HandleMessage(args)

		// 刷新活跃状态
		user.isAlive <- true
	}
}
```

到此超时强踢的功能就写完了。整体的思路是：

- 为 User 结构体增加一个记录活跃状态的通道类型字段 `isAlive chan bool`；在处理客户端消息的时候刷新这个字段（即向这个通道发送一个 `true`）；
- 而在 `handler()` 的时候执行监听一个计时器，如果客户端活跃了计时器就会被拦截刷新，如果客户端一直没有活跃则计时器会计时结束，执行强踢。

## 编译运行
```shell
$ go build -o server server.go main.go user.go

$ ./server
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/443247278499.png)

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
            Name    string
            Addr    string
            C       chan string
            conn    net.Conn
            server  *Server
            isAlive chan bool
        }

        var count int

        func NewUser(conn net.Conn, server *Server) *User {
            count++
            user := &User{
                Name:    "user" + strconv.Itoa(count),
                Addr:    conn.RemoteAddr().String(),
                C:       make(chan string),
                conn:    conn,
                server:  server,
                isAlive: make(chan bool),
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
        func (u *User) HandleMessage(args []string) {
            switch args[0] {
            case "alluser":
                u.showOnlineUser()
            case "rename":
                u.rename(args[1])
            default:
                u.server.generateMsg(u, args[0])
            }
        }

        func (u *User) SendMsg(msg string) {
            u.conn.Write([]byte(msg))
        }

        // 打印所有在线用户
        func (u *User) showOnlineUser() {
            u.server.mapLock.Lock()
            userName := ""
            for _, user := range u.server.onlineMap {
                userName += "[ " + user.Name + " @ " + user.Addr + "]" + "\n"
            }
            u.server.mapLock.Unlock()
            u.SendMsg(userName)

        }

        func (u *User) rename(newName string) {
            _, ok := u.server.onlineMap[newName]
            if ok && u.Name != newName {
                u.SendMsg("当前用户名被使用。\n")
                return
            }
            if u.Name == newName {
                u.SendMsg("改了个寂寞\n")
                return
            }
            u.server.mapLock.Lock()
            delete(u.server.onlineMap, u.Name)
            u.server.onlineMap[newName] = u
            u.server.mapLock.Unlock()

            u.Name = newName
            u.SendMsg("修改成功\n")
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
            "strings"
            "sync"
            "time"
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
            fmt.Print("Connection Successfuly.\n")
            conn.Write([]byte("Connection Successfuly.\n"))

            user := NewUser(conn, s)
            user.Online()

            // 接收客户端发送的消息
            go s.massTexting(conn, user)

            // 当前广播阻塞
            for {
                select {
                case <-user.isAlive:
                    // 当前用户是活跃的，应重置计时器
                    // 不做任何事情，为了激活select{}，使计时器重置
                case <-time.After(time.Second * 10):
                    // 已经超时，强制踢下线
                    user.SendMsg("超时未有动作，您已被强迫下线。")
                    user.Offline()
                    // 关闭连接
                    conn.Close()
                }
            }
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
                    conn.Close()
                    return
                }
                // 提取用户的消息（去除\n）
                msg := string(buf[:n-1])
                args := strings.Split(msg, " ")
                // 针对msg 进行处理
                user.HandleMessage(args)

                user.isAlive <- true
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