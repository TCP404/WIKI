# reflect

什么是反射？

官方 Doc 中 **Rob Pike** 定义：
!!! cite
    Reflection in computing is the ability of a program to examine its own structure, particularly through types; it’s a form of meta programming. It’s also a great source of confusion.

    在计算机领域，反射是一种让程序——主要是通过类型——理解其自身结构的一种能力。它是元编程的组成之一，同时它也是一大引人困惑的难题。

**维基百科** 定义：
!!! cite
    在计算机科学中，反射是指计算机程序在运行时（Run time）可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，反射就是程序在运行的时候能够“观察”并且修改自己的行为。

**《Go 语言圣经》** 中这样定义：
!!! cite
    Go 语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法，但是在编译时并不知道这些变量的具体类型，这称为反射机制。

- 反射应用场景
    1. 由于没约定好，或者传入的类型很多、这些类型不能统一表示，导致不确定函数参数类型，就需要用反射来确定类型。
    2. 需要根据某些条件，如用户的输入，来决定调用哪个函数，则需要在运行期动态地执行函数。
- 不建议用反射的理由
    1. 反射代码难以阅读，降低代码可读性。
    2. 静态语言的优势之一在于编译期就能发现一些类型错误，而反射这种运行期的怪物会导致这一优势的失去。包含反射的代码很可能运行很久，才会出错，而且经常是直接 `panic`，造成严重后果。
    3. 反射对性能影响较大，相比正常代码慢出一两个数量级。

## 相关基础

### Golang 的类型
Golang 的类型：

- 变量包括`（type，value）`两部分
- *type* 包括 *static type* 和 *concrete type*。
    - *static type* 就是在编码时看得见的类型，如：int、string
    - *concrete type* 是 runtime 系统看得见的类型。
- 类型能否断言成功，取决于变量的 *concrete type*。因此，一个 reader 变量如果它的 *concrete type* 实现了 write 方法，reader 也可以被断言为 writer。

### 静态类型和动态类型
在反射的概念中，编译时就知道的变量类型叫 **静态类型**，运行时才知道的变量类型叫 **动态类型**。

- 静态类型：变量声明时赋予的类型。
    ```go
    type Myint int    // int 就是静态类型，type 是 static type
    
    type A struct {
        Name string   // string 就是静态类型，type 是 static type
    }
    
    var i *int        // *int 就是静态类型，type 是 static type
    ```

- 动态类型：运行时给这个变量赋值时，这个值的类型。如果值为 `nil` 则没有动态类型。
    一个变量的动态类型在运行时可能改变，则主要依赖于它的赋值，前提是这个变量是接口类型。
    ```go
    var A interface{}    // 静态类型 interface{}
    A = 10               // 静态类型 interface{}    动态类型 int
    A = "String"         // 静态类型 interface{}    动态类型 string
    var M *int
    A = M                // 静态类型 interface{}    动态类型 *int
    ```

!!! info
    Golang 的反射就是建立在类型之上的。

    Golang 变量的指定类型是静态的，在创建变量的时候类型就已经确定，如指定 `int、string`，所以称之为 *static type*。
    
    Golang 变量的指定类型是 interface 类型相关的，则类型在运行期才能确定，所以称之为 *concrete type*。

    只有 interface 类型才有反射一说。


### interface 类型
Golang 是通过接口实现的，任何 **接口值** 都是由一个 **实际类型** 和 **实际类型的值** 两个部分组成的。

每个 interface 变量都有一个对应的 *pair*，*pair* 中记录了 type 的类型和值：
```
pair -> (concrete type, value)
```
`concrete type` 是实际变量的类型，`concrete value` 是实际变量的值。
一个 interface 变量包含了 2 个指针，一个指向实际值的类型，一个指向实际值。

- eg：打开文件会返回一个 `*os.File` 变量。
    ```go
    f, err := os.OpenFile("/home/abc.txt", os.O_RDWR, 0777)
    var r io.Reader = f
    ```
    接口变量 r 的 *pair* 信息记录如下：`(f, *os.File)`。

    *pair* 在接口变量管道连续赋值过程中是不变的，将接口变量 r 赋值给另一个接口变量 w，他们的 *pair* 是相同的。
    ```go
    var w io.Writer = r.(io.Writer)
    ```
    接口变量 w 的 *pair* 信息记录如下：`(f, *os.File)`。即使 w 是空接口类型， pair 也是不变的。

