# http
Golang 内置的HTTP服务端和客户端的实现。

## 导包
```go
import "net/http"
```

## HTTP协议

超文本传输协议 HyperText Transfer Protocol，所有的 www 文件都必须遵守这个标准。

## RESTful API

RESTful 是目前最流行的 API 设计规范，用于 web 数据接口的设计。
这里只做简要介绍。


基本的 HTTP/HTTPS 请求包括 GET、POST、HEAD、PUT、DELETE、CONNECT、OPTIONS、TRACE。

常用的有 GET、POST两种。
GET会将参数等明文传输，即在URL中就可以看得到请求的参数，被认为是不安全的，通常用来请求普通页面。
POST会将参数等包含在请求体中，通常用来向服务器提交数据。

### URL 设计

关于 URL 的设计规范主要有5条：

1. 动词+宾语
2. 动词覆盖
3. 宾语得是名词

```
# GET：读取（Read）
# POST：新建（Create）
# PUT：更新（Update）
# PATCH：更新（Update），通常是部分更新
# DELETE：删除（Delete）
```

## 客户端：发起HTTP请求



### 基本请求方法

http 包提供了 `Get()` 方法和 `Post()` 方法，可以很方便的发起 Get 请求和 Post 请求。但是其他的如 Put 请求，Delete 请求只能通过 `NewRequest()` 方法构造一个请求，然后自己提交请求的发送。



`NewRequest()`函数签名如下：

```go
func NewRequest(method, url string, body io.Reader) (*Request, error) 
```

参数说明：

- method：请求方法，值为 `http.MethodGet` 这类http包定义好的变量
- url：请求的目标地址
- body：

```go
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
    // 构造一个 Get 请求
	req, _ := http.NewRequest(http.MethodGet, "http://httpbin.org/get", nil)
    // 调用 DefaultClient.Do() 方法提交请求
    rsp, _ := http.DefaultClient.Do(req)
    // 读取响应体内容
	c, _ := ioutil.ReadAll(rsp.Body)
	defer rsp.Body.Close()
	// 打印响应体内容
	fmt.Println(string(c))
}
```

一共需要4步：

1. 调用 `http.NewRequest()` 构造一个 Get 请求。
2. 调用 `http.DefaultClient.Do()` 发起网络请求，也可以是自己构造的Client，再去调用 `Do()` 方法。
3. 调用 `ioutil.ReadAll()` 读取响应体内容，类型为 `[]byte`
4. 打印响应体内容



同样的姿势，我们可以接着发起 Post 请求，Put 请求

```go
// Post.go
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	req, _ := http.NewRequest(http.MethodPost, "http://httpbin.org/post", nil)
	rsp, _ := http.DefaultClient.Do(req)
	c, _ := ioutil.ReadAll(rsp.Body)
	defer rsp.Body.Close()

	fmt.Println(string(c))
}
```

```go
// Put.go
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	req, _ := http.NewRequest(http.MethodPut, "http://httpbin.org/put", nil)
	rsp, _ := http.DefaultClient.Do(req)
	c, _ := ioutil.ReadAll(rsp.Body)
	defer rsp.Body.Close()

	fmt.Println(string(c))
}
```

> http 包中对所有方法的定义
>
> ```go
> // Common HTTP methods.
> //
> // Unless otherwise noted, these are defined in RFC 7231 section 4.3.
> const (
> 	MethodGet     = "GET"
> 	MethodHead    = "HEAD"
> 	MethodPost    = "POST"
> 	MethodPut     = "PUT"
> 	MethodPatch   = "PATCH" // RFC 5789
> 	MethodDelete  = "DELETE"
> 	MethodConnect = "CONNECT"
> 	MethodOptions = "OPTIONS"
> 	MethodTrace   = "TRACE"
> )
> ```
>
> 在 构造请求 `NewRequest(method, url, body)` 时填入 method 参数时，可以填入 `http.Method+请求类型` ，也可以填入等号右边相应的字符串。



### http.Get()

除了上面构造请求的方式，http 包还提供两个方法 `Get()` 和 `Post()` 方便我们使用。

`Get()` 的函数签名如下：

```go
func Get(url string) (resp *Response, err error)
```

