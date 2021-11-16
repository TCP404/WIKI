---
title: 'Golang [基础] 10-结构体'
seotitle: 'Golang [基础] 10-结构体'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: ff53e8a4
date: 2021-07-17 11:30:00
updated: 2021-07-17 11:30:00
---

结构体的创建与使用

<!--more-->

# 10-结构体

Go语言中没有“类”的概念，也不支持“类”的继承等面向对象的概念。Go语言中通过结构体的内嵌再配合接口比面向对象具有更高的扩展性和灵活性。

当我们想表示一些事物时，我们可以用基本数据类型表示其各项基本属性，通过结构体将其组合起来。在 Go 中可以通过 `struct` 实现面向对象。

## 定义

{% note warning, **注意：结构体是值类型！！！** %}

使用 `type` 和 `struct` 来定义结构体。

基本语法如下：
```go
type structT struct {
    field1 T
    field2 T
    ...
}
```
`structT`：标识只定义结构体名称，同一包内不能重复。
`field`：结构的基本属性的名字，结构体中的字段名不能重复
`T`：字段的具体类型

eg：
```go
type person struct {
    name string
    age int8
    city string
}
```
同类型字段可以写在一行
```go
type person struct {
    name, city string
    age int8
}
```

通过以上代码，我们就可以得到一个 `person` 的自定义类型，它有 `name`、`age`、`city` 三个字段，表示人的姓名、年龄、城市三个属性。
这样就可以通过 `person` 这个结构体很方便的在程序中表示和存储人的信息了。

> 在定义结构体时，建议各个字段按字段类型从小到大排序，有助于[内存对齐](https://segmentfault.com/a/1190000017527311)。
> 类型大小参考[《数据类型》一章](https://blog.csdn.net/mr_pro/article/details/113820586)

Go 内置的基本数据类型用来描述一个值，而结构体用来描述一组值，本质上是一种聚合型数据类型。
一个结构体就像 Java 中的一个类，不过 Java 中的类还有构造函数、方法等，这些 Go 的结构体一样可以实现，方式有些不同

## 实例化
只有当结构体实例化时，才会真正地分配内存。即必须实例化后才能使用结构体的字段。
结构体本身也是一种数据类型，我们可以像声明基本数据类型一样声明结构体

```go
// 方式1
var identifier structT
identifier.field1 = value1
identifier.field2 = value2
...
```
`identifier`：结构体实例名称
`structT`：结构体类型
`field`：结构体字段

### 方式1：基本实例化
```go
type person struct {
    name string
    city string
    age int8
}

func main() {
    // 方式1
    var p1 person
    p1.city = "SWA"
    p1.name = "Boii"

    fmt.Printf("p1 = %v \n", p1)	// p1 = {Boii 0 SWA}
    fmt.Printf("p1 = %#v \n", p1)	// p1 = main.person{name:"Boii", age:0, city:"SWA"}
}
```
仔细观察：
- 方式1声明以后逐一给每个字段赋值，可以结构体中定义字段时的顺序，没有赋值的字段默认为零值
- 方式2 和 方式3在赋值的时候需要所有字段都赋值，且需要按顺序。
- 通过`.`可以访问结构体中的字段，例如`p1.city`.

### 方式2：new(T) 结构体指针
通过 `new()` 可以对结构体实例化，得到的是结构体指针，其各个字段都为零值。

```go
type person struct {
    name string
    city string
    age  int8
}
func main() {

    var p1 = new(person)
    fmt.Printf("%T \n", p1)      // *main.person
    fmt.Printf("p1 = %#v \n", p1) // p1 = &main.person{name:"", age:0, city:""}
    
    p1.name = "Boii"
    fmt.Printf("p1 = %#v \n", p1) // p1 = &main.person{name:"Boii", age:0, city:""}
}
```
**注意**：Go 中的结构体指针可以直接使用`.`来访问结构体成员。
`p1.name = "Boii"` 相当于 `(*p1).name = "Boii"`，这是 Go 的语法糖。

### 方式3：&T{} 取结构体地址
使用 `&` 对结构体取地址操作相当于对该结构体类型进行了一次 `new`实例化操作。
```go
type person struct {
    name string
    city string
    age  int8
}
func main() {
    var p2 = &person{"Boii", 10, "SWA"}
    fmt.Printf("%T \n", p2)       // *main.person
    fmt.Printf("p2 = %#v \n", p2) // p = &main.person{name:"Boii", age:10, city:"SWA"}

    p3 := &person{}
    p3.name = "Candy"
    p3.age = 10
    p3.city = "SWA"
    fmt.Printf("%T \n", p3)       // *main.person
    fmt.Printf("p3 = %#v \n", p3) // p = &main.person{name:"Candy", age:10, city:"SWA"}

}
```

所以：`new(Type)` 和 `&Type{}` 是等价的。

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201122164056342_11482.png)

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201122164122020_21432.png)