!!! info
    interface及其pair的存在，是Golang中实现反射的前提，理解了pair，就更容易理解反射。

    反射就是用来检测存储在接口变量内部`(类型concrete type, 值value)` pair对的一种机制。


## 反射的使用
- reflect包提供了几个重要类型和函数：
    - **reflect.Type接口** 表示接口值的"具体类型"；`reflect.TypeOf()`函数返回类型的名称，也就是 *pair* 中的 type
    - **reflect.Value结构** 表示接口值的"具体类型的值"；`reflect.ValueOf()`函数返回具体类型的值，也就是 *pair* 中的 value

1. 先有一个接口类型的变量
2. 把接口变量转化成 `reflect` 对象（`reflect.Type` 或 `reflect.Value`）
3. 根据不同情况调用不同的函数

最简单的基本使用：
```go
package main

import (
    "fmt"
    "reflect"
)
func main() {
    var x float64 = 1.23

    t := reflect.TypeOf(x)	/* 实例到type */
    v := reflect.ValueOf(x)	/* 实例到value */
    fmt.Println(t)         // float64
    fmt.Println(v)         // 1.23
}
```

### reflect.Type
??? note
    ```go
    type Type interface {
        // Kind返回该接口的具体分类
        Kind() Kind

        // Name返回该类型在自身包内的类型名，如果是未命名类型会返回""
        Name() string

        // PkgPath返回类型的包路径，即明确指定包的import路径，如"encoding/base64"
        // 如果类型为内建类型(string, error)或未命名类型(*T, struct{}, []int)，会返回""
        PkgPath() string

        // 返回类型的字符串表示。该字符串可能会使用短包名（如用base64代替"encoding/base64"）
        // 也不保证每个类型的字符串表示不同。如果要比较两个类型是否相等，请直接用Type类型比较。
        String() string

        // 返回要保存一个该类型的值需要多少字节；类似unsafe.Sizeof
        Size() uintptr

        // 返回当从内存中申请一个该类型值时，会对齐的字节数
        Align() int

        // 返回当该类型作为结构体的字段时，会对齐的字节数
        FieldAlign() int


    /* 判断相关 */
        // 如果该类型实现了u代表的接口，会返回真
        Implements(u Type) bool

        // 如果该类型的值可以直接赋值给u代表的类型，返回真
        AssignableTo(u Type) bool

        // 如该类型的值可以转换为u代表的类型，返回真
        ConvertibleTo(u Type) bool


    /* 类型相关 */
        // 返回该类型的字位数。如果该类型的Kind不是Int、Uint、Float或Complex，会panic
        Bits() int

        // 返回array类型的长度，如非数组类型将panic
        Len() int

        // 返回该类型的元素类型，如果该类型的Kind不是Array、Chan、Map、Ptr或Slice，会panic
        Elem() Type

        // 返回map类型的键的类型。如非映射类型将panic
        Key() Type

        // 返回一个channel类型的方向，如非通道类型将会panic
        ChanDir() ChanDir


    /* 结构体相关 */
        // 返回struct类型的字段数（匿名字段算作一个字段），如非结构体类型将panic
        NumField() int

        // 返回struct类型的第i个字段的类型，如非结构体或者i不在[0, NumField())内将会panic
        Field(i int) StructField

        // 返回索引序列指定的嵌套字段的类型，
        // 等价于用索引中每个值链式调用本方法，如非结构体将会panic
        FieldByIndex(index []int) StructField

        // 返回该类型名为name的字段（会查找匿名字段及其子字段），
        // 布尔值说明是否找到，如非结构体将panic
        FieldByName(name string) (StructField, bool)

        // 返回该类型第一个字段名满足函数match的字段，布尔值说明是否找到，如非结构体将会panic
        FieldByNameFunc(match func(string) bool) (StructField, bool) {}

        // 返回该类型的方法public方法的数目
        // 匿名字段的方法会被计算；主体类型的方法会屏蔽匿名字段的同名方法；
        // 匿名字段导致的歧义方法会滤除
        NumMethod() int

        // 返回该类型方法集中的第i个方法，i不在[0, NumMethod())范围内时，将导致panic
        // 对非接口类型T或*T，返回值的Type字段和Func字段描述方法的未绑定函数状态
        // 对接口类型，返回值的Type字段描述方法的签名，Func字段为nil
        Method(int) Method

        // 根据方法名返回该类型方法集中的方法，使用一个布尔值说明是否发现该方法
        // 对非接口类型T或*T，返回值的Type字段和Func字段描述方法的未绑定函数状态
        // 对接口类型，返回值的Type字段描述方法的签名，Func字段为nil
        MethodByName(string) (Method, bool)


    /* 函数相关 */
        // 如果函数类型的最后一个输入参数是"..."形式的参数，IsVariadic返回真
        // 如果这样，t.In(t.NumIn() - 1)返回参数的隐式的实际类型（声明类型的切片）
        // 如非函数类型将panic
        IsVariadic() bool

        // 返回func类型的参数个数，如果不是函数，将会panic
        NumIn() int

        // 返回func类型的返回值个数，如果不是函数，将会panic
        NumOut() int

        // 返回func类型的第i个参数的类型，如非函数或者i不在[0, NumIn())内将会panic
        In(i int) Type

        // 返回func类型的第i个返回值的类型，如非函数或者i不在[0, NumOut())内将会panic
        Out(i int) Type
    }
    ```

