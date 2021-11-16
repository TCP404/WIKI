---
title: 'Gin [0-Get Start]'
seotitle: 'Gin [0-Get Start]'
icons:
  - fas fa-fire red
  - fas fa-star yellow
pin: false
tags:
  - gin
categories:
  - Golang
  - Framework
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/Gin.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/gin.png'
abbrlink: 451cd62
date: 2021-07-19 18:12:21
updated: 2021-07-19 18:12:21
---

Hello Gin.

<!--more-->

# GetStart

## 要求
- \>= Go v1.13

## 安装

前提：

- 安装 [Golang](https://golang.org/dl/)
- 设置 Go 工作区

1. 下载并安装 gin：

```bash
$ go get -u github.com/gin-gonic/gin
```

2. 在代码中引入
```go
import "github.com/gin-gonic/gin"
import "net/http"
```

## 开始

先创建一个go文件
```bash
$ touch demo.go
```

接着编写下面的代码：
```go
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

func main() {
    app := gin.Default()
    app.GET("/hi", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "msg": "Hello Gin!",
        })
    })
    app.Run()    // 默认运行在 8080 端口
}
```

然后执行 `go run demo.go` 命令运行代码：
```bash
$ go rum demo.go
```

最后打开浏览器访问 [http://localhost:8080/hi](http://localhost:8080/hi)