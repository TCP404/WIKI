# 选项设计模式

在编写程序时，经常会遇到这样一类场景：

定义一个包内可见的结构体，对外提供一个 `NewSTRUCT()` 方法来创建。 而创建时要求使用者给出结构体变量的值作为参数。当结构体变量很多的时候，会写出一堆很丑很可怕的代码。例如：

```go
type student struct {
    name   string	// *必填
    age    int		// *必填
    height int		// 选填
    weight int		// 选填 
    score  float32	// 选填
}

func NewStudent (name string, age int) *student {
    return &student{name: name, age: age}
}

func NewStudentWithHight (name string, age, height int) *student {
    return &student{name: name, age: age, height: height}
}

func NewStudentWithWeight (name string, age, weight int) *student {
    return &student{name: name, age: age, weight: weight}
}

func NewStudentWithScore (name string, age int, score float32) *student {
    return &student{name: name, age: age, score: score}
}

func NewStudentWithHightAndWeight(name string, age, height, weight int) *student {
    return &student{name: name, age: age, height: height, weight: weight}
}
...
```

这样的代码真的… 又臭又长。

因为 Golang 不支持函数重载，所以你不得不使用不同的函数名来对应不同的配置选项。

解决方法有两种：`Builder 模式` 和 `Function Option 模式`

## Builder 模式

Builder 模式又称生成器模式，通过为目标结构体多创建一个 **Builder 结构体** 来负责创建 student 这件事，一般以`目标结构体的名称 + Builder` 命名。

使用 Builder 模式的步骤如下：

1.   定义一个 Builder 结构体，并在结构体中携带一个目标结构体的指针
2.   并在构造 Builder 的函数里要求一些 *必填* 的参数，然后返回一个 Builder 对象
3.   逐一为目标结构体的 *选填* 参数定义属于 Builder 结构体的方法，注意这些方法的返回值只能是 Builder结构体，这样才可以链式调用
4.   最后编写 Builder 结构体的 `Build()` 方法，返回目标结构体

```go
// 目标结构体：student
type student struct {
    name   string	// *必填
    age    int		// *必填
    height int		// 选填
    weight int		// 选填 
    score  float32	// 选填
}

// 目标结构体的 builder：studentBuilder
type studentBuilder struct {
    stu *student	// 携带一个目标结构体 student 的指针
}

// 学生 builder 的创建方法，要求输入必填项
func NewStudentBuilder (name string, age int) *studentBuilder{
    s := student{name: name, age: age}
    return &studentBuilder{stu: &s}
}

// 其他选填项，重点在于这是 builder 的方法，返回值是 builder 自己，这样才可以链式调用
func (b *studentBuilder) Height (h int) *studentBuilder {
    b.stu.height = h
    return b
}

func (b *studentBuilder) Weight (w int) *studentBuilder {
    b.stu.weight = w
    return b
}

func (b *studentBuilder) Score (s float32) *studentBuilder {
    b.stu.score = s
    return b
}

// 最重要的一步：通过 Build 方法把 student 结构体返回出去
func (b *studentBuilder) Build () *student {
    return b.stu
}

// 把 student 的构建函数也代理过来，这一步可选可不选
func NewStudent(name string, age int) *studentBuilder {
    s := student{name: name, age: age}
    return &studentBuilder{stu: &s}
}
```

这样在创建 student 的时候可以这样调用：

```go
func main() {
    s1 := NewStudentBuilder("Boii", 18).Height(180).Weight(150).Score(100.0).Build()
    // 或者
    s2 := NewStudent("Boii", 18).Height(180).Weight(150).Score(100.0).Build()
}
```

简洁清晰！

## Function Option 模式

上面的 Builder 模式其实是设计模式中的一种创建型模式，在 Java 中可以使用内部类把 Builder 嵌在目标类中，但是在 Golang 就得多搞出一个结构体。但是使用 Function Option 模式则不需要多余的结构体，还可以发挥函数式编程的优点。

使用 Function Option 模式的步骤如下：

1.   自定义一个 Option 类型，以一个函数为基础，该函数接收一个目标结构体指针为参数
2.   逐一为目标结构体的每一个变量定义一个选项函数，接收变量的值，返回 Option 类型；在返回的 Option 中将变量的值赋给目标结构体
3.   定义目标结构体的构造函数，在构造函数中：
     -   参数：将 *必填* 选项作为普通参数，外加 Option 类型的可变参数
     -   返回值：目标结构体
     -   函数体：构造目标结构体对象，遍历可变参数并执行，最后返回目标结构体对象

```go
// 目标结构体：student
type student struct {
    name   string	 // *必填
    age    int		 // *必填
    height int		 // 选填
    weight int		 // 选填 
    score  float32	// 选填
}

// 自定义一个 Option 类型，以一个函数为基础，函数结构一个目标结构体 student 指针作为参数
type Option func(s *student) 

// 逐一为目标结构体的每一个变量定义一个选项函数，结构变量的值 n ，返回 Option 类型
func Name(n string) Option {
    return func(s *student) {	// 在返回的 Option 中将变量赋值
        s.name = n
    }
}

func Age(a int) Option {
    return func(s *student) {
        s.age = a
    }
}

func Height(h int) Option {
    return func(s *student) {
        s.height =h
    }
}

func Weight(w int) Option {
    return func(s *student) {
        s.weight = w
    }
}

func Score(sc float32) Option {
    return func(s *student) {
        s.score = sc
    }
}

// 定义目标结构体的构造函数
// 必填选项作为普通参数，Option 作为可变参数
// 返回目标结构体对象
func NewStudent(name string, age int, options ... Option) *student {
    s := student{		// 构造一个目标结构体对象 
        name:   name,
        age:    age,
        height: 180,
    }
    for _, option := range options {	// 遍历 options 得到传进来的一堆选项函数
        option(&s)		// 执行这些选项函数
    }
    return &s	// 返回目标结构体对象
}
```

这样我们就可以像这样创建目标结构体：

```go
func main() {
    s1 := NewStudent("Boii", 18, Height(177), Weight(150), Score(100.00))
    s2 := NewStudent("Boii", 18, Height(177), Weight(150))
    s3 := NewStudent("Boii", 18, Score(100.00))
}
```

## 结语

最推荐的还是 Function Option 模式，直接使用函数式编程，代码读起来也优雅。

它能带来一下好处（摘自[左耳朵耗子叔](https://coolshell.cn/articles/21146.html#%E9%85%8D%E7%BD%AE%E5%AF%B9%E8%B1%A1%E6%96%B9%E6%A1%88)）：

-   直觉式的编程
-   高度的可配置化
-   很容易维护和扩展
-   自文档
-   对于新来的人很容易上手
-   没有什么令人困惑的事（是nil 还是空）

