# 5-数组

数组是同一种数据类型元素的集合。

在 Go 中 数组从声明时就确定，使用时可以修改数组成员，但不能修改数组大小。

## 数组定义

!!! warning "注意"
    **数组是值类型！！！**

```go
var idn [len]T
```
`idn`：数组名、`len`：数组长度、`T`：数组类型

数组长度必须是常量，且一旦定义就不能更改。
`[5]int`和`[10]int`是不同的类型。

```go
var arr1 [5]int
var arr2 [8]int
arr1 = arr2     // 禁止这样做，因为此时 arr1 和 arr2 是不同的类型
```
arr2 不可以赋值给 arr2，因为它俩类型不同。

如果换成 `var arr1 [5]int` 和 `var arr2 [5]int` 这样就可以。

```go
func main() {
    var arr1 = [3]int{12, 3}
    var arr2 [3]int

    arr2 = arr1
    for _, e := range arr2 {
        fmt.Println(e)
    }
}

// ----------------------------------------
// Output:
12
3
0
```

## 数组初始化
初始化方式有很多种

### 方法1
使用初始化列表设置数组元素的值
```go
var idn [len]T
var idn = [len]T{initial list}
```

eg:
```go
var a [3]int
var b = [3]int{1, 2}
var c = [3]string{"Alice", "Boii"}

fmt.Println(a)    // [0 0 0]
fmt.Println(b)    // [1 2 0]
fmt.Println(c)    // [Alice Boii]
```

### 方法2
让编译器根据初始值个数自行推断数组长度
```go
var idn = [...]T{initial list}
```

eg:
```go
var b = [...]int{1, 2, 3, 4}
fmt.Println(b)                    // [1 2 3 4]
fmt.Printf("Type of b: %T \n", b) // Type of b: [4]int
```

### 方法3
通过指定索引值来初始化数组，数组长度为 `最大的下标+1`。

通过这种方式的可以不指定数组长度，如果指定了数组长度且没有更大的下标，则以指定长度为最大长度。
```go
var idn = [...]T{idx: elem, idx: elem}
```

eg:
```go
func main() {
    a := [...]rune{1: 'i', 5: 'v'}
    b := [7]int{0: 2, 4: 5}

    fmt.Println(a)    // [0 i 0 0 0 v]
    fmt.Println(b)    // [2 0 0 0 5 0 0]

    fmt.Printf("Type of a: %T \n", a) // Type of a: [6]rune
    fmt.Printf("Type of b: %T \n", b) // Type of b: [7]int

    // b := [3]int{0: 2, 4: 5}
    // 上面这句会报错，指定长度3，却又指定了索引4
}
```

## 数组遍历

数组遍历可以通过 `for` 循环，也可以通过 `for...range`，比较推荐 `for...range`

```go
func main() {
    str := [...]string{"广州", "深圳", "东莞"}

    for i := 0; i < len(str); i++ {
        fmt.Println(str[i])
    }

    for idx, value := range str {
        fmt.Println(idx, value)
    }
}
```

## 多维数组
Go 中支持多维数组，这里以二维数组为例。

### 定义
```go
func main() {
    a := [3][2]string{
        {"广东", "广州"},
        {"浙江", "杭州"},
        {"四川", "成都"},
    }
    fmt.Println(a)          // [[广东 广州] [浙江 杭州] [四川 成都]]
    fmt.Println(a[1][1])    // 支持索引取值： 杭州
}
```

### 遍历
```go
func main() {
    a := [3][2]string{
        {"广东", "广州"},
        {"浙江", "杭州"},
        {"四川", "成都"},
    }

    for _, v1 := range a {
        for _, v2 := range v1 {
            fmt.Printf("%s \t", v2)
        }
        fmt.Println()
    }
}

// ----------------------------------------
// Output:
广东 	广州
浙江 	杭州
四川 	成都
```

!!! warning "注意"
    **注意：** 多维数组只有第一层可以省略长度。
eg：

```go
// 合法写法
a := [...][2]string{
    {"广东", "广州"},
    {"浙江", "杭州"},
    {"四川", "成都"},
}

// 非法写法
a := [...][...]string{
    {"广东", "广州"},
    {"浙江", "杭州"},
    {"四川", "成都"},
}
```

## 数组是值类型
数组是值类型，赋值和传参会赋值整个数组。因此改变副本的值，不会改变本身的值。

```go
func modifyArray(x [3]int) {
    x[0] = 100
}

func modifyArray2(x [3][2]int) {
    x[2][0] = 100
}
func main() {
    a := [3]int{10, 20, 30}
    modifyArray(a) //在modify中修改的是a的副本x
    fmt.Println(a) //[10 20 30]
    b := [3][2]int{
        {1, 1},
        {1, 1},
        {1, 1},
    }
    modifyArray2(b) //在modify中修改的是b的副本x
    fmt.Println(b)  //[[1 1] [1 1] [1 1]]
}
```

注意：

1. 数组支持 `==`、`!=` 操作符，因为内存总是被初始化过的。
2. `[len]*T` 表示指针数组，本质是 **数组**，存放指针的数组
3. `*[len]T` 表示数组指针，本质是 **指针**，指向数组的指针