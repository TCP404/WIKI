# 上传下载和重定向

文件上传是后端开发中很常见的需求。

文件上传的原理无非是客户端 `选择文件并点击上传`，浏览器将文件内容读取然后传送到服务器；

服务端读取数据并新建文件，写入浏览器传输过来的内容。

## 单个文件上传

!!! note ""
    === "前端部分"

        ```html
        <!DOCTYPE html>
        <html lang="zh-CN">
        <head>
            <meta charset="UTF-8">
        </head>
        <body>

            <form action="/upload" method="post" enctype="multipart/form-data">
                <input type="file" name="f1"/>
                <input type="submit" value="上传"/>
            </form>

        </body>
        </html>
        ```

        ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/6000782296428.png)

        前端需要用一个表单来进行上传，这里我们设置了表单提交的请求方式为 POST。

        注意一定要指定编码类型 `enctype="multipart/form-data"`

        上传文件后点击上传按钮会向 `http://localhost:9090/upload` 这个路径发起POST请求。

    === "后端部分"

        ```go
        func main() {
            r := gin.Default()

            r.POST("/upload", func(c *gin.Context) {
                // 从请求中读取文件
                f, _ := c.FormFile("f1")
                // 保存文件
                c.SaveUploadedFile(f, "./" + f.Filename)
                // 给客户端返回上传成功的消息
                c.JSON(200, gin.H{"code": 2003, "msg": "上传成功"})
            })

            r.Run(":9090")
        }
        ```
        后端的处理主要分3步：

        1. 从请求中读取文件
        2. 保存文件
        3. 给客户端返回上传成功的消息

        通过 `gin.FormFile()` 可以获取前端上传的文件，参数是 `<input />` 标签的 `name` 属性，通过参数可以定位到上传的任何一个文件。

        获取到文件后可以保存在数据库中，或者通过 `gin.SaveUploadedFile()` 将文件保存在某个位置，只需要将文件对象 `f` 和 路径作为参数传入即可。

        最后给客户端返回一条上传成功的消息。


    === "测试"

        ![上传文件](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1762924054854.png)

        ![上传成功](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/5500107717484.png)

        ![成功保存](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/5880663088309.png)

    === "使用 Postman"

        使用 PostMan 时需要给 Header 添加一个属性 `Content-Type: multipart/form-data`

        ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/3692804942610.png)

        然后在 Body 中的 form-data 中上传数据

        ![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1504988688338.png)

    === "使用 curl 命令"

        ```bash
        curl -X POST http://localhost:9090/upload \
        -F "file=@/xx/xx/xxx.zip" \
        -H "Content-Type: multipart/form-data"
        ```

## 多个文件上传

!!! note ""
    === "前端部分"

        ```html
        <!DOCTYPE html>
        <html lang="zh-CN">

        <head>
            <meta charset="UTF-8">
        </head>

        <body>
            <form action="/upload_multi" method="post" enctype="multipart/form-data">

                <input type="file" name="files" multiple>
                <input type="submit" value="上传">

            </form>

        </body>

        </html>
        ```

    === "后端部分"

        ```go
        func main() {
            r := gin.Default()

            r.POST("/upload", func(c *gin.Context) {
                // 从请求中读取文件
                form, _ := c.MultipartForm()
                files := form.File["files"]

                // 逐个保存文件
                for _, file := range files {
                    c.SaveUploadedFile(f, "./"+f.Filename)
                }

                // 给客户端返回上传成功的消息
                c.JSON(200, gin.H{"code": 2003, "msg": "上传成功"})
            })

            r.Run(":9090")
        }
        ```

单个文件上传和多个文件上传只有一点点区别

多个文件上传使用 `gin.Context.MultipartForm()` 来获取，该方法会返回一个 Form 结构体，结构体中map类型字段 `File` 存储了所有已上传的文件

通过遍历来逐个处理，然后给客户端返回上传成功的消息即可。


## 上传文件大小限制

gin 默认上传的文件大小最大为 32Mb，如果要调小或调大，可以设置 `gin.Engine.MaxMultipartMemory`

