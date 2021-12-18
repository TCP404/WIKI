# 接收请求

作为一个服务器，最基本的任务就是接收请求，然后是返回结果。

客户端发来的请求可以有多种，如 GET、POST、PUT、DELETE。
最基本的就是 GET、POST 两种，不过这里还是建议遵循 RESTful 规范。

## 基本处理：Handle()
gin 接收请求有一种最基本的方法，使用 `Handle()` 方法。

其方法签名如下：
```go
func (group *RouterGroup) Handle(httpMethod, relativePath string, handlers ...HandlerFunc) IRoutes;
```
其接受的第一个参数为 HTTP 请求方式，第二个为路由路径，第三个为该请求对应的处理函数。

???+ example "最基本的栗子，接收一个 GET 请求"

    ```go
    func main() {
        app := gin.Default()

        app.Handle("GET", "/hello", func(c *gin.Context) {
            c.JSON(http.StatusOK, gin.H{
                "msg": "Hello!",
            })
        })

        app.Run(":9090")
    }
    ```

    第4行调用了 `Handle()`，请求方式为 GET 方法，路由路径为 `/hello`，处理方法是返回一串JSON数据。

    浏览器访问 `http://localhost:9090/hello` 可得到下面的结果

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/957715546196.png)


**接收 POST、PUT 等方式的请求**

通过 `Handle()` 第一个参数我们可以接收不同方式的请求，例如 POST

```go
func main() {

    r := gin.Default()

    // http://127.0.0.1:9090/login
    r.Handle("POST", "/login", func(c *gin.Context) {    // 处理 POST 请求
        user := c.PostForm("user")    // 获取表单参数
        pass := c.PostForm("pass")    // 获取表单参数

        // 处理业务逻辑
        if user == "Boii" && pass == "123" {
            c.JSON(http.StatusOK, gin.H{
                "code": 2001,
                "msg":  "登录成功",
            })
            return
        }

        c.YAML(http.StatusOK, gin.H{
            "code": 4001,
            "msg":  "登录失败",
        })
    })

    r.Run(":9090")
}
```

通过 Postman 可以设定表单参数然后发起请求，我们用gin写的代码就可以接收到一个 POST 请求，并进行处理。
![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/840022092674.png)


可以看到， Postman 中设置了两个参数 `user` 和 `pass`，向 `http://localhost:9090/login` 发清请求。

关于获取参数后面会详解。


## 快捷方式：gin.GET()、gin.POST()...

像 GET、POST、PUT、DELETE 等这些 HTTP方式的处理函数会很常用，所以 gin 提供了一系列方法供我们更方便的使用。
??? note "源码"

    ```go
    // POST is a shortcut for router.Handle("POST", path, handle).
    func (group *RouterGroup) POST(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodPost, relativePath, handlers)
    }

    // GET is a shortcut for router.Handle("GET", path, handle).
    func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodGet, relativePath, handlers)
    }

    // DELETE is a shortcut for router.Handle("DELETE", path, handle).
    func (group *RouterGroup) DELETE(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodDelete, relativePath, handlers)
    }

    // PATCH is a shortcut for router.Handle("PATCH", path, handle).
    func (group *RouterGroup) PATCH(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodPatch, relativePath, handlers)
    }

    // PUT is a shortcut for router.Handle("PUT", path, handle).
    func (group *RouterGroup) PUT(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodPut, relativePath, handlers)
    }

    // OPTIONS is a shortcut for router.Handle("OPTIONS", path, handle).
    func (group *RouterGroup) OPTIONS(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodOptions, relativePath, handlers)
    }

    // HEAD is a shortcut for router.Handle("HEAD", path, handle).
    func (group *RouterGroup) HEAD(relativePath string, handlers ...HandlerFunc) IRoutes {
        return group.handle(http.MethodHead, relativePath, handlers)
    }
    ```
通过源码可以看出，gin 很贴心的做了一层封装，提供了对应请求方式的 `shortcut`，使开发者更方便的调用。
参数上只需要传递 `路由路径` 和 `处理函数` 即可，这些“快捷方式”会帮我们调用 `handle()`。

eg：

```go hl_lines="5"
func main() {
    r := gin.Default()

    // http://127.0.0.1:9090/hello
    r.GET("/hello", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{ "msg": "Hello"})
    })
    r.Run(":9090")
}
```

```go hl_lines="5"
func main() {
    r := gin.Default()

    // http://127.0.0.1:9090/login
    r.POST("/login", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{ "msg": "Hello"})
    })
    r.Run(":9090")
}
```