**Kind** 有 `slice、map、ptr、struct、interface、string、Array、Function、int` 或其他基本类型组成。

**Kind** 和 **Type** 之间要做好区分。

如：在 `main()` 中定义 `type Person struct {}`，那么 **Kind** 就是 `struct`，**Type** 就是 `main.Person`。

```go
type Person struct {
    name string
    age  int
}

func main() {
    pers := person{"Boii", 18}
    t := reflect.TypeOf(pers)
    v := reflect.ValueOf(pers)
    fmt.Println(v)             // {Boii 18}
    fmt.Println(t)             // main.person
    fmt.Println(t.Name())      // person
    fmt.Println(t.Kind())      // struct
}
```

### reflect.Value

??? note

    ```go
    // 类型相关
    func (v Value) Addr() Value
    func (v Value) Bool() bool
    func (v Value) Bytes() []byte
    func (v Value) Complex() complex128
    func (v Value) Int() int64
    func (v Value) Slice(i, j int) Value
    func (v Value) Slice3(i, j, k int) Value
    func (v Value) String() string
    func (v Value) Uint() uint64
    func (v Value) Pointer() uintptr
    func (v Value) UnsafeAddr() uintptr
    func (v Value) Kind() Kind
    func (v Value) Type() Type
    func (v Value) InterfaceData() [2]uintptr


    // 函数相关
    func (v Value) Call(in []Value) []Value
    func (v Value) CallSlice(in []Value) []Value


    // 判断相关
    func (v Value) CanAddr() bool
    func (v Value) CanInterface() bool
    func (v Value) CanSet() bool
    func (v Value) IsNil() bool
    func (v Value) IsValid() bool
    func (v Value) IsZero() bool
    func (v Value) OverflowComplex(x complex128) bool
    func (v Value) OverflowFloat(x float64) bool
    func (v Value) OverflowInt(x int64) bool
    func (v Value) OverflowUint(x uint64) bool


    // 转换相关
    func (v Value) Convert(t Type) Value
    func (v Value) Interface() (i interface{})
    func (v Value) Elem() Value


    // struct 相关
    func (v Value) NumField() int
    func (v Value) NumMethod() int
    func (v Value) Field(i int) Value
    func (v Value) FieldByIndex(index []int) Value
    func (v Value) FieldByName(name string) Value
    func (v Value) FieldByNameFunc(match func(string) bool) Value
    func (v Value) Method(i int) Value
    func (v Value) MethodByName(name string) Value


    // Array、Slice 相关
    func (v Value) Index(i int) Value


    // Map 相关
    func (v Value) MapIndex(key Value) Value
    func (v Value) MapKeys() []Value
    func (v Value) MapRange() *MapIter


    // Array、Slice、Map 相关
    func (v Value) Cap() int
    func (v Value) Len() int


    // 通道相关
    func (v Value) Recv() (x Value, ok bool)
    func (v Value) Send(x Value)
    func (v Value) TryRecv() (x Value, ok bool)
    func (v Value) TrySend(x Value) bool
    func (v Value) Close()


    // 改值相关
    func (v Value) Set(x Value)
    func (v Value) SetBool(x bool)
    func (v Value) SetBytes(x []byte)
    func (v Value) SetCap(n int)
    func (v Value) SetComplex(x complex128)
    func (v Value) SetFloat(x float64)
    func (v Value) SetInt(x int64)
    func (v Value) SetLen(n int)
    func (v Value) SetMapIndex(key, elem Value)
    func (v Value) SetPointer(x unsafe.Pointer)
    func (v Value) SetString(x string)
    func (v Value) SetUint(x uint64)
    ```