### 匿名结构体

在定义一些临时数据结构等场景下还可以使用匿名结构体。
```go
func main() {
    // 定义匿名结构体
    // 字段之间用 分号 隔开
    var user struct{name string; age int8}
    // 通过结构体实例使用匿名结构体
    user.name = "Boii"
    user.age = 18

    fmt.Printf("%v \n", user)  // {Boii 18}
    fmt.Printf("%#v \n", user) // struct { name string; age int8 }{name:"Boii", age:18}

    // 相同类型可以简写
    var stu struct{name string; ID, age int}

    // 规范写法
    var stud struct {
        name    string
        ID, age int
    }
}
```

## 初始化
结构体没有初始化的时候，成员变量都是对应类型的零值，这点在上面的例子中已经有所体现。
```go
type stu struct {
    name string
    age  int8
    ID   int8
}

func main() {
    var s1 stu
    fmt.Printf("s1 = %#v \n", s1) // s1 = main.stu{name:"", age:0, ID:0}
}
```

### 使用键值对的方式初始化
```go
identifier := structT{
    field1: value1,
    field2: value2,
    ...
}


var identifier = structT{
    field1: value1,
    field2: value2,
    ...
}
```

```go
s2 := stu{
    name: "Boii",
    age:  18,
}
fmt.Printf("s2 = %#v \n", s2) // s2 = main.stu{name:"Boii", age:18, ID:0}

var s3 = stu{
    name: "Eva",
    age:  18,
}
fmt.Printf("s3 = %#v \n", s3) // s3 = main.stu{name:"Eva", age:18, ID:0}
```
使用键值对的方式可以不按结构体定义时字段的顺序，可以不初始化每一个字段，没被初始化的字段就默认为零值。
注意最后一个字段也要加上逗号`,`。

```go
s3 := &stu{
    age: 18,
    ID:  101,
}
fmt.Printf("s3 = %#v \n", s3) // s3 = &main.stu{name:"", age:18, ID:101}
```
也可以对结构体指针进行键值对初始化

### 使用值列表的方式初始化
初始化结构体的时候可以简写，不写键，只写值。
```go
identifier := structT{
    value1,
    value2,
    ...
}

var identifier = structT{
    value1,
    value2,
    ...
}
```

eg：
```go
s4 := stu{
    "Boii",
    18,
    102,
}
fmt.Printf("s4 = %#v \n", s4) // s4 = main.stu{name:"Boii", age:18, ID:102}
```
使用这种要注意：
1. 必须初始化结构体里所有字段
2. 初始值填充顺序必须和字段在结构体中声明的顺序一致
3. 不能和键值对的初始化方式混用

### 一点细节

初始化时，如果值写成多行，则最后一个也要带上逗号 `,` ;
如果写成一行，则最后一个可以不带逗号 `,` ; 也可以带上逗号。



```go
type Person struct {
    name string
    age  int
    sex  string
}

func main() {
    p1 := Person{
        "Boii",
        18,
        "male",
    }

    p2 := Person{
        name: "Boii",
        sex:  "male",
        age:  18,
    }

    p3 := Person{ "Boii", 18, "male" }

    p4 := Person{ name: "Boii", sex: "male", age: 18, }
}
```

## 方法和接收者

例如下面的通式，方法名前面的**接收者变量和接收者类型**，就是一种限定。
```go
// 方法
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
    方法体
}

// 对比：函数
func 函数名(参数列表) (返回参数) {
    函数体
}
```
**接收者指的就是被允许调用的调用者**。注意理解：下面将以接收者代替调用者的说法。

