# 中间件

用在 `客户端` 与 `服务端` 之间的插件，我们称之为 `中间件`。

![中间件示意图](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/5592618688329.png)

中间件是一个概念，其实质就是一些遵循一定规范的函数，也称做 `钩子函数(Hook)`。

中间件适合处理一些公共的业务逻辑，比如登录认证、权限校验、数据分页、记录日志、耗时统计等。

## 定义中间件

定义一个中间件，其实就是定义一个 `gin.HandlerFunc`

这个 `HandlerFunc` 其实是一个函数，例如前面我们已经写了很多次的 GET请求、POST请求，第一个参数为路由路径，第二个参数写的就是这种格式的处理函数。

```go
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes
```

对于这个 `HandlerFunc`，源码中是这样定义的：

```go title="github.com/gin-gonic/gin/gin.go"
// HandlerFunc defines the handler used by gin middleware as return value.
type HandlerFunc func(*Context)
```

形参列表只有一个 `*gin.Context` 类型的参数，没有返回值。

这样的函数就是一个 `HandlerFunc`，就可以作为一个中间件。

eg：定义一个计时中间件

```go
func timer(c *gin.Context) {
    start := time.Now()
    c.Next()
    end   := time.Since(start)

    fmt.Println(end)
}
```
简简单单，上面的栗子就已经定义了一个名为 timer 的中间件，用于计时。

这里的 `c.Next()` 是主动调用后面的 `HandlerFunc`，它还有一个兄弟：`c.Abort()`，这个我们放在后面讲。

## 局部注册中间件

使用中间件也很简单，我们在接收请求的时候，即编写 `Handler()`、或者 `GET()` 这些方法时，最后一个参数就可以传入一个或多个 `HandlerFunc`。

eg:
```go
func sleep(c *gin.Context) {
    start := time.Now()
    time.Sleep(time.Second)    // 睡眠一秒
    end   := time.Since(start)

    fmt.Println("sleep middleware: ", end)
}

func indexHandler(c *gin.Context) {
    fmt.Println("Index handler")
    c.JSON(http.StatusOK, gin.H{ "msg": "Here is index." })
}

func main() {
    r := gin.Default()

    r.GET("/", sleep, indexHandler)    // 局部注册中间件到此路由

    r.Run(":9090")
}
```
输出：
![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/835062854610.png)

上面的栗子中可以看出，局部注册就是把中间件写在需要使用该中间件的路由中。

这里 `timer` 和 `indexHandler` 本质上都是一样的，但是被调用顺序不同。

客户端请求 `/` 这个路径时，会先调用 `timer` 这个中间件，等它执行完后再去执行 `indexHandler`。

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/3420038008778.png)


## 全局注册中间件

全局注册中间件，会使得该作用域下所有路由都会执行对应的中间件。注册时使用`Use()` 方法。

啥意思呢？

就是说，假设你定义了 `r := gin.Default()`，

然后你写 `r.Use(timer)`，这样你的网站所有的路由都会执行 timer 这个中间件。

如果你是用 `userG := r.Group("/user")`，然后写 `userG.Use(timer)`，这样 userG 这个路由组下所有路由都会执行 timer 这个中间件

而组外的其他路由并不一定会执行 timer。


!!! example
    ```go
    // 定义一个中间件
    func timer(c *gin.Context) {
        start := time.Now()
        c.Next()
        end := time.Since(start)
        fmt.Println(end)
    }

    func main() {
        r := gin.Default()

        userG := r.Group("/user")    // 定义一个路由组 userG
        userG.Use(timer)             // 绑定 timer 中间件
        {
            // 以下两个路由会执行 timer 中间件
            userG.POST("/register", Register)
            userG.POST("/login", Login)
        }

        // 定义一个路由组 boogG
        bookG := r.Group("/book")
        {
            // 以下两个路由不会执行 timer 中间件
            bookG.GET("/search/:id", SearchBook)
            bookG.DELETE("/remove/:id", RemoveBook)
        }

        r.Run(":9090")
    }
    ...
    ```

    ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1537822060392.png)

    可以清楚看到，请求 /user 组下的路径时，调用了 timer 打印了时间，而请求 /book 组时没有。


## 中间链（职责链模式）

### c.Next()
上面的栗子中，我们使用到了 `c.Next()` 函数，这会改变程序的执行流。

中间件里没有使用到 `c.Next()` 的时候，其执行流如下

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/4472457420778.png)

没有 `c.Next()` 的时候会执行完前一个中间件，就执行下一个中间件。


而在中间件中使用 `c.Next()` 时则会改变其执行流。

其实就是将下一个中间件的执行时机提前了，但是下一个中间件执行完以后还会回到当前的中间件。

其执行流如下图：

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/77417672863.png)


