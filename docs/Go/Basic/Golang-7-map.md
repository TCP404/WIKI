---
title: 'Golang [基础] 7-map'
seotitle: 'Golang [基础] 7-map'
pin: false
tags:
  - Golang
categories: [Golang, Basic]
headimg: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/cover/go2.png'
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/golang.png'
abbrlink: d554428f
date: 2021-07-17 11:19:18
updated: 2021-07-17 11:19:18
---

键值对的创建与使用

<!--more-->

# 7-map

> map 是 Go 提供的一种映射关系容器，其内部使用`散列表(hash)`实现
> map 是一种无序的基于 `key-value` 的数据结构
> Go 中的 map 是引用类型，必须初始化才能使用。

- key 可以是任意可用 `== 或 !=` 比较的类型，如：string、int、float
- 数组、切片不能作为 key
- 指针和接口类型可以作为 key
- 包含数组切片的结构体不能作为 key，只包含原生类型的结构体可以作为 key
- 如果结构体要作为 key 可以提供 `Key()` 和 `Hash()` 方法
- value 可以是任意类型

- 通过 key 在map 中查找是很快的，但是还是比数组和切片慢100倍。如果性能很重要的话还是用切片来解决问题。
- map可以用函数作为值，这样就可以用来做分支结构：用key来选择要执行的函数。



## 声明

{% note warning, **注意：map 是引用类型！！！** %}

定义语法如下
```go
var map1 map[keyT]valueT

// 示例
var map1 map[string]int
var map2 map[int]rune
```

声明时不需要直到map长度，map是可以动态增长的
未初始化的map的值是 nil，需要使用 `make()` 函数来分配内存
```go
make(map[keyT]valueT, cap)
```
`cap` 是可选的，但是我们应该再初始化map的时候就为其指定一个合适的容量。

eg：
```go
func main() {
    var m1 map[int]string
    m2 := make(map[int]string)

    fmt.Println(m1)			// map[]
    fmt.Println(m2)			// map[]
    fmt.Printf("%T \n", m1)	// map[int]string
    fmt.Printf("%T \n", m2)	// map[int]string
}
```


## 声明并初始化

```go
func main() {
    m1 := map[int]string{
        1: "Alice",
        2: "Boii",
        3: "Candy",
        4: "Danish",
    }
    fmt.Println(m1)            // map[1:Alice 2:Boii 3:Candy 4:Danish]
    fmt.Printf("%T \n", m1)    // map[int]string
}
```
**注意：**这种方式，每一对 `k-v`后面都要有`,`，**最后一对也要有**

## 单独赋值
```go
func main() {
    m1 := make(map[int]string)

    m1[1] = "Alice"
    m1[2] = "Boii"
    m1[5] = "Eva"

    fmt.Println(m1)	    // map[1:Alice 2:Boii 5:Eva]
}
```

## 判断键是否存在
Go 中有个判断 map 中键是否存在的特殊写法，基本格式为：
```go
val, ok := map[key]
```

eg：
```go
func main() {
    m1 := map[byte]string{
        'a': "Alice",
        'b': "Boii",
        'c': "Candy",
        'd': "Danish",
    }

    val, ok := m1['a']
    fmt.Println(val) // Alice
    fmt.Println(ok)  // true

    if val, ok := m1['b']; ok {
        fmt.Println("I got", val)
    } else {
        fmt.Println("None.")
    }
}
```

## 遍历
可以使用 `for...range` 遍历
```go
func main() {
    m1 := map[byte]string{
        'a': "Alice",
        'b': "Boii",
        'c': "Candy",
        'd': "Danish",
    }

    for k, v := range m1 {
        fmt.Println(k, v)
    }
}
```

只想遍历 key 的时候可以
```go
for k := range m1 {
    fmt.Println(k)
}
```

只想遍历 value 的时候可以
```go
for _, v := range m1 {
    fmt.Println(v)
}
```
{% note warning, **注意：** 遍历 map 时元素顺序与添加键值对顺序无关。 %}

## delete()
Go 的内建函数 `delete()` 可以从 map 中删除一对键值对，格式如下：
```go
delete(map, key)
```

eg：
```go
func main() {
    m1 := map[int]byte{
        1: 'A',
        2: 'B',
        3: 'C',
    }
    fmt.Println(m1)	// map[1:65 2:66 3:67]
    delete(m1, 2)
    fmt.Println(m1)	// map[1:65 3:67]
}
```

## 值为切片的map
map 中 value 可以是任何类型，所以也可以是切片类型

```go
var sm map[keyT][]T

sm := make(map[keyT][]T, cap)

sm := map[keyT][]T{initial value}
```
`[]T`：某种类型的切片