- **接收者变量**：命名时，官方建议使用接收者类型首个字母小写。eg：`p Person`、`c Connector`
- **接收者类型**：接收者类型和参数相似，可以是指针类型或非指针类型
- **方法名、参数列表、返回参数**：具体格式与函数定义相同

```go
// Dog 结构体
type Dog struct {
    name string
    age  int8
}

// NewDog 构造函数
func NewDog(name string, age int8) *Dog {
    return &Dog{name, age}
}

// Run Dog奔跑的方法
// 限定了 Dog 结构体类型才可以调用此方法
func (d Dog) Run(distance int) {
    fmt.Printf("%d 岁的狗狗 %s 跑了 %d 米\n", d.age, d.name, distance)
}

func main() {
    boby := NewDog("Boby", 2)
    boby.Run(100)
    doge := Dog{"Doge", 3}
    doge.Run(12)
}

// -------------------------------------------------
// Output:
2 岁的狗狗 Boby 跑了 100 米
3 岁的狗狗 Doge 跑了 12 米
```

### 指针类型的接收者
> 如果指定指针类型的接收者，在方法内修改接收者的成员变量，则**结束方法后修改依然有效**。

例如我们为 `Dog` 添加一个 `setAge` 方法：
```go
func (d *Dog) SetAge(newAge int8) {
    d.age = newAge
}

func main() {
	boby := NewDog("Boby", 2)
	fmt.Println(boby.age) // 2
	boby.SetAge(10)
	fmt.Println(boby.age) // 10
}
```

### 值类型的接收者
> 当方法作用于值类型接收者时，Go 会在运行时将接收者的值复制一份。不过**在方法内对接收者的修改只是对副本的修改**，不会影响接收者。
>
> 但是在编译器中，一个结构体“值实现”了一个方法，编译器会自动帮你多来一次“指针实现”。
> 即值实现 = 值实现 + 指针实现；指针实现 = 指针实现。
>
> 这一点在接口中有区别。

```go
func (d Dog) SetAge(newAge int8) {
    d.age = newAge
}

func main() {
	boby := NewDog("Boby", 2)
	fmt.Println(boby.age) // 2
	boby.SetAge(10)
	fmt.Println(boby.age) // 2
}
```

### 两种类型接收者的差异

值接收者方法（Value receiver method）和指针接收者方法（Pointer receiver method），都可以被结构体变量（struct variable）或结构体（struct pointer）指针调用。

值接收者方法中，对接收者的修改，**不会**影响调用者。
指针接收者方法中，对接收者的修改，**会**影响调用者。

```go
type A struct {
    age int
}

func (a A) read() {
    a.age = 50
}

func (a *A) write(n int) {
    a.age = n
}

func main() {
    aV := A{age: 10}
    aP := &A{age: 10}
    
    aV.read()
    fmt.Println(aV.age)	// 10
    aV.write(15)
    fmt.Println(aV.age)	// 15
    
    aP.read()
    fmt.Println(aP.age)	// 10
    aP.write(18)
    fmt.Println(aP.age)	// 18
}
```

在上述栗子中，

-   aP 是结构体指针，在 aP 调用值方法 `aP.read()` 时，编译器会自动转换为 `(*aP).read()`；

-   aV 是结构体变量，在 aV 调用指针方法 `aV.write()` 时，编译器会自动转换为 `(&aV).write(15)`。

|                      | **值接收者方法**                | **指针接收者方法**                       |
| -------------------- | ------------------------------- | ---------------------------------------- |
| **结构体变量调用者** | 可以调用，类似于传值            | 可以调用，相当于 `(&aV).write(15)`       |
| **结构体指针调用者** | 可以调用，相当于 `(*aP).read()` | 可以调用，类似与传指针，所以会影响调用者 |
| **修改**             | 不影响调用者                    | 影响调用者                               |

#### 如何选择

要看类型的本质。如果一个结构体类型的实例，应该是独一份的，那么就应该用指针接收者方法。

例如，文件结构体。每个文件都是独一份的，对应地，文件结构体返回一个文件对象，这个对象应该唯一的代表这个文件，所以文件结构体的方法应该使用指针接收者方法。

### 任意类型添加方法
在 Go 中，接收者类型可以是任何类型，不仅仅是结构体，任何类型都可以拥有方法。