### 404 页面

```go hl_lines="4"
func main() {
    r := gin.Default()

    r.NoRoute(func(c *gin.Context) {
        c.HTML(http.StatusNotFound, "views/404.html", nil)
    })

    r.Run(":9090")
}
```
不需要指定路径，只要用户访问不存在的路径，就会执行此路由。

## 处理请求参数

### 获取 GET 请求的参数

GET 请求的参数会显式的携带在URL中。

可以通过 `gin.Context.Query()` 或 `gin.Context.DefaultQuery()` 两种方法获取。

他们的区别是 `DefaultQuery()` 需要填入默认值。

```go
func (c *Context) Query(key string) string;
func (c *Context) DefaultQuery(key, defaultValue string) string;
```

???+ example
    ```go
    func main() {
        r := gin.Default()

        r.GET("/hello", func(c *gin.Context) {
            name := c.Query("name")
            age := c.DefaultQuery("age")
            c.JSON(200, gin.H{"msg": "Hello " + name + ", you're " + age})
        })

        r.Run(":9090")
    }
    ```
    ![age 给了值](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/2988503917018.png)

    ![age 没给值](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/5990713787204.png)


### 获取 POST 表单参数

POST 请求的参数不会显式的携带在URL中，而是包裹在请求体 Body 中，是通过表单提交的。

可以通过 `gin.Context.PostForm()` 或 `gin.Context.DefaultPostForm()` 两种方法获取。

同样他们的区别仅在于一个需要填入默认值。

```go
func (c *Context) PostForm(key string) string;
func (c *Context) DefaultPostForm(key, defaultValue string) string;
```

???+ example
    ```go
    func main() {
        r := gin.Default()

        // http://127.0.0.1:9090/login
        r.POST("/login", func(c *gin.Context) {
            user := c.DefaultPostForm("user", "admin")
            pass := c.PostForm("pass")
            if user == "Boii" && pass == "123" {
                c.JSON(http.StatusOK, gin.H{
                    "code": 2001,
                    "msg":  "登录成功",
                })
                return
            }
            c.YAML(http.StatusOK, gin.H{
                "code": 4001,
                "msg":  "登录失败",
            })
        })
        r.Run(":9090")
    }
    ```

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/783508891344.png)


### 处理其他格式的请求参数
除了URL中的参数、表单提交的参数，在客户端发起 HTTP 请求时还可以使用 JSON、XML、YAML 等格式。

获取这些格式的参数，可以通过 `gin.Context.Bind()` 来获取，

Bind() 有一系列的方法：
```go
func (c *Context) Bind(obj interface{}) error;
func (c *Context) BindJSON(obj interface{}) error;
func (c *Context) BindXML(obj interface{}) error;
func (c *Context) BindQuery(obj interface{}) error;
func (c *Context) BindYAML(obj interface{}) error;
func (c *Context) BindHeader(obj interface{}) error;
func (c *Context) BindUri(obj interface{}) error;
func (c *Context) MustBindWith(obj interface{}, b binding.Binding) error;
```

显然，要解析请求中什么类型的数据就调用什么方法，参数 `obj` 需要我们传入一个结构体变量，用于装载解析后的数据。

结构体每个字段都需要有 tag，否则会解析失败。

??? example
    ```go
    type UserInfo struct {
        User string
        Pass string
    }

    var u UserInfo

    func main() {
        r := gin.Default()

        r.POST("/login", func(c *gin.Context) {
            c.BindJSON(&u)
            c.JSON(200, gin.H{
                "user": u.User,
                "pass": u.Pass,
            })
        })

        r.Run(":9090")
    }
    ```

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/320000449262.png)

    这个例子可以解析 HTTP 请求中通过 JSON 携带的数据。

#### Bind()
如果想根据请求中的 `content-type` 属性来选择，可以使用 `Bind()` 方法，该方法会自动选择