`reflect`中的结构体主要包括：*Type，Value，ChanDir，Kind，MapIter，Method，SelectCase，SelectDir，SliceHeader，StringHeader，StructField，StructTag，ValueError* 等。其中，Type 和 Value 之前已经介绍过了。

- `ChanDir`：管道的方向，有三个值：**RecvDir / SendDir / BothDir**，分别为 **接受，发送，双向**；
- `Kind`：Type中的类型信息，包括：Invalid, Bool, Int, Int8, Int16, Int32, Int64, Uint, Uint8, Uint16, Uint32, Uint64, Uintptr, Float32, Float64, Complex64, Complex128, Array, Chan, Func, Interface, Map, Ptr, Slice, String, Struct, UnsafePointer，
- `MapIter`：Map的迭代器，包括三个方法：**Key、Value、Next**；
- `Method`：描述方法的信息，包括：**方法名，包路径，类型，函数，所处的下标**；
- `SelectCase`：描述select 操作的信息，**case的方向SelectDir，使用的Channel，发送的值Send**；
- `SelectDir`：描述SelectCase中的方向，有三个值：**SelectSend / SelectRecv / SelectDefault**
- `SliceHeader`：描述切片Slice的信息，包括**指针，长度，容量**；
- `StringHeader`：描述字符串string的信息，包括**指针，长度**；
- `StructField`：描述结构体中的域field中的信息，包括：**域名，包路径，类型，标签Tag，在结构体中的偏移量offset，Type.FieldByIndex中的下标index，是否是匿名**；
- `StructTag`：描述标签信息，有两个方法：**Get、Lookup**；
- `ValueError`：在调用一个Value不支持的方法时会报错，并记录到ValueError中。


## 反射API

![](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210213215422285_5740.png)

上图很好的概括了 所有的API

`Type` 表示 `reflect` 的 `Type` 类型： `reflect.Type`
`Value` 表示 `reflect` 的 `Value` 类型： `reflect.Value`
`interface{}` 表示接口实例、任意类型
`Special Type` 表示其他特殊类型

- **Type -> Value**：通过 `reflect` 包的静态方法 `New()、NewAt()、Zero()`
- **Value -> Type**：通过 `reflect对象`的 `Type()` 方法
- **interface{} -> Type**：通过 `reflect` 包的静态方法 `TypeOf()`
- **interface{} -> Value**：通过 `reflect` 包的静态方法 `ValueOf()`
- **Value -> interface{}**：通过  `reflect对象`的 `Interface()` 方法，得到是一个静态类型


### 从 接口实例 获取 Type

**interface{} -> Type**：通过 `reflect` 包的静态方法 `TypeOf()``

```go
func TypeOf(i interface{}) Type
```

!!! example

    ```go
    var x int = 10
    t := reflect.TypeOf(x)

    fmt.Printf("t的值：%v   \n", t)    // int
    fmt.Printf("t的类型：%T \n", t)    // *reflect.rtype
    ```

### 从 接口实例 获取 Value

**interface{} -> Value**：通过 `reflect` 包的静态方法 `ValueOf()`

```go
func ValueOf(i interface{}) Value
```

!!! example

    ```go
    var x int = 10
    v := reflect.ValueOf(x)

    fmt.Printf("t的值：%v \n", v)	// 10
    fmt.Printf("t的类型：%T \n", v)  // reflect.Value
    ```

### 从 Type 获取 Value

**Type -> Value**：通过 `reflect` 包的静态方法 `New()、NewAt()、Zero()`

Type 里面只有类型信息，所以直接从一个 Type 接口变量里面是无法获得实例的 Value 的，但可以通过该 Type 构建一个新实例的 Value。

- New 返回的是一个 Value，该 Value 的 type 为 PtrTo(typ)，即 Value 的 Type 是指定 typ 的指针类型
    ```go
    func New(typ Type) Value
    ```

- Zero 返回的是一个 typ 类型的零值，注意返回的 Value 不能寻址，位不可改变
    ```go
    func Zero(typ Type) Value
    ```

- 如果知道一个类型值的底层存放地址，则还有一个函数是可以依据 type 和该地址值恢复出 Value 的：
    ```go
    func NewAt(typ Type, p unsafe.Pointer) Value
    ```

!!! example

    ```go
    var x int = 10
    t := reflect.TypeOf(x)
    newX := reflect.New(t)     // Type -> Value
    zeroX := reflect.Zero(t)   // Type -> Value

    fmt.Println(newX)          // 0xc000012090
    fmt.Printf("%v \n", newX)  // 0xc000012090
    fmt.Printf("%T \n", newX)  // reflect.Value

    fmt.Println(zeroX)         // 0
    fmt.Printf("%v \n", zeroX) // 0
    fmt.Printf("%T \n", zeroX) // reflect.Value
    ```



### 从 Value 获取 Type

**Value -> Type**：通过 `reflect对象`的 `Type()` 方法

```go
func (v Value) Type() Type
```

!!! example
    ```go
    var x int = 10
    v := reflect.ValueOf(x)

    t := v.Type()                     // value -> type
    fmt.Printf("t的值：%v   \n", t)    // int
    fmt.Printf("t的类型：%T \n", t)    // *reflect.rtype
    ```

### 从 Value 获取 接口实例

**Value -> interface{}**：通过  `reflect对象`的 `Interface()` 方法，得到是一个静态类型

```go
//该方法最通用，用来将 Value 转换为空接口，该空接口内部存放具体类型实例
//可以使用接口类型查询去还原为具体的类型
func (v Value) Interface() （i interface{})

