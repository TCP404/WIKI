反射，底层开发用的比较多，应用层开发用的不多。

反射三法则：

1. 从 ==接口值== 可以得到 ==反射对象==
2. 从 ==反射对象== 可以得到 ==接口值==
3. 要修改反射对象，其值必须可设置

两个基本反射对象：

- reflect.Type
- reflect.Value

反射包 reflect 基本就是围绕这两个类型展开的

三个基本概念：

1. Value：值
2. Type：类型，可以有很多种，你也可以自定义类型
3. Kind：种类，只有那几种，都是 go 预先定义的那几种

值很好理解，而类型和种类需要区别一下。

![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Go/IMG/structure-of-interface-variable.jpg)

类型可以有很多种，你也可以自定义类型；而种类只有那几种，都是 go 预先定义的那几种。不管什么类型都需要归类到预设的这些种类中。具体有哪写种类可以到 [go 官方文档中查看](https://pkg.go.dev/reflect#Kind)

eg：

```go
package main

import (
    "reflect"
    "fmt"
)

type Person struct {    // 定义一个结构体，名为 Person
    Name string
    Age  int
}

func main() {
    // 创建一个结构体对象
    p := Person{
        Name: "Boii",
        Age:  18,
    }

    t := reflect.TypeOf(p)    // 拿到 p 的类型

    fmt.Println("Type", t)            // main.Person
    fmt.Println("Kind", t.Kind())    // struct
}
```

类型 Type 和 Kind 一定要分清楚，它俩不仅仅是反射包里有，在认识接口 interface 的时候也非常关键。一定要牢记它俩的区别。Type 有无限多种可能，Kind 只有 go 定义的那几种。

## 反射操作结构体

## 反射操作函数