??? note "源码"

    ```go
    // gin 源码
    // context.go

    // Bind checks the Content-Type to select a binding engine automatically,
    // Depending the "Content-Type" header different bindings are used:
    //     "application/json" --> JSON binding
    //     "application/xml"  --> XML binding
    // otherwise --> returns an error.
    // It parses the request's body as JSON if Content-Type == "application/json" using JSON or XML as a JSON input.
    // It decodes the json payload into the struct specified as a pointer.
    // It writes a 400 error and sets Content-Type header "text/plain" in the response if input is not valid.
    func (c *Context) Bind(obj interface{}) error {
        b := binding.Default(c.Request.Method, c.ContentType())
        return c.MustBindWith(obj, b)
    }

    // gin 源码
    // binding.go

    // Default returns the appropriate Binding instance based on the HTTP method
    // and the content type.
    func Default(method, contentType string) Binding {
        if method == http.MethodGet {
            return Form
        }

        switch contentType {
        case MIMEJSON:
            return JSON
        case MIMEXML, MIMEXML2:
            return XML
        case MIMEPROTOBUF:
            return ProtoBuf
        case MIMEMSGPACK, MIMEMSGPACK2:
            return MsgPack
        case MIMEYAML:
            return YAML
        case MIMEMultipartPOSTForm:
            return FormMultipart
        default: // case MIMEPOSTForm:
            return Form
        }
    }
    ```

可以看到，context 中的 `Bind()` 方法调用了 `binding.Default()` 方法;

在 `binding.Default()` 方法中，如果请求方式是 GET 则返回 Form，否则的话，根据请求中的 `content-type` 属性返回对应的格式。

接着会调用 `#!go gin.Context.MustBindWith(obj, b) -> gin.Context.ShouldBindWith(obj, b) -> b.Bind()`；

到 `b.Bind()` 这里的时候，会根据 `b` 的类型调用各自的 `Bind()` 方法。

例如 JSON 的话会调用 `jsonBinding.Bind()` 方法，然后在里面调用 `decodeJSON()` 方法。


![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/638857661664.png)



### 一次解析多个参数
上面登录的栗子中，我们简简单单的获取了两个参数，所以可以用两次 `PostForm()` 方法。

但是当参数多起来的时候，这种方式并不是很好。好在 gin 还提供另外的方法方便我们一次解析多个参数。

要一次解析多个参数，我们需要一个 **结构体** 来装载这些参数的值，然后将结构体传给 `gin.Context.Bind()` 、 `gin.Context.ShouldBind()` 、 `gin.Context.ShouldBindQuery()` 等方法。

```go
func (c *Context) ShouldBind(obj interface{}) error;
func (c *Context) ShouldBindQuery(obj interface{}) error;
```

要注意的是，我们定义的这个结构体每个字段都需要有 tag，否则会解析失败。

??? example "例如"
    ```go
    func main() {
        r := gin.Default()

        type UserInfo struct {
            User string `form:"user"`
            Pass string `form:"pass"`
        }
        // http://127.0.0.1:9090/login
        r.POST("/login", func(c *gin.Context) {
            var u UserInfo
            c.ShouldBind(&u)
            c.JSON(http.StatusOK, gin.H{
                "code": 2001,
                "msg":  "登录成功",
                "user": u.User,
                "pass": u.Pass,
            })
        })
        r.Run(":9090")
    }
    ```

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/5790709568721.png)

    效果是一样的。


## 解析路由路径中的参数

### gin.Context.Param()
路由路径有时候并不固定，而是根据实际情况变化的。

例如下面的栗子，不同的用户有不同的ID，要获取这个ID，需要用到 `gin.Context.Param()` 方法，填入冒号通配符后面的变量 `id` 即可。

在使用 PUT、DELETE 等请求方法的时候是修改、删除某条记录的目的，这需要在路径中指定一个关键值（如 `id`），这里就可以用像下面这样去解析。

??? example

    ```go
    func main() {
        r := gin.Default()

        // http://127.0.0.1:9090/user/10086
        r.PUT("/user/:id", func(c *gin.Context) {
            userId := c.Param("id")
            c.JSON(http.StatusOK, gin.H{
                "code": 2001,
                "msg":  "ID is " + userId,
            })
        })
        r.Run(":9090")
    }
    ```


    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/3683906512884.png)


### gin.Context.Params

除了`Param()`，还可以使用 `Params.Get()`、`Params.ByName()` 去获取想要的参数。

??? example

    ```go
    func main() {
        r := gin.Default()

        // http://127.0.0.1:9090/Boii/18/1234567
        r.GET("/params/:name/:age/:tele", func(c *gin.Context) {
            params := c.Params

            name, _ := params.Get("name")
            age     := params.ByName("age")
            tele    := c.Param("tele")

            c.JSON(http.StatusOK, gin.H{
                "name": name,
                "age":  age,
                "tele": tele,
            })
        })

        r.Run(":9090")
    }
    ```

    ![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/1504620189883.png)

