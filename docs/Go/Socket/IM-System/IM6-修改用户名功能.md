# IM6-修改用户名

修改用户名思路基本和 [《IM5-在线用户查询》](./IM5-%E5%9C%A8%E7%BA%BF%E7%94%A8%E6%88%B7%E6%9F%A5%E8%AF%A2%E5%8A%9F%E8%83%BD.md) 中的思路一样：

首先要读取客户端发来的内容，根据内容进入不同的分支执行不同的处理。

这里我们先约定修改用户名的指令为：`rename 新用户名`

在 Server 的 `massTexting()` 读取客户端输入之后将内容进行分割，然后传递给 `user.HandleMessage()`

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
			return
		}
		// 提取用户的消息（去除\n）
		msg := string(buf[:n-1])
		args := strings.Split(msg, " ")
		// 针对msg 进行处理
		user.HandleMessage(args)
	}
}
```

注意 `strings.Split()` 返回的是字符串切片，所以我们还需要对 `HandleMessage()` 的参数进行修改。

=== "修改前"

    ```go
    // HandleMessage 用户处理消息功能
    func (u *User) HandleMessage(msg string) {
        switch msg {
        case "alluser":		// 查询当前在线用户
            u.showOnlineUser()
        default:
            u.server.generateMsg(u, msg)
        }
    }
    ```

=== "修改后"

    ```go
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
    ```


将形参改成了字符串切片，然后判断切片中的第一个元素来进入不同的分支。

这里增加了 `"rename"` 的分支，我们将修改用户名的具体实现放到 `rename()` 中。

```go
func (u *User) rename(newName string) {

	_, ok := u.server.onlineMap[newName]

	if ok && u.Name != newName {
		u.sendMsg("当前用户名已被使用。\n")
		return
	}

	if u.Name == newName {
		u.sendMsg("改了个寂寞\n")
        return
	}
    // 改名逻辑，先删后加
	u.server.mapLock.Lock()
	delete(u.server.onlineMap, u.Name)
	u.server.onlineMap[newName] = u
	u.server.mapLock.Unlock()

	u.Name = newName
	u.sendMsg("修改成功\n")
}
```
`rename()` 的实现逻辑是：

- 先判断在线用户列表中是否存在 `newName` ，如果存在且不是当前用户的名字，则向客户端打印当该用户名被使用了。
- 如果不存在，则先删除在线用户列表中旧的用户名键值对 `onlineMap[u.Name]`，然后增加一条新的键值对，并修改当前 `user` 实例的 `Name` 字段。


## 编译运行

```shell
$ go build -o server main.go server.go user.go

$ ./server
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/1534027720860.png)

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

        func (u *User) sendMsg(msg string) {
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
            u.sendMsg(userName)

        }

        func (u *User) rename(newName string) {
            _, ok := u.server.onlineMap[newName]
            if ok && u.Name != newName {
                u.sendMsg("当前用户名被使用。\n")
                return
            }

            if u.Name == newName {
                u.sendMsg("改了个寂寞\n")
                return
            }

            u.server.mapLock.Lock()
            delete(u.server.onlineMap, u.Name)
            u.server.onlineMap[newName] = u
            u.server.mapLock.Unlock()

            u.Name = newName
            u.sendMsg("修改成功\n")
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
                args := strings.Split(msg, " ")
                // 针对msg 进行处理
                user.HandleMessage(args)
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