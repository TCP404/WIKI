# 基础类型

## 变量常量
```swift
var score = 0 // 变量
let pi = 3.14 // 常量

var greeting = "Hello"
var numberOfToys = 8
var isMorning = true

var numberOfToys: Int = 8
var x = 0, y = 0.0, z = "code"  // 一行声明多个变量
let PI = 3.14, E = 2.7         // 一行声明多个常量
```

变量名只要是 Unicode 字符就行
```swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dog and cow"
```

变量能改,常量不能
```swift
// good
var name = "foo"
name = "bar"

// bad
let age = 10
age = 20     // 编译时报错
```

## 注释
```swift
// 这一行表示 Swift 中的注释。
/*
这都被注释掉了。
没有一个会跑！
*/
```


## 类型

类型注解, 对于不复杂的可以不写注解, swift 会自动推导
```swift
var numberOfToys: Int = -8        // 有符号整型
var numberOfAge: UInt = 18        // 无符号整型
var minPrice: Float = 8.99       // 32 位浮点型, 精度至少为小数点后 6 位
var price: Double = 8.99         // 64 位浮点型, 精度至少为小数点后 15 位
var greeting: String = "Hello"   // 字符串类型
var nice: Character = "!"        // 字符类型
var isMorning: Bool = true       // 布尔类型
var red, green, blue: Double             // 这三个都是 Double 类型
var red: Int, green: Int, blue: Double   // 也可以这样一行声明不同类型
```

整数字面量
```swift
let binaryInteger = 0b10001       // 以二进制表示的 17
let octalInteger = 0o21           // 以八进制表示的 17
let decimalInteger = 17           // 以十进制表示的 17
let hexadecimalInteger = 0x11     // 以十六进制表示的 17
```

浮点数字面量
```swift
// 十进制
let decimalDouble = 125.0
let exponentDouble = 1.25e2 // 表示 1.25 x 10² 或 125.0。
let decimalDoubleNegative = 0.0125
let exponentDoubleNegative = 1.25e-2 // 表示 1.25 x 10⁻² 或 0.0125。

// 十六进制
let hexDecimalDouble = 0x3C
let hexExponentDouble = 0xFp2 // 表示 15 x 2² 或 60.0。


let paddedDouble = 000123.456  // 可以额外填充 0
let oneMillion = 1_000_000     // 可以额外填充 0
let justOverOneMillion = 1_000_000.000_000_1 // 可以用下划线分割, 提高可读性
```

布尔类型
```swift
let good = true
let bad = false
```

元组
```swift
let http404Error = (404, "Not Found")
let http200Ok: (Int, String) = (200, "Ok")
```

解构元组
```swift
let (code, msg) = http404Error // 解构
let (okCode, _) = http200Ok    // 不要都可以用 _ 替代
```

访问元组
```swift
print("The status code is \(http404Error.0)") // The status code is 404
```

命名元组
```swift
let http200Status = (code: 200, desc: "OK")
```

访问命名元组
```swift
print("The status code is \(http200Status.code)")  // The status code is 200
```

## 可选类型

类型后加个 `?`, 表示值可能是某个类型, 也可能不存在.
```swift
var statusCode: Int? = 404
statusCode = nil
```

当值为 nil 时,跳过对其进行操作的代码
```swift
if let msg = john.robot?.say {
    print("John's robot say \(msg)")
}
```

使用 `??` 运算符提供一个后备值
```swift
let name = john.name ?? "unknown name"

// john.name 为 nil 时, name 为 unknown name
// john.name 不为 nil 时, name 为 john.name 的值
```

使用 `!` 强制解包
```swift
let name = john.name!
// john.name 不为 nil 时, name 为 join.name 的值
// john.name 为 nil 时, 程序停止运行
```

## 字符串
字符串字面量,由一对双引号(`"`)包裹
```swift
// 单行字符串
let name = "Foo"

// 空字符串
let firstName = String()
let lastName = ""

// 多行字符串
let myLongString = """
Swift?
这是我最喜欢的语言！
"""

// 多行字符串不换行
let singleString = """
Swift? \
这是我最喜欢的语言！
"""
print(singleString) // 打印: Swift? 这是我最喜欢的语言！

```

字符串插值, 用反斜杠 `\` 加一对括号`()`
```swift
var apples = 6
print("I have \(apples) apples!")
// 打印: I have 6 apples!
```



## 区间运算符
```swift
let numbers = [0,1,2,3,4,5,6,7,8,9]


for i in numbers[0...5] { print(i) } // 闭区间, 打印 0,1,2,3,4,5
for i in numbers[0..<5] { print(i) } // 半开区间, 打印 0,1,2,3,4
for i in numbers[5...] { print(i) }  // 单侧闭区间, 打印 5,6,7,8,9
for i in numbers[...5] { print(i) }  // 单侧闭区间, 打印 0,1,2,3,4,5
for i in numbers[..<5] { print(i) }  // 单侧开区间, 打印 0,1,2,3,4

// 没有 0>.. 这种形式
```
