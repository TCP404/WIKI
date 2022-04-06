# 泛型

!!! tip
    2022年3月15日，go1.18发布，带来了泛型、模糊测试、工作区等新特性

    多了几个关键字：comparable、any，一个符号 `~`

## 泛型是啥

简单说，泛型就是某种类型，我们可以对“某种类型”加以限定。例如当一个函数接收一个参数，这个参数的类型只要支持相加即可，这在弱类型语言中很好解决，但是在强类型语言中就不太好确定，你可能需要写N个函数接收N中类型。

在以前 Go 语言的实践中，通常是用 `interface{} + 在函数中进行类型断言` 实现，现在有了泛型之后就简单多了。

## 泛型函数

### 非泛型的栗子

!!! example 
    ```go
    package main

    func SumInt(params []int) int {
        var sum int 
        for _, v := range params {
            sum += v
        }
        return sum
    }

    func SumFloat(params []float64) float64 {
        var sum float64
        for _, v := range params {
            sum += v
        }
        return sum
    }

    func main() {
        ints   := []int{1, 2, 4}
        floats := []float64{1.1, 2.2, 4.4}
        
        resultInt   := SumInt(ints)
        resultFloat := SumFloat(floats)
        
        fmt.Printf("Not-Generic: resultInt- %v, resultFloat- %v", resultInt, resultFloat)
    }

    // Output:
    Not-Generic: resultInt- 7, resultFloat- 7.7
    ```

上面的栗子要做的事情是接收一个切片，切片类型需要是可以相加的，但是只能写成两个函数去调用。

用泛型函数可以只写一个。

### 泛型的栗子

!!! example
    ```go
    package main

    func Sum[T int | float64](params []T) T {
        var sum T 
        for _, v := range params {
            sum += v
        }
        return sum
    }

    func main() {
        ints   := []int{1, 2, 4}
        floats := []float64{1.1, 2.2, 4.4}
        
        // 显式,指明类型的调用
        resultInt   := Sum[int](ints)
        resultFloat := Sum[float64](floats)
        
        // 隐式, 通过编译器自动推导类型
        // resultInt   := Sum(ints)
        // resultFloat := Sum(floats)
        
        fmt.Printf("Generic: resultInt- %v, resultFloat- %v", resultInt, resultFloat)
    }
    ```

改造步骤，在原有的基础上：

- 在 **函数名** 后 **参数列表** 前，通过 `[泛型名 类型列表]` 的方式指定此函数接受哪种类型，例如上面的 `[T int | float64]` 表示 Sum() 函数接受 int 和 float64 两种类型，并取名为 T。

    只要一个变量是 int 或 float64，那在这个函数里就是 T 类型，那么这个函数的作用域内都可以使用这个 T 类型去指代某一种类型。

- 然后在形参和返回值两个地方，使用 T 去替代原本的类型，表示接收 int 或 float64 类型的参数，并返回 int 或 float64 类型的返回值。

- 在调用的时候，放心大胆的传递符合类型定义的实参即可。

    在调用处，可以 **显式** 的写出传入的实参是类型列表中的哪种类型，如 `#!go Sum[int](ints)` 中的 `[int]`；
    
    也可以不写（**隐式**），编译器会自动推导类型。例如：`#!go resultInt := Sum(ints)`、`#!go resultFloat := Sum(floats)`。

### 小结

-   非泛型函数：

    声明：`#!go func 函数名 (形参列表) 返回值`

    调用：`#!go 函数名(实参列表)`

-   泛型函数：

    声明：`#!go func 函数名 [泛型名 类型列表] (形参列表) 返回值`

    调用：`#!go 函数名[类型](实参列表)` 或 `#!go 函数名(实参列表)`

-   自动推导不是万能的，如果泛型函数的形参列表为空，那么在调用时需要显式写明类型。

    ```go
    func Foo[T int | foat64]() T {
        i := getNumber()
        return i
    }
    
    ...
        num := Foo[int]()
    	// num := Foo(), 这种不行，没有形参编译器无法推导
    ...
    ```



## 泛型数据结构

### 泛型切片

泛型切片是这么定义的：

```go
type 自定义泛型切片名[泛型名 类型列表] []泛型名
```

??? example

    ```go
    type vector[T any] []T
    type vector[T int | float64] []T
    ```

那么在声明的时候是这么定义的：

```go
var 变量名 自定义泛型切片名[类型]
```

??? example

    ```go
    type vector[T any] []T

    func main() {
        var v1 vector[int]
        v1 = vector[int]{1, 2, 4}

        v2 := vector[string]{"a", "b", "d"}
    }
    ```

### 泛型 map

泛型 map 是这么定义的：

```go
type 自定义泛型map名[泛型名 类型列表] map[泛型名]泛型名 
```

??? example

    ```go
    type M[T any] map[string]T

    type M[T comparable] map[T]T
    type M[K comparable, V any] map[K]V
    ```