```go
type Status int    // 自定义类型

func (s Status) say() {
    fmt.Println("OK")
}

func main() {
    var s Status
    s.say()
}
```

> **注意**： 非本地类型不能定义方法，也就是说我们不能给别的包的类型定义方法。



## 内存布局

在 Go 中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套了结构体也一样。

```go
type person struct {
    name string
    age  int8
}

type stu1 struct {
    ID     int8
    info   person
    depart string
}

type stu2 struct {
    ID     int8
    info   *person
    depart string
}
```
![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201123131640736_15271.png)

## 匿名字段
Go 中的结构体允许声明时只有类型而没有名字，这样的字段叫做匿名字段

```go
type stu struct {
    string
    int8
}

func main() {
    s1 := stu{
        "Boii",
        18,
    }
    fmt.Printf("%#v\n", s1)         // main.stu{string:"Boii", int8:18}
    fmt.Println(s1.string, s1.int8) // Boii 18
}
```
> 匿名字段默认采用类型作为字段名，同一个结构体中一种类型只能有一个匿名字段。

所以上面的结构体等价于：
```go
type stu struct {
    string string
    int8   int8
}
```

## 嵌套结构体

嵌套结构体，简单说就是套娃
```go
type person struct {
    name string
    age  int8
}

// 娃娃2
type stu struct {
    ID     int8
    info   person  // 套娃娃1
    depart string
}

func main() {
    stuA := stu{
        ID: 101,
        info: person{
            "Alice",
            18,
        },
        depart: "CA",
    }
    fmt.Printf("%#v\n", stuA) // main.stu{ID:1, info:main.person{name:"Boii", age:18}, depart:"CA"}
}
```

如果结构体太大，担心开销的话，可以用结构体指针
```go
// 娃娃1
type person struct {
    name string
    age  int8
}

// 娃娃2
type stu2 struct {
    ID     int8
    info   *person // 套娃娃1的指针
    depart string
}

func main() {
    stuB := stu2{
        ID: 102,
        info: &person{
            "Boii",
            20,
        },
        depart: "SE",
    }
    fmt.Printf("%#v\n", stuB) // main.stu2{ID:1, info:(*main.person)(0xc0000044a0), depart:"CA"}
}
```

用结构体和结构体指针的差别如下图：
![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20201123131640736_15271.png)

显然当结构体较大的时候，使用结构体指针的操作开销会很小。

### 嵌套结构体的匿名字段

```go
type addr struct {
    province string
    city     string
}

type user struct {
    name string
    age  int8
    addr // 匿名字段
}

func main() {
    var userA user
    userA.name = "Boii"
    userA.age = 18
    userA.addr.province = "GD" // 匿名字段默认使用类型名作为字段名
    userA.city = "SWA"         // 匿名字段可以省略

    fmt.Printf("%#v \n", userA) // main.user{name:"Boii", age:18, addr:main.addr{province:"GD", city:"SWA"}}
}
```
### 嵌套结构体命名冲突
当嵌套结构体出现命名冲突的时候，只要不省略，把关系链写清楚就没事了。

```go
type addr struct {
    province   string
    city       string
    createTime string    // 命名冲突
}

type user struct {
    name       string
    age        int8
    addr       // 匿名字段
    createTime string    // 命名冲突
}

func main() {
    var userA user
    userA.name = "Boii"
    userA.age = 18
    userA.addr.province = "GD"
    userA.city = "SWA"
    // 命名冲突字段
    userA.createTime = "2020"        // 关系链写清楚
    userA.addr.createTime = "2021"   // 关系链写清楚

    fmt.Printf("%#v \n", userA)
    // main.user{name:"Boii", age:18, addr:main.addr{province:"GD", city:"SWA", createTime:"2021"}, createTime:"2020"}
}
```


## 结构体的继承
在面向对象中，继承指的是子类可以拥有父类所有非私有属性和方法。

Go 中使用结构体嵌套也可以实现其他编程语言中面向对象的继承。
方法是通过在 "子结构体" 中嵌套 "父结构体" ，这样 "子结构体" 就拥有 "父结构体" 的属性了