???+ example

    ```go
    // 中间件1
    func m1(c *gin.Context) {
        fmt.Println("m1 in...")
        c.Next()
        fmt.Println("m1 out...")
    }

    // 中间件2
    func m2(c *gin.Context) {
        fmt.Println("m2 in...")
        c.Next()
        fmt.Println("m2 out...")
    }

    // 首页处理函数
    func indexHandler(c *gin.Context) {
        fmt.Println("Index handler in...")

        c.JSON(http.StatusOK, gin.H{
            "msg": "Here is index.",
        })
        c.Next()
        fmt.Println("Index handler out...")
    }


    func main() {
        r := gin.Default()

        r.GET("/", m1, m2, indexHandler)    // 依次局部注册三个中间件

        r.Run(":9090")
    }
    ```

    ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/2607899532868.png)


    三个 `HandlerFunc` 的注册顺序依次是 `m1、m2、indexHandler`

    每个都调用了 `c.Next()`，

    所以他们的执行流应该是：

    ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/3247650618373.png)


### c.Abort()

`c.Abort()` 是 `c.Next()` 的兄弟。

`c.Next()` 调用下一个 HandlerFunc;
`c.Abort()` 阻止调用下一个 HandlerFunc。一旦阻止了，则后面不管有多少个 HandlerFunc，它们都没有机会执行了。

???+ example

    ```go
    // 中间件1
    func m1(c *gin.Context) {
        fmt.Println("m1 in...")
        c.Next()
        fmt.Println("m1 out...")
    }

    // 中间件2
    func m2(c *gin.Context) {
        fmt.Println("m2 in...")
        c.Abort()    // 阻止调用后面的 HandlerFunc
        fmt.Println("m2 out...")
    }

    // 中间件3
    func m3(c *gin.Context) {
        fmt.Println("m3 in...")
        c.Next()
        fmt.Println("m3 out...")
    }

    // 首页处理函数
    func indexHandler(c *gin.Context) {
        fmt.Println("Index handler in...")

        c.JSON(http.StatusOK, gin.H{
            "msg": "Here is index.",
        })
        c.Next()
        fmt.Println("Index handler out...")
    }


    func main() {
        r := gin.Default()

        r.GET("/", m1, m2, m3, indexHandler)    // 依次局部注册四个中间件

        r.Run(":9090")
    }
    ```

    ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/2547203825659.png)

    在 `m2` 的时候调用了 `c.Abort()`，所以后面的 `m3、indexHandler` 都没有机会执行了。

## 中间件之间数据传递

中间件之间要传递数据，可以通过 `c.Set(key, value)` 的方式发送，在获取的地方用 `c.Get(key)` 的方式获取。

获取的方法有非常多，如下图：

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/4093224325543.png)

在这里的获取要注意执行流的问题，如果第2个中间件执行 `c.Set()`，但是在第1个中间件就执行了 `c.Get()`，那会什么都拿不到。

???+ example
    ```go
    // 中间件1
    func m1(c *gin.Context) {
        fmt.Println("m1 in...")
        c.Set("k", 123)    // 设置数据
        c.Next()
        fmt.Println("m1 out...")
    }

    // 首页处理函数
    func indexHandler(c *gin.Context) {
        fmt.Println("Index handler in...")

        fmt.Println(c.MustGet("k"))    // 获取数据

        c.JSON(http.StatusOK, gin.H{
            "msg": "Here is index.",
        })

        fmt.Println("Index handler out...")
    }


    func main() {
        r := gin.Default()

        r.GET("/", m1, indexHandler)

        r.Run(":9090")
    }
    ```

    ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1886277063287.png)

## 默认中间件

`gin.Default()` 默认使用了 `Logger` 和 `Recovery` 中间件，其中：

`Logger中间件` 将日志写入 `gin.DefaultWriter`，即使配置了 `GIN_MODE=release`。

`Recovery中间件` 会 recover 任何 panic。如果有panic的话，会写入500响应码。

如果不想使用上面两个默认的中间件，可以使用 `gin.New()` 新建一个没有任何默认中间件的路由。

## 并发注意事项

gin中间件中使用goroutine

当在中间件或 `handler` 中启动新的 `goroutine` 时，不能使用原始的上下文`（c *gin.Context）`，必须使用其只读副本`（c.Copy()）`。

```go
func m1(c *gin.Context) {
    start := time.Now()
    end := time.Since(start)
    c.Set("timer", end)

    // go func(c)         // 错误！！！
    go func(c *gin.Context) { // 只能使用上下文的拷贝，不能使用原本的上下文
        fmt.Println("Other goroutine: ", c.MustGet("timer"))
    }(c.Copy())

}

func indexHandler(c *gin.Context) {
    timer := c.MustGet("timer")
    fmt.Println("indexHandler: ", timer)
}

func main() {
    r := gin.Default()

    r.GET("/", indexHandler)

    r.Run(":9090")
}
```

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1064955184484.png)