要注意一点就是，map 的 key 要求是可哈希的（Hashable），或者说可比较的，所以不可能出现这种泛型 map：`type M[T any] map[T]T`。这种是非法的，因为 any 表示任何类型，而 map 的 key 不是啥类型都可以。

那么在声明的时候是这样的：

```go
var 变量名 自定义泛型map名[key的类型, value的类型]
```

??? example

    ```go
    type M[K comparable, V any] map[K]V

    var m1 M[string, float64]
    m1 = make(M[string, float64])

    m1["a"] = 1.1
    m1["b"] = 2.2
    ```

### 泛型通道

泛型通道这么定义：

```go
type 自定义泛型通道名[泛型名 类型列表] chan 泛型名
```

??? example

    ```go
    type C[T any] chan T
    type C[T int | bool] chan T
    type C[T comparable] chan T
    ```

在使用的时候是这样的：

```go
var 变量名 自定义泛型通道名[类型]
```

??? example

    ```go
    type C[T any] chan T

    c1 := make(C[int],2)

    c1 <- 1
    c2 <- 2

    c2 := make(C[byte], 2)

    c2 <- 'a'
    c2 <- 'b'
    ```



### 泛型结构体

泛型结构体是这么定义的：

```go
type 自定义泛型结构体名[泛型名 类型列表] struct {
    字段名 字段类型
    ...
}
```

??? example

    ```go
    type Tree[T any] struct {
        data T
        l, r *Tree[T]
    }
    ```

这样就定义了一个接受任意类型的二叉树的结点。

在使用的时候是这样的：

```go
var 变量名 自定义泛型结构体名[类型]
```

??? example

    ```go
    type Tree[T any] struct {
        data T
        l, r *Tree[T]
    }

    var leaf1, leaf2 Tree[int]
    root := Tree[int]{
        data: 1,
        l:    &leaf1,
        r:    &leaf2,
    }

    leaf1 = Tree[int]{
        data: 2,
        l:    nil,
        r:    nil,
    }

    leaf2 = Tree[int]{
        data: 3,
        l:    nil,
        r:    nil,
    }
    ```



### 小结

可以发现，定义泛型数据结构的时候，都是先是自定义的数据结构名，然后旁边加上类型约束；在声明泛型数据结构的时候，都是自定义的数据结构名，然后旁边加上指定类型。

```go
type 自定义泛型切片名[泛型名 类型列表] []泛型名
type 自定义泛型map名[泛型名 类型列表] map[泛型名]泛型名 
type 自定义泛型通道名[泛型名 类型列表] chan 泛型名
type 自定义泛型结构体名[泛型名 类型列表] struct {
    ...
    字段名 字段类型
}

var 变量名 自定义泛型切片名[类型]
var 变量名 自定义泛型map名[key的类型, value的类型]
var 变量名 自定义泛型通道名[类型]
var 变量名 自定义泛型结构体名[类型]
```

## 泛型约束

上面举了泛型函数，还有泛型的数据结构怎么定义和声明使用，下面我们关注泛型的约束。

对比泛型函数和非泛型函数，就是差了个类型列表而已，他们的定义大概是这样的：

```go
[X, Y constraint1, Z constraint2]
```

这个叫 **类型参数列表（type paramter list）**，

-   用中括号 `[ ]` 包裹
-   X，Y，Z 都是类型参数，或者叫泛型名，建议采用大驼峰命名法，体现它是一个类型
-   constraint1，constraint2 都是类型约束（type constraint），或者叫类型列表、类型集

在使用的时候，除了上面那些用法，还有其他用法，下面慢慢说。

举个栗子，现在要写一个 max 函数，

-   非泛型写法：

    ```go
    func max(a, b int) int {
        if a > b {
            return a
        }
        return b
    }
    ```

    这样只接收 int 类型，如果是 int8、int64 这些也不能传进来

-   泛型写法：

    ```go
    func max[T int | int8 | int16 | int32 | int64](a, b T) T {
        if a > b {
            return a
        }
        return b
    }
    ```

    这样就可以传 int 系列类型的参数进去。

    但是有个问题，如果还要加上 uint 系列呢，接着列出来？很丑！！！

    那么可以用 interface 的方式去定义。

-   interface 的方式

    ```go
    type Num interface{
        int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 | float32 | float64
    }
    
    func max[T Num](a, b T) T {
        if a > b {
            return a
        }
        return b
    }
    
    func min[T Num](a, b T) T {
        if a < b {
            return a
        }
        return b
    }
    
    // 调用
    func main() {
        max(1, 2)
        max(1.0, 2.2)
    }
    ```

    可以看到泛型函数的定义一下子变得简洁，而且 Num 这个 interface 还能复用给 min 函数。

    但是有个问题，如果我基于 int 类型自定义了一个类型，底层类型也是 int，但是编译不通过呀。

    没事，用 `~` 符号，这个符号跟类型一起，表示底层类型是该类型的都能接受。例如 `~int` 表示 int 和底层类型是 int 的所有类型。