//Value 自身也提供丰富的方法，直接将 Value 转换为简单类型实例，如果类型不匹配，则直接引起 panic
func (v Value) Bool () bool
func (v Value) Float() float64
func (v Value) Int() int64
func (v Value) Uint() uint64
```

!!! example
    eg1：

    ```go
    var x int = 10
    v := reflect.ValueOf(x)

    iInt := v.Interface()
    vType := iInt.(int)
    vInt := v.Int()

    fmt.Println(iInt)            // 10
    fmt.Printf("%T \n", iInt)    // int
    fmt.Println(vType)           // 10
    fmt.Printf("%T \n", vType)   // int


    fmt.Println(vInt)            // 10
    fmt.Printf("%T \n", vInt)    // int64
    ```

    eg2：

    ```go
    package main

    import (
        "fmt"
        "reflect"
    )
    func main() {
        var x float64 = 1.23

        t := reflect.TypeOf(x)	// 获得 reflect.Type  对象
        v := reflect.ValueOf(x)	// 获得 reflect.Value 对象
        fmt.Println(t)          // float64
        fmt.Println(v)          // 1.23
        fmt.Println(v.Kind())   // float64
        fmt.Println(v.Type())   // float64	/* value 到 type */
        fmt.Println(v.Float())  // 1.23	    /* value 到 实例 */

        /* value 到 实例 */
        v = reflect.ValueOf(x)                 // 接口类型变量 -> 反射类型对象
        convertV := v.Interface().(float64)    // 反射类型对象 -> 接口类型变量, 可以理解为 强制转换
        fmt.Println(convertV)    // 1.23

        p := reflect.ValueOf(&x)
        convertP := p.Interface().(*float64)
        fmt.Println(convertP)    // 0xc000012090
    }
    ```

### 指针类型 与 值类型的转换

- **指针 -> 值**：`t.Elem()`，t 只可以是引用类型
- **指针 -> 值**：`v.Elem()`，v 只可以是指针

    ```go
    // t 必须是 Array、Chan、Map、Ptr、Slice，否则会引起 panic
    // Elem 返回的是其内部元素的 Type
    func (t *rtype) Elem() Type
    ```

    !!! example
        eg：

        ```go
        func main() {
            var x int = 10
            var s []int = []int{10, 20, 30}
        
        // TypeOf：只接受引用类型
            t1 := reflect.TypeOf(s)
            te1 := t1.Elem()
            fmt.Println(te1)         // int
            fmt.Printf("%T \n", te1) // *reflect.rtype
        
            t2 := reflect.TypeOf(&s)
            te2 := t2.Elem()
            fmt.Println(te2)         // []int
            fmt.Printf("%T \n", te2) // *reflect.rtype
        
            t3 := reflect.TypeOf(x)
            te3 := t3.Elem() // panic: reflect: Elem of invalid type int


            t4 := reflect.TypeOf(&x)
            te4 := t4.Elem()
            fmt.Println(te4)         // int
            fmt.Printf("%T \n", te4) // *reflect.rtype


        // ValueOf：只接受prt类型
            v1 := reflect.ValueOf(s)
            ve1 := v1.Elem() // panic: reflect: call of reflect.Value.Elem on slice Value


            v2 := reflect.ValueOf(&s)
            ve2 := v2.Elem()
            fmt.Println(ve2)         // [10 20 30]
            fmt.Printf("%T \n", ve2) // reflect.Value
        
            v3 := reflect.ValueOf(x)
            ve3 := v3.Elem() // panic: reflect: call of reflect.Value.Elem on int Value


            v4 := reflect.ValueOf(&x)
            ve4 := v4.Elem()
            fmt.Println(ve4)         // 10
            fmt.Printf("%T \n", ve4) // reflect.Value
        }
        ```


- **值 -> 指针**：`PtrTo(t)`
    ```go
    // PtrTo 返回的是指向 t 的指针型 Type
    func PtrTo(t Type) Type
    ```

    !!! example
        eg：

        ```go
        func main() {
            var x int = 10
            var s []int = []int{10, 20, 30}
        
            xt := reflect.TypeOf(x)
            st := reflect.TypeOf(s)
        
            px := reflect.PtrTo(xt)
            ps := reflect.PtrTo(st)
        
            fmt.Println(px) // *int
            fmt.Println(ps) // *[]int
        }
        ```

### Value 的可修改性
```go
//通过 CanSet 判断是否能修改
func (v Value) CanSet() bool {}
//通过 Set 进行修改
func (v Value) Set(x Value) {}
func (v Value) SetBool(x bool) {}
func (v Value) SetBytes(x []byte) {}
func (v Value) SetCap(n int) {}
func (v Value) SetComplex(x complex128) {}
func (v Value) SetFloat(x float64) {}
func (v Value) SetInt(x int64) {}
func (v Value) SetLen(n int) {}
func (v Value) SetMapIndex(key, elem Value) {}
func (v Value) SetPointer(x unsafe.Pointer) {}
func (v Value) SetString(x string) {}
func (v Value) SetUint(x uint64) {}
```

!!! example
    ```go
    func main() {
        var x int = 10
        var s []int = []int{10, 20, 30}

        vpx := reflect.ValueOf(&x)
        vepx := vpx.Elem()
        if vepx.CanSet() == true {    // CanSet 判断
            vepx.SetInt(20)           // Set 设置
        }
        fmt.Println(x) // 20

        vps := reflect.ValueOf(&s)
        veps := vps.Elem()
        if veps.CanSet() == true {    // CanSet 判断
            veps.Index(1).SetInt(345) // Index 获取元素，然后设置
        }
        fmt.Println(s) // [10 345 30]
    }
    ```

### 反射结构体
结构体相比基本类型稍微多一点步骤。

- 第一步，传入结构体指针，获取可修改的 `reflect` 对象
- 第二步，使用 `Elem()` 获得字段列表
- 第三步，使用 `Field()`系列方法 或 `Method()` 系列方法，获得指定字段或方法
- 第四步，使用 `Set()`系列方法修改数值 或 使用 `Call()` 调用结构体方法

结构体的 `Type` 是 `main.person`

结构体的 `Kind` 是 `struct`

结构体的 `Name` 是 `person`


- `NumField()` 可以获得结构体字段的数量
- `NumMethod()` 可以获得结构体方法的数量，但只能获取 public 方法
- `Field(idx)` 可以通过下标获得指定的字段
- `FieldByName(fieldName)` 可以通过字段名字获得指定的字段
- `Method(idx)` 可以通过下标获得指定的方法
- `MethodByName(methodName)` 可以通过方法名获得指定的方法
- `NumIn()` 可以获得参数个数
- `NumOut()` 可以获得返回值个数

=== "访问操作"
    eg：访问结构体的字段和方法

    ```go
    package main

    import (
        "fmt"
        "reflect"
    )

    type person struct {
        name string
        Age  int
    }

    func (p person) Eat() {
        fmt.Println("eat")
    }

    func (p person) Run() {
        fmt.Println("run")
    }

    func (p person) gogogo() {
        fmt.Println("go")
    }

    func main() {

        pers := person{"Boii", 18}
        t := reflect.TypeOf(pers)
        v := reflect.ValueOf(pers)
        fmt.Println(v)             // {Boii 18}
        fmt.Println(t)             // main.person
        fmt.Println(t.Name())      // person
        fmt.Println(t.Kind())      // struct

        // 获取字段
        // { Name   PkgPath  Type   Tag  Offset Index  Anonymous}
        // { name    main   string         0     [0]     false}
        // { Age     main    int           16    [1]     false}
        for i := 0; i < t.NumField(); i++ {
            field := t.Field(i)
            value := v.Field(i)
            fmt.Printf("字段名称: %s\t字段类型: %s\t字段数值: %v\n", field.Name, field.Type, value)
        }
        // 字段名称: name  字段类型: string 字段数值: Boii
        // 字段名称: Age   字段类型: int    字段数值: 18


        for i := 0; i < v.NumField(); i++ {
            fmt.Println(v.Field(i))
            // Boii
            // 18
        }


        // 获取方法，t.NumMethod() 只能获取到 public 方法
        // {Name  PkgPath           Type                          Func                  Index}
        // {Eat            func(main.person, string)  <func(main.person, string) Value>   0}
        // {Run            func(main.person)          <func(main.person) Value>           1}
        for i := 0; i < t.NumMethod(); i++ {
            method := t.Method(i)
            fmt.Printf("方法名称: %s\t方法类型: %v\n", method.Name, method.Type)
        }
        // 方法名称: Eat   方法类型: func(main.person)
        // 方法名称: Run   方法类型: func(main.person)


        for i := 0; i < v.NumMethod(); i++ {
            fmt.Println(v.Method(i))
            // 0x433600
            // 0x433600
        }
    }
    ```

=== "修改操作"

    如果想要对结构体的字段等进行修改，需要传入结构体指针来获得 `reflect` 对象。

    ```go
    package main

    import (
        "fmt"
        "reflect"
    )

    type person struct {
        name string
        Age  int
    }

    func (p person) Eat(food string) {
        fmt.Println("eat", food)
    }

    func (p person) Run() {
        fmt.Println("run")
    }

    func (p person) go() {
        fmt.Println("go")
    }

    func main() {
        pers := person{"Boii", 18}
        v := reflect.ValueOf(&pers).Elem()

        // panic: reflect: reflect.Value.SetString using value obtained using unexported field
        v.FieldByName("name").SetString("Eva")    // 传入结构体指针以后，Field 只能访问到 public 字段

        v.Field(1).SetInt(20)
        fmt.Println(pers) // {Boii 20}

        // 调用有参方法
        v.Method(0).Call([]reflect.Value{reflect.ValueOf("apple")}) // eat apple
        // 调用无参方法
        v.Method(1).Call(nil)                                       // run
        v.Method(1).Call(make([]reflect.Value, 0))                  // run
    }
    ```

    通过结构体指针获得的 `reflect` 对象，要注意2点

    1. 需要 `Elem()` 方法获取到元素列表
    2. 只能访问到 public 的 字段和方法

    通过 `Call()` 可以调用结构体的方法，
    如果无参，可以传入 `nil` 或者空切片
    如果有参，需传入Value切片 `[]reflect.Value`

    ```go
    //没有参数，直接写nil
    .Call(nil)
    // 或者
    args1 := make([]reflect.Value, 0)
    .Call(args1)

    // 有参数
    args2 := []reflect.Value{ reflect.ValueOf("反射机制"), reflect.ValueOf(100), ...}
    .Call(args2)
    ```

### 反射调用函数
反射调用函数和 调用方法其实差不多，可以通过函数名获得 `reflect` 对象，使用 `Call()` 调用

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    //函数的反射
    f1 := fun1
    value := reflect.ValueOf(f1)
    fmt.Printf("Kind : %s , Type : %s\n", value.Kind(), value.Type()) //Kind : func , Type : func()

    value2 := reflect.ValueOf(fun2)
    fmt.Printf("Kind : %s , Type : %s\n", value2.Kind(), value2.Type()) //Kind : func , Type : func(int, string)

    //通过反射调用函数
    value.Call(nil)

    args := []reflect.Value{reflect.ValueOf(100), reflect.ValueOf("hello")}
    value2.Call(args)

}

func fun1(){
    fmt.Println("我是函数fun1()，无参的。。")
}

func fun2(i int, s string){
    fmt.Println("我是函数fun2()，有参数。。",i,s)
}
```