!!! example

    ```go
    func main() {
        r := gin.Default()
        // 为 multipart forms 设置较高的内存限制 (默认是 32 MiB)
            r.MaxMultipartMemory = 2 << 30;    // 2*2^30 byte = 2Gb

        r.Run(":9090")
    }
    ```

## 下载文件

实现下载功能要先获取文件名，然后给响应头添加一个属性 `Content-Disposition: attachment; filename=文件名`

然后调用 `gin.Context.File()` 方法，传入文件路径即可。

```go
func main() {
    r := gin.Default()
    filename := "a.txt"
    r.GET("/download", func(c *gin.Context) {
        // 获取文件名
        filename := "a.txt"
        // 设置响应头
        c.Header("Content-Disposition", "attachment; filename="+filename)
        // 向客户端返回文件数据
        c.File("./"+filename)
    }
    r.Run(":9090")
}
```

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1820376192887.png)


当然，还可以在响应头中添加更多信息

```go
func main() {
    r := gin.Default()
    filename := "a.txt"
    r.GET("/download", func(c *gin.Context) {
        // 获取文件名
        filename := "a.txt"
        // 设置状态码
        c.Writer.WriteHeader(http.StatusOK)
        // 添加内容位置声明
        c.Header("Content-Disposition", "attachment; filename="+filename)
        // 添加内容类型声明
        c.Header("Content-Type", "application/text/plain")
        // 向客户端返回数据
        c.File("./" + filename)
    }
    r.Run(":9090")
}
```

!!! warning "注意"
    `c.File(path)` 中的参数 path 最好是绝对路径，可以使用 `path.Join(路径, 文件名)` 来得到绝对路径。

    处理不好 path 参数容易出错。

## 重定向 和 转发

重定向和转发在实际开发过程中很有必要。

比如对 API 进行升级后，一些原本的请求路径不再使用，但是为了兼容，防止一些旧版本继续请求的时候请求失败，所以需要对原本的路由做重定向或转发处理。


区别：

- 重定向：用户请求一个旧的地址的时候，返回301，并告知新地址；然后用户再次请求新地址，得到正确返回。
- 重定向：重定向过后浏览器的URL会发生变化。
- 重定向：可以将请求转到别的地址，这个地址可以是外部链接，也可以是内部其他路由。

- 转发：用户请求一个旧地址的时候，服务端内部将请求路径修改为新地址，然后返回给用户
- 转发：转发过后浏览器的URL不会发生变化。
- 转发：只能转发到站内路由。


![重定向和转发](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/3628459928845.png)


=== "重定向"

    ```go
    c.Redirect(http.StatusMovedPermanently, "重定向的路径")
    ```

    eg：

    ```go
    func main() {
        r := gin.Default()

        r.GET("/index", func(c *gin.Context) {
            c.JSON(http.StatusOK, gin.H{ "msg": c.FullPath() })
        })

        // 重定向到站内其他路由
        r.GET("/redirect", func(c *gin.Context) {
            c.Redirect(http.StatusMovedPermanently, "/index")
        })

        // 重定向到站外链接
        r.GET("/redirect2", func(c *gin.Context) {
            c.Redirect(http.StatusMovedPermanently, "https://www.boii.xyz")
        })

        r.Run(":9090")
    }
    ```

    ![重定向到站内](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/5296060724295.png)

    ![重定向到站外](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/3265866644390.png)

=== "转发"

    ```go
    c.Request.URL.Path = "转发的目标路径"
    ```

    eg：

    ```go
    func main() {
        r := gin.Default()

        r.GET("/index", func(c *gin.Context) {
            c.JSON(http.StatusOK, gin.H{ "msg": c.FullPath() })
        })

        // 转发到站内其他路由
        r.GET("/forward", func(c *gin.Context) { // URL 不会改变
            c.Request.URL.Path = "/index"    // 修改请求的目标路径
            r.HandleContext(c)
        })

        r.Run(":9090")
    }
    ```

    ![转发到站内](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/623353626862.png)