-   `~` 符号

    ```go
    type Num interface {
    	~int | ~int8 | ~int16 | ~int32 | ~int64 |
    	~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    	~float32 | ~float64
    }
    
    func max[T Num](a, b T) T {
        if a > b {
            return a
        }
        return b
    }
    
    func min[T Num](a, b T) T {
        if a < b {
            return a
        }
        return b
    }
    
    // 调用
    type MyInt int
    
    func main() {
        var a, b MyInt = 1, 2
        max(a, b)
        max(1, 2)
    }
    ```

    在调用处，先是定义了一个基于 int 的自定义类型 MyInt，然后声明了两个自定义类型的参数，一样可以传进去。

    但是我懒，我不想手动写，那就用 constraints 包。
    
-   constraints 包
    
    ```go
    import (
    	"golang.org/x/exp/constraints"
        ...
    )
    
    func max[T constraints.Order](a, b T) T {
        if a > b {
            return a
        }
        return b
    }
    
    func min[T constraints.Order](a, b T) T {
        if a < b {
            return a
        }
        return b
    }
    ```
    
    constraints 是官方给出的一个很简单的包，主要就是定义了几个泛型，他们的关系如下图。不过这个包在 Go 1.18 正式发布时并没有带上。使用命令 `go get golang.org/x/exp/constraints ` 可以下载到。
    
    ![image-20220316234847716](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/package-constraints.png)
    



上面的栗子引出许多个跟泛型相关的点：

-   使用 `|` 定义一个类型集
-   使用 `~T` 接受所有底层类型为 T 的类型
-   使用 interface 方式使得泛型约束可以复用
-   使用 constraints 包可以省去定义一些基础类型的泛型的工作

下面继续补充一下 interface 方式、关键字 comparable 和 any、类型约束字面值。

### interface 方式

简单的 interface 约束

```go
type Signed interface{
    ~int | ~int8 | ~int16 | ~int32 | ~int64
}
```

Go 1.18以前定义接口

```go
type Stringer interface{
    String()
}
```

在定义 interface 约束的时候是可以加上方法集的，因为 Go 中 interface 定义的是一种类型，那么上栗子中，

-   Signed 是基础类型为 int 系列类型的一种类型；
-   Writer 是实现了 Write 方法的一种类型；

两个都是类型，它们可以同时用：

```go
type Foo interface{
    ~int | ~int8 | ~int16 | ~int32 | ~int64
    String()
}

func AddAndPrint[T Foo](param T) {
    param++
    param.String()
}
```

### 关键字 comparable 和 any

Go 1.18 新增了两个关键字 comparable 和 any，他们定义如下：

```go
// any is an alias for interface{} and is equivalent to interface{} in all ways.
type any = interface{}

// comparable is an interface that is implemented by all comparable types
// (booleans, numbers, strings, pointers, channels, arrays of comparable types,
// structs whose fields are all comparable types).
// The comparable interface may only be used as a type parameter constraint,
// not as the type of a variable.
type comparable interface{ comparable }
```

any 只是 interface{} 的别名，所以以后写代码时如果需要用到 interface{} 这种类型时，可以直接用 any 替代。

comparable 是一些可以用 `==` 和 `!=` 比较的类型（不包括 `>`、`>=`、`<`、`<=` 这些哦），可以是布尔类型、数值类型、字符串类型、指针类型、通道类型、comparable类型的数组、可比较的结构体（字段中没有slice、map类型）。

comparable 只能用在泛型约束中，不能用作声明一个变量的类型。



### 类型约束字面值

```go
[S interface{~[]E}, E interface{}]

[S ~[]E, E interface{}]

[S ~[]E, E any]
```

类型约束可以提前定义好，也可以在约束列表中定义，例如上面的三个类型约束，虽然读取到 S 的时候不知道 E 是什么类型，在类型约束结束前就读取到 E 是 interface{}，所以这样的类型约束是可以的，而 E 就是**类型约束字面值**。

在类型限制的位置，`interface{E}`也可以直接写为`E`，因此就可以理解`interface{~[]E}`可以写为`~[]E`。

## 使用场景

-   需要使用 slice、map、channel 类型时，但是 slice、map、channel 中的元素可能有多种；

-   定义一些比较通用的数据结构时，比如通用的链表、树等等；

-   当一个方法的实现对所有某些类型都一样时。

-   不要为了泛型而使用泛型

    ```go
    // good
    func foo(w io.Writer) {
       b := getBytes()
       _, _ = w.Write(b)
    }
    
    // bad
    func foo[T io.Writer](w T) {
       b := getBytes()
       _, _ = w.Write(b)
    }
    ```

    单纯是调用`io.Writer`的`Write`方法，把内容写到指定地方。使用`interface`作为参数更合适，可读性更强。


!!! note ""
    Avoid boilerplate.
 
    Corollary: Don't use type parameters prematurely; wait until you are about to write boilerplate code.

@Ian 给的建议是：当你发现针对不同类型，会写出同样的代码逻辑时，才去使用泛型。也就是 `Avoid boilerplate code`。