# IM2-实现用户上线广播

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/3976583799263.png)


实现用户上线广播功能，类似于 QQ 的好友上线提醒功能。

当客户端发来一个请求，服务端接收以后会交给 handler 处理；

handler 会先将这个用户 user 加入 onlineMap 中，然后告知广播 broadcast 进行播送；

而用户 user 在实例化时就会启动一个 goroutine 监听好友上线消息这个 channel，一旦有消息就 write 出来。

## 构建 User

首先我们先建立一个 User 结构体，包含用户的名字、地址、监听好友上线消息的通道、连接，暂时就负责一件事：监听好友上线消息。

```go
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
```

User 提供了一个工厂函数用于实例化，并在实例化的时候开启一个 goroutine `go user.listenMessage()` 用于监听好友上线消息。

监听也简单，就是循环从通道中读取，一旦读取到消息就 write 到客户端。

```go
func (u *User) listenMessage() {
	for {
		msg := <-u.C    // 从通道中读取
		u.conn.Write([]byte(msg + "\n"))
	}
}
```

## 改造 Server

### Server 结构体
接下来我们需要对 Server 结构体进行改造。

首先得包含一个 map 保存着已上线的用户，顺带配把锁可以在写入的时候使用，然后再加一个通道 用于存放用户上线的 message

```go
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
```

### handler 处理
我们这个服务的流程是：Client 发来请求，Server 接收 -> handler 处理。

所以现在要从 handler 开始处理。

```go
// v1版
func (s *Server) handler(conn net.Conn) {
    fmt.Println("Connection Successfuly.")                   // 服务端打印连接成功信息
    conn.Write([]byte("Connection Successfuly."))    // 客户端打印连接成功信息
}
```

建立连接之后分别在 服务端 和 客户端 都打印了连接成功的信息。

接着我们得实例化一个 User ，并加入的到 onlineMap 中，然后让广播播送这个用户上线的消息。

```go
// v2版
func (s *Server) handler(conn net.Conn) {
	fmt.Println("Connection Successfuly.\n")
	conn.Write([]byte("Connection Successfuly.\n"))

	// 将用户加入 onlinemap
	user := NewUser(conn)
	s.mapLock.Lock()
	s.onlineMap[user.Name] = user
	s.mapLock.Unlock()

	// 用户上线广播
	s.generateMsg(user, "Online")
	// 阻塞当前广播
	select {}
}
```
`NewUser()` 之后会返回一个 User 实例，同时该实例开启了一个 `goroutine` 监听好友上线消息；

接着由于 map 不是线程安全的，所以要先上锁，把 当前 user 加入到 onlineMap ，然后解锁；

最后调用 `generateMsg()` 生成广播消息内容，并放入消息通道。

```go
// 消息生成器
func (s *Server) generateMsg(user *User, msg string) {
	sendMsg := "[" + user.Addr + "]" + user.Name + ": " + msg + "\n"
	s.message <- sendMsg
}
```
消息生成器，负责生成一条广播消息，然后放到 消息通道 `s.message`中；


### broadcast 广播

广播负责从 消息通道中取出消息，然后遍历在线用户列表 onlineMap ，把消息发到每个 用户的好友上线消息通道 `user.C` 中。
```go
// 监听消息通道，然后进行播送
func (s *Server) broadcast() {
	for {
		msg := <-s.message
		s.mapLock.Lock()
		for _, user := range s.onlineMap {
			user.C <- msg
		}
		s.mapLock.Unlock()
	}
}
```


而用户 user 是有一个 goroutine 一直在监听这个通道的：
```go
func (u *User) listenMessage() {
	for {
		msg := <-u.C    // 从通道中读取
		u.conn.Write([]byte(msg + "\n"))
	}
}
```
所以一旦有消息就会打印到客户端中,这样就实现了好友上线广播功能了。

最后在 `Start()` 中开启一个 `goroutine` 来监听消息通道并发送消息，也就是调用`广播`这个方法
```go
func (s *Server) Start() {
    ...
	defer listener.Close()
	fmt.Println("=> Server is listening on :", address)

	// 启动监听msg 的 goroutine
	go s.broadcast()

	...

		go s.handler(conn)
	}
}
```


## 编译运行

```shell
$ go build -o server server.go user.go main.go

$ ./server
```
![运行效果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/2521541188493.png)


## 总结

在原先服务架构上，增加一个 User 负责监听好友上线消息；

Server 中加入 在线用户列表 和 消息通道

在 handler 处理的时候，实例化一个 user 并加入在线用户列表，然后进行广播

在广播的时候，先生成广播消息，加入到消息通道中；然后从通道中读取消息，遍历在线用户列表，逐个发送好友上线的消息。

user 负责监听好友上线消息的 goroutine 收到消息后就会向客户端打印。

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

    === "server.go"

        ```go
        // server.go
        package main

        import (
            "fmt"
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

            // 启动监听msg 的 goroutine
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
            s.generateMsg(user, "Online")
            // 当前广播阻塞
            select {}
        }

        // 消息生成器
        func (s *Server) generateMsg(user *User, msg string) {
            sendMsg := "[" + user.Addr + "]" + user.Name + ": " + msg + "\n"
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