# 路由组

开发中常常会遇到一种需求：

将系统划分成多个模块，例如用户管理模块、书籍管理模块等，这些模块下各自有子模块。

在设计路由路径的时候可能就需要设置成下面的样子：

- /user/register
- /user/login
- /user/:id/info
- /user/:id/setting
- /book/add
- /book/search/:id
- /book/remove/:id
- /book/modify/:id

这样的设计虽然没有什么问题，但是不方便管理，如果突然对 user 或者 book 这个大的模块进行修改还导致所有子模块全部都要改动。

所以合理的使用 **路由组** 可以很好的解决这些问题。

## 示例

???+ example 

    ```go
    // main.go
    func main() {
        r := gin.Default()
        // 创建 /user 路由组
        userG := r.Group("/user")
        // {} 是书写规范
        {
            userG.POST("/register", Register)
            userG.POST("/login", Login)
        }

        // 创建 /book 路由组
        bookG := r.Group("/book")
        {
            bookG.GET("/search/:id", SearchBook)
            bookG.DELETE("/remove/:id", RemoveBook)
        }

        r.Run(":9090")
    }

    // routes.go
    type UserInfo struct {
        User string `form:"user"`
        Pass string `form:"pass"`
    }

    func Register(c *gin.Context) {
        var u UserInfo
        c.ShouldBind(&u)
        c.JSON(http.StatusOK, gin.H{ "msg": "Hello " + u.User })
    }

    func Login(c *gin.Context) {
        var u UserInfo
        c.ShouldBind(&u)
        c.JSON(http.StatusOK, gin.H{ "msg": "Hello " + u.User })
    }

    func SearchBook(c *gin.Context) {
        bookID := c.Param("id")
        c.JSON(http.StatusOK, gin.H{ "msg": "ID " + bookID })
    }

    func RemoveBook(c *gin.Context) {
        bookID := c.Param("id")
        c.JSON(http.StatusOK, gin.H{ "msg": "ID " + bookID + "was removed." })
    }
    ```


首先通过 `gin.Group()` 传入组的共同前缀，例如上面的 `/user`、`/book`，获得一个路由组对象

然后就可以像之前一样的去绑定各个子路由了。之前的 `GET()`、`POST()` 等方法同样适用。

效果：

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/3664825137700.png)


![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/vx_images/4105243857095.png)



## 路由拆分和注册

