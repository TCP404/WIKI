# IM4-用户业务封装

在前面[《IM3-消息群发功能》](./IM3-%E6%B6%88%E6%81%AF%E7%BE%A4%E5%8F%91%E5%8A%9F%E8%83%BD.md)的例子，User 的功能全都放在 Server 的方法中处理，这样导致 Server 的方法太过复杂,
所以我们需要将属于 User 的功能抽取出来封装好，然后再在 Server 中调用。

我们先看原先的 `User.go`：
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
除了 User 结构体，只有一个监听好友上线的的方法，而现在的 User 其实还具有 上线、离线、发消息的功能。

所以我们需要将 Server 中这三块那过来。

1. 第一步：在 `User.go` 中定义 上线、离线、发消息 的方法。

    ```go
    func (u *User) Online() {}

    func (u *User) Offline() {}

    func (u *User) HandleMessage() {}
    ```

    然后我们去看看这三个功能的具体实现是怎样的

    - 用户上线功能

        ```go
        // 将用户加入 onlinemap
        user := NewUser(conn)
        s.mapLock.Lock()
        s.onlineMap[user.Name] = user
        s.mapLock.Unlock()

        // 用户上线广播
        s.generateMsg(user, "已上线")
        ```

    - 用户离线功能

        ```go
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
        }
        ```

    - 用户群发功能

        ```go
        // 提取用户的消息（去除\n）
        msg := string(buf[:n-1])
        // 将得到的消息进行广播
        s.generateMsg(user, msg)
        ```


    用户上线功能的实现逻辑是将 user 实例加入到 在线用户列表，然后进行广播

    用户离线功能的实现逻辑是将 user 实例从 在线用户列表中删除，然后进行广播

    用户群发功能的实现逻辑是将客户端读取到的消息进行广播

    这三个功能都需要用到广播，而广播是 Server 的功能，所以我们需要对 User 结构体进行改造，让其增加一个 `*Server` 字段；

    在实例化 User 的时候，也要将所在的 Server 一并传过来。

2. 第二步：改造 User 结构体并封装

    ```go
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
    ```

    现在我们就可以把用户上线的具体实现那过来放在 `user.go` 中的 `Online()` 里了：

    ```go
    // Online 用户上线功能
    func (u *User) Online() {
	    // 用户上线，将用户加入 onlinemap
	    u.server.mapLock.Lock()
	    u.server.onlineMap[u.Name] = u
	    u.server.mapLock.Unlock()

	    // 用户上线广播
	    u.server.generateMsg(u, "已上线")
    }
    ```

    用户离线的功能类似：
    ```go
    // Offline 用户下线功能
    func (u *User) Offline() {
	    // 用户下线，将用户从 onlineMap 中移除
	    u.server.mapLock.Lock()
	    delete(u.server.onlineMap, u.Name)
	    u.server.mapLock.Unlock()

	    // 用户下线广播
	    u.server.generateMsg(u, "已离线")
    }
    ```

    用户群发功能更简单，但是需要加上参数：
    ```go
    // HandleMessage 用户处理消息功能
    func (u *User) HandleMessage(msg string) {
	    u.server.generateMsg(u, msg)
    }
    ```

3. 第三步：修改 Server 中用户功能的实现，改为调用上面三个方法

    ```go
    func (s *Server) handler(conn net.Conn) {
	    fmt.Println("Connection Successfuly.\n")
	    conn.Write([]byte("Connection Successfuly.\n"))

	    // 将用户加入 onlinemap
	    // user := NewUser(conn)
	    // s.mapLock.Lock()
	    // s.onlineMap[user.Name] = user
	    // s.mapLock.Unlock()

	    // 用户上线广播
	    s.generateMsg(user, "已上线")

        /* 改为 */
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
			    // s.generateMsg(user, "已离线")
            /* 改为 */
            user.Offline()
			    return
		    }
		    // 提取用户的消息（去除\n）
		    msg := string(buf[:n-1])
		    // 将得到的消息进行广播
		    // s.generateMsg(user, msg)
            /* 改为 */
            user.HandleMessage(msg)
	    }
    }
    ```

## 编译运行
```shell
$ go build -o server server.go main.go user.go

$ ./server
```

![运行效果](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/1879953228493.png)


## 完整代码

??? example

    === "user,go"

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

            u.server.generateMsg(u, msg)
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