结合上面 “嵌套结构体的匿名字段” 中的例子，无命名冲突时，可以通过 `子结构体实例.父结构体属性`调用的特性，可以很容易的编写出适合 "子结构体" 的方法。

```go
// animal 动物
type animal struct {
    name string
}

// dog 狗
type dog struct {
    variety string
    *animal // 子结构体嵌套父结构体实现继承，必须使用匿名字段
}

func (d *dog) bark() {
    // dog 没有 name 属性，但是内嵌的 animal 有
    fmt.Printf("%s 正在汪汪汪~", d.name)
}

func main() {
    doge := &dog{
        variety: "Doge",
        animal: &animal{ // 注意嵌套的是结构体指针
            name: "Boby",
        },
    }
    doge.bark() // Boby 正在汪汪汪~
}
```



## 结构体标签 Tag
`Tag` 是结构体的元信息，可以在运行的时候通过反射机制读取。所以只有首字母大写的可被导出的变量能被反射读取，转成JSON。
`Tag` 在结构体字段的后方定义，由一对**反引号**包起来。

eg：
```go
type person struct {
    Name string `json:"name"`
}
```

- 一个 Tag 由一个或多个键值对组成。
- **键值对**之间使用空格分隔
- **键**与**值**之间使用冒号分隔，不能有空格
- 值使用双引号括起来。
- 值为 `-` 时该字段不会被序列化

编写 `Tag` 时必须严格遵守键值对的规则。结构体标签的解析容错能力很差，写错了编译期和运行期都不会提示任何错误，反射也无法正确取值。



## 结构体序列化

**序列化（struct -> JSON）**：结构体对象 转成 JSON格式字符串
**反序列化（JSON -> struct）**：JSON格式字符串 转成 结构体对象

使用 `encoding/json` 完成。
```go
import "encoding/json"
```

```go
// 序列化函数，struct -> JSON
func Marshal(v interface{}) ([]byte, error) {}
// 反序列化函数，JSON -> struct
func Unmarshal(data []byte, v interface{}) error {}
```

eg：
```go
package main

import (
    "encoding/json"
    "fmt"
)

// 1. 序列化 encode：把结构体变量 -> json格式的字符串
// 2. 反序列化 decode：json格式的字符串 -> 结构体变量

type person struct {
    Name string `json:"name" db:"name" ini:"name"`
    Age  int    `json:"age"`
    Sex  string
    Addr string `json:"-"`    // value 为 "-" 时不序列化
}

func main() {
    p1 := person{"Boii", 18, "male"}

    /* 序列化 */
    b, err := json.Marshal(p1)
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(string(b)) // {"name":"Boii","age":18,"Sex":"male"}
    fmt.Printf("%T \n", b) // []uint8

    /* 反序列化 */
    str := `{"name":"Eva","age":18}`
    var p2 person
    json.Unmarshal([]byte(str), &p2)
    fmt.Printf("%#v \n", p2) // main.person{Name:"Eva", Age:18, Sex:""}
    fmt.Printf("%T \n", p2)  // main.person
}
```
序列化时，key 会优先选择字段的 Tag 中指定的 key

{% note warning, **结构体中非 public 的字段不会被序列化** %}

## 结构体的深浅拷贝

{% noteblock info %}
深拷贝：即**为新的对象分配了内存**，对新对象的修改不会影响旧对象
浅拷贝：即**复制了旧对象的地址**，对新对象的修改会影响旧对象

值类型都是深拷贝
引用类型都是浅拷贝
{% endnoteblock %}

看懂以上两句话，结构体的深浅拷贝就很容易实现了

```go
package main

import "fmt"

type Person struct {
    name string
    age  int
}

func main() {
    // 结构体的深拷贝
    person1 := Person{"Boii", 18}
    person2 := person1   // 深拷贝
    fmt.Println(person1) // {Boii 18}
    fmt.Println(person2) // {Boii 18}

    person2.age = 20

    fmt.Println(person1) // {Boii 18}
    fmt.Println(person2) // {Boii 20}

    // 结构体的浅拷贝
    person3 := &person1 // 浅拷贝
    person3.age = 20
    fmt.Println(person1) // {Boii 20}
    fmt.Println(person3) // &{Boii 20}
}
```
结构体是值类型，直接赋值的时候是深拷贝，赋值一个指针则是浅拷贝
