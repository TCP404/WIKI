# IM8-私聊功能

## 约定
和改名类似，我们要先约定一个格式：`-> USERNAME CONTENT`.

先是一个 `->` 符号，然后第二个参数是用户名，第三个参数是私聊的内容。

客户端输入 `-> user1 Hello` 之后，内容会在 `user.go` 中的 `HandleMessage()` 方法处理。

原本的 `user.go - HandleMessage()`：
```go
// HandleMessage 用户处理消息功能
func (u *User) HandleMessage(args []string) {
	switch args[0] {
	case "alluser":
		u.showOnlineUser()
	case "rename": // rename NEWNAME
		u.rename(args[1])
	default:
		u.server.generateMsg(u, args[0])
	}
}
```

现在我们给它增加一个 case，当第一个参数是 `->` 符号时，调用`u.privateChat()` 方法。
```go
// HandleMessage 用户处理消息功能
func (u *User) HandleMessage(args []string) {
	switch args[0] {
	case "alluser":
		u.showOnlineUser()
	case "rename": // rename NEWNAME
		u.rename(args[1])
	case "->": // -> USERNAME CONTENT
		u.privateChat(args[1], args[2:])
	default:
		u.server.generateMsg(u, args[0])
	}
}
```


## 具体实现
```go
func (u *User) privateChat(username string, chatContent []string) {

	if username == "" || len(chatContent) == 0 {
		u.SendMsg("消息格式不正确：-> USERNAME MESSAGE")
		return
	}
	// 根据用户名得到对方的 user 对象
	remoteUser, ok := u.server.onlineMap[username]
	if !ok {
		u.SendMsg("输入的用户名不正确或可能对方不在线！")
		return
	}
	// 拼接消息内容，通过对方的 user 对象将消息内容发送过去
	content := ""
	for _, s := range chatContent {
		content += s + " "
	}
	remoteUser.SendMsg(u.Name + " -> " + content + "\n")
}
```
实现私聊功能很简单，从在线用户列表中获得对方用户 user 对象，拼接好消息内容，然后通过对方用户 user 对象调用 `SendMsg()` 即可。

## 编译运行
```go
$ go build -o server server.go main.go user.go

$ ./server
```

![运行效果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/5701416248572.png)

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
            case "rename": // rename NEWNAME
                u.rename(args[1])
            case "->": // -> USERNAME CONTENT
                u.privateChat(args[1], args[2:])
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

        func (u *User) privateChat(username string, chatContent []string) {

            if username == "" || len(chatContent) == 0 {
                u.SendMsg("消息格式不正确：-> USERNAME MESSAGE")
                return
            }
            // 根据用户名得到对方的 user 对象
            remoteUser, ok := u.server.onlineMap[username]
            if !ok {
                u.SendMsg("输入的用户名不正确或可能对方不在线！")
                return
            }
            // 拼接消息内容，通过对方的 user 对象将消息内容发送过去
            content := ""
            for _, s := range chatContent {
                content += s + " "
            }
            remoteUser.SendMsg(u.Name + " -> " + content + "\n")
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
                case <-time.After(time.Second * 60):
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