eg：
```go
func main() {
    // 创建一个map, key 为 string, value 为 []int，容量为5
    sliceMap1 := make(map[string][]int, 5)

    // var sliceMap1 map[string][]int 用这种声明方式也行

    // 为 map 中每一对 k-v 初始化
    sliceMap1["A"] = make([]int, 3, 3)
    sliceMap1["B"] = make([]int, 3, 3)
    sliceMap1["C"] = make([]int, 3, 3)
    sliceMap1["D"] = make([]int, 3, 3)
    sliceMap1["E"] = make([]int, 3, 3)

    fmt.Println("1: ",sliceMap1)    // 1:  map[A:[0 0 0] B:[0 0 0] C:[0 0 0] D:[0 0 0] E:[0 0 0]]
    for k, v := range sliceMap1 {
        fmt.Println(k, v)
    }
    // A [0 0 0]
    // B [0 0 0]
    // C [0 0 0]
    // D [0 0 0]
    // E [0 0 0]

    // 用这种方式初始化效果一样
    sliceMap2 := map[string][]int{
        "A": {1, 2, 3},
        "B": make([]int, 3),
        "C": {2, 4, 0, 9},
        "D": make([]int, 3),
        "E": make([]int, 3),
    }
    fmt.Println("2: ",sliceMap2)    // 2:  map[A:[1 2 3] B:[0 0 0] C:[2 4 0 9] D:[0 0 0] E:[0 0 0]]
    for k, v := range sliceMap2 {
        fmt.Println(k, v)
    }
    // A [1 2 3]
    // B [0 0 0]
    // C [2 4 0 9]
    // D [0 0 0]
    // E [0 0 0]
}
```

## 值为map的切片
看起来好像有点绕，但是捋清楚就好办了
`map[keyT]valT`是map类型，`[]T`是切片类型，那么`[]map[keyT]valT`就是map类型切片了
```go
// 类型定义
[]map[keyT]valT

// 声明
var identifier []map[keyT]valT

// 声明并初始化
var identifier = []map[keyT]valT{
    {map1key1: val, map1key2: val, map1key3: val},
    {map2key1: val, map2key2: val, map2key3: val},
    {map3key1: val, map3key2: val, map3key3: val},
}

// 声明并初始化
identifier := []map[keyT]valT{
    {map1key1: val, map1key2: val, map1key3: val},
    {map2key1: val, map2key2: val, map2key3: val},
    {map3key1: val, map3key2: val, map3key3: val},
}

// 使用make
identifier := make([]map[keyT]valT, cap)
identifier[skey] = make(map[keyT]valT)
identifier[skey][mkey] = val
identifier[skey] = map[keyT]valT{mkey1: val, mkey2: val}

```

eg：
```go
var ms []map[int]string
```

```go
func main() {
    // 声明一个切片，map[int]string 类型的，并初始化
    ms := []map[int]string{
        {1: "A", 2: "B", 3: "C"},
        {4: "I", 6: "N", 9: "G"},
        {3: "R", 7: "Y", 5: "Q"},
        make(map[int]string),
    }
    for _, v := range ms {
        fmt.Println(v)
    }
    // map[1:A 2:B 3:C]
    // map[4:I 6:N 9:G]
    // map[3:R 5:Q 7:Y]
    // map[]
}
```

```go
func main() {
    // make 一个切片，类型为 map[int]string, 长度和容量都为 3
    mapSlice := make([]map[int]string, 3)
    // 为切片第一个元素创建一个 map，然后为其逐个添加 k-v
    mapSlice[0] = make(map[int]string)
    mapSlice[0][1] = "A"
    mapSlice[0][2] = "B"
    mapSlice[0][3] = "C"

    // 为切片第二个元素 创建并初始化一个 map
    mapSlice[1] = map[int]string{4: "I", 6: "N", 9: "G"}

    for _, v := range mapSlice {
        fmt.Println(v)
    }
    // map[1:A 2:B 3:C]
    // map[4:I 6:N 9:G]
    // map[]
}
```

{% noteblock warning, 注意 %}

- Go 内置的 map 不是并发安全的，并发安全的 map 可以使用标准包 sync 中的 map
- 不要直接修改 map value 中某个成员的值。如果想修改 map value 中的某个成员的值，必须整体赋值。

eg：
```go
type User struct {
    name string
    age  int
}

func main() {
    ma := make(map[rune]User)
    boii := User{"Boii", 18}
    ma['a'] = boii

    // ma['a'].age = 19  // ERROR，不能通过 map 引用直接修改

    boii.age = 19
    ma['a'] = boii	// 必须整体替换
}
```

{% endnoteblock %}

## 练习
1. 写一个程序，统计一个字符串中每个单词出现的次数。比如：”how do you do” 中 how=1 do=2 you=1。

思路：
先用 `strings.Split` 把单词切割出来
然后利用 map ，把单词作为 key，出现频次作为 value，遍历一下就能得到答案

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    str := "how do you do I do not know how to tell you"
    strSplit := strings.Split(str, " ")
    strMap := make(map[string]int)
    for _, word := range strSplit {
        strMap[word] += 1
    }
    fmt.Println(strMap)
}

// -----------------------------------------
// Output:
map[I:1 do:3 how:2 know:1 not:1 tell:1 to:1 you:2]
```