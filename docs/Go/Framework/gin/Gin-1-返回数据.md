# 返回

在接收到请求，经过一番处理之后，我们需要返回结果，也就是响应请求。

响应请求的格式有 >= 5种格式：

- []byte
- string
- JSON
- XML
- YAML
- protobuf
- Template


=== "[]byte 方式"

    返回字节切片格式时，需要使用 `gin.Context.Writer` 的 `Write` 方法，将字节切片作为参数传入即可，无需填写状态码。

    ```go
    func main() {
        r := gin.Default()

        r.GET("/byte", func(c *gin.Context) {
            fullPath := "请求路径： " + c.FullPath()

            c.Writer.Write([]byte(fullPath))
        })

        r.Run()
    }
    ```

    ![byte 的返回结果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/745417048783.png)


=== "string 方式"

    使用 string 方式则方便许多，直接使用 `gin.Context` 的 `String` 方法；

    ```go
    func (c *Context) String(code int, format string, values ...interface{});
    ```
    参数含义为`状态码、格式字符串、值`。

    ```go
    func main() {
        r := gin.Default()

        r.GET("/byte", func(c *gin.Context) {
            fullpath := "请求路径： " + c.FullPath()

            c.String(200, "请求路径: %s", c.FullPath())
        })

        r.Run()
    }
    ```

    ![string 返回结果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/1050203606306.png)


=== "JSON 格式"

    最常用的就是返回 JSON 格式，方法时使用 `gin.Context` 的 `JSON` 方法：

    ```go
    func (c *Context) JSON(code int, obj interface{});
    ```
    参数含义为 `状态码、数据`

    返回的数据可以用 `map[string]interface{}` 来装载，而 gin 贴心的为这个类型做了定义 `gin.H`。
    可以直接使用 `gin.H`，这样就不用写长长一串了。

    ```go
    func main() {
        r := gin.Default()

        r.GET("/json1", func(c *gin.Context) {
            // 方法1 使用 map[string]interface{}，即 gin.H
            data := gin.H{
                "name": "Boii",
                "age":  17,
            }
            c.JSON(http.StatusOK, data)
        })

        r.Run()
    }
    ```

    除了 `gin.H`，我们还可以使用结构体来装载我们的数据。

    ```go
    func main() {
        r := gin.Default()

        r.GET("/json2", func(c *gin.Context) {
            // 方法2 使用结构体
            data := struct {
                Name string `json:"name"`
                Age  int    `json:"age"`
            }{"Boii", 18}
            c.JSON(http.StatusOK, data)
        })

        r.Run()
    }
    ```

    ![Json 结果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/872328395398.png)


=== "YAML 格式"

    yaml 也是一种很好用的格式，近年来也开始逐渐兴起

    ```go
    func (c *Context) YAML(code int, obj interface{});
    ```
    参数含义为 `状态码、数据`，基本和JSON是一样的，数据可以使用 `gin.H` 也可以使用结构体。

    ```go
    func main() {
        r := gin.Default()

        r.GET("/json1", func(c *gin.Context) {
            // 方法1 使用 map[string]interface{}，即 gin.H
            data := gin.H{
                "name": "Boii",
                "age":  17,
            }
            c.YAML(http.StatusOK, data)
        })

        r.Run()
    }
    ```

    除了 `gin.H`，我们还可以使用结构体来装载我们的数据。

    ```go
    func main() {
        r := gin.Default()

        r.GET("/json2", func(c *gin.Context) {
            // 方法2 使用结构体
            data := struct {
                Name string `json:"name"`
                Age  int    `json:"age"`
            }{"Boii", 18}
            c.YAML(http.StatusOK, data)
        })

        r.Run()
    }
    ```

    ![yaml 结果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/4617055721149.png)


=== "XML 格式"

    还有一种也是很常用的格式是 XML，但是这种只能使用 `gin.H`，不能使用结构体装载数据。

    ```go
    func (c *Context) XML(code int, obj interface{});
    ```

    ```go
    func main() {
        r := gin.Default()

        r.GET("/xml", func(c *gin.Context) {
            data := gin.H{
                "name": "Boii",
                "age": 18,
            }
            c.XML(http.StatusOK, data)
        })

        r.Run()
    }
    ```
    ![XML 结果](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/vx_images/5324325669553.png)