使用方法如下：

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    // 发起 HTTP Get 请求
    rsp, err := http.Get("https://www.boii.xyz")
    HandleErr(err)
    
    // 延迟关闭响应主体
    defer rsp.Body.Close()

    // 读取响应主体
    body, err := ioutil.ReadAll(rsp.Body)
    HandleErr(err)

    // 打印响应内容
    fmt.Println(string(body))
}
```
编译运行：
```bash
$ go run main.go

<!DOCTYPE html>
<html>
    <head hexo-theme="https://github.com/volantis-x/hexo-theme-volantis/tree/3.0.0">
        <meta charset="utf-8"><meta name="robots" content="index,follow">
        <meta name="renderer" content="webkit">
...
```

运行之后程序就会向 `https://www.boii.xyz` 这个网址发送请求，然后接收其响应内容，读取并打印。

浏览器的本质，就是这么一个程序，只不过浏览器能够渲染 HTML、CSS等等，让网页更加好看，体验更好。而我们的程序没有太多处理，只是简单的将返回内容打印出来而已。

在栗子中，总共分为4步：

1. 调用 `http.Get(url)` 发起 Get 请求
2. 调用 `ioutil.ReadAll(rsp.Body)` 读取响应体中的内容
3. 打印响应体内容
4. 调用 `rsp.Body.Close()` 关闭响应体

这样就能完成一个最基础的 Get 请求。



### http.Post()

```go
func Post(url, contentType string, body io.Reader) (resp *Response, err error)
```

使用方法如下：

```go
import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	// 发起 Post 请求
	rsp, _ := http.Post("http://httpbin.org", "", nil)
	// 延迟关闭响应体
    defer rsp.Body.Close()
    // 读取所有响应体内容
	c, _ := ioutil.ReadAll(rsp.Body)
    // 打印响应体内容
	fmt.Println(string(c))
}
```

### 设置请求查询参数

```go
package main

import (
    "net/http"
    "net/url"
    "fmt"
    "io/ioutil"
)

func main() {
    apiUrl ：= "http://127.0.0.1:8000/get"
    data := url.Value{"name": "Boii", "age": 18}
    
    u, err := url.ParseRequestURI(apiUrl)
    HandleErr(err)
    u.RawQuery = data.Encoding()

    // 发起 HTTP 请求
    resp, err := http.Get(u.String())
    HandleErr(err)
    // 延迟关闭响应体
    defer resp.Body.Close()

    // 读取响应体内容
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil { panic(err) }
    // 打印响应内容
    fmt.Println(string(b))

}
```

```go
// Server.go

func getHandler(w http.ResponseWriter, r *http.Request) {
    defer r.Body.Close()
    data := r.URL.Query()
    fmt.Println(data.Get("name"))
    fmt.Println(data.Get("age"))

    answer := `{"status": "ok"}`
    w.Write([]byte(answer))
}
```



## 服务端：接收HTTP请求

### 默认 Server

ListenAndServe 使用指定的监听地址和处理器启动一个 HTTP 服务端。处理器参数通常是 nil，表示采用包变量 `DefaultServeMux` 作为处理器。

`Handle` 和 `HandleFunc` 函数可以向 `DefaultServeMux` 添加处理器。



服务器 Demo：

```go
import (
	"net/http"
)

func main() {
    http.HandleFunc("/", sayHello)		// 设置访问路由
    http.ListenAndServe(":8080", nil)	// 设置监听端口并启动服务
}

// 请求处理函数
func sayHello(w http.ResponseWriter, r *http.Request) {
    fmt.Fprint(w, "Hello HTTP!")    
}
```

仅仅需要使用 `http.HandleFunc()` 和 `http.ListenAndServe()` 就可以启动一个 web 服务器实例。

这两个函数的签名如下：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func ListenAndServe(addr string, handler Handler) error
```

`HandleFunc()` 需要两个参数：

第一个参数 `pattern` 指明了请求路径，当设置为 `“/”` 时，我们可以通过 `http://host:port/` 直接访问；

第二个参数 `handler` 接受一个处理函数，即接收HTTP请求后如何处理的函数，该参数要求传递一个 无返回值、只有两个参数分别是 ResponseWriter 类型和 Request 类型 的函数。



`ListenAndServe()` 也需要两个参数：

第一个参数 `addr` 指明了本服务实例监听的是发送给哪个端口的HTTP请求，如栗子中指定了 `:8080`，我们可以通过 `http://host:8080` 来访问。

第二个参数 `handler` 接收一个实现了 `Handler` 接口的变量。在http包中，Handler 接口定义如下：

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

### 解析 url 数据









