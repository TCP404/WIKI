# Control Flow

## For-In
```swift
var foo = [1, 2, 3, 4, 5]
var total = 0
for i in foo {
    total += i
}
print(total)  // 15
```

## While

```swift
var i = 0
while i < 10 {
    i += 1
}
print(i) // 10
```


## Repeat-While

也就是 do-while

```swift
var i = 0
repeat {
    i += 1
} while i < 10
print(i)  // 10
```

## If

```swift
var i = 10

if i == 10 {
    print("ten")
} else if i == 0 {
    print("zero")
} else {
    print("foo")
}
```

if 表达式
```swift
var i = 10

let ret = if i == 10 {
    "ten"
} else if i == 0 {
    "zero"
} else {
    "foo"
}

print(ret) // ten
```

如果声明一个可选值(例如 `String?`), 那么必须有一个 nil 分支
```swift
var i = 10

let ret: String? = if i == 10 {
    "ten"
} else {
    nil  // 必须有一个 nil 分支
}
print(ret) // ten

// 或者

let ret = if i == 10 {
    "ten"
} else {
    nil as String?
}
print(ret) // ten
```

## Switch

```swift
switch <some value to consider> {
case <value 1>:
    <respond to value 1>

case <value 2>, <value 3>:
    <respond to value 2 or 3>

default:
    <otherwise, do something else>
}
```

```swift
var i = 10

switch i {
case 1:
    print("one")
case 10:
    print("ten")
default:
    print("foo")
}
// ten
```

switch 表达式

```swift
var i = 10

let ret = switch i {
case 1: "one"
case 10: "ten"
case 15, 20: "good"
default: "foo"
}

print(ret)  // ten
```

区间匹配
```swift
let i = 15

let ret = switch i {
case 0: "zero"
case 1..<10: "less 10"
case 10..<20: "less 20"
default: "huge"
}
print(ret)  // less 20
```

元组
```swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("\(somePoint) is at the origin")
case (_, 0):
    print("\(somePoint) is on the x-axis")
case (0, _):
    print("\(somePoint) is on the y-axis")
case (-2...2, -2...2):
    print("\(somePoint) is inside the box")
default:
    print("\(somePoint) is outside of the box")
}
// (1, 1) is inside the box
```


值绑定
```swift
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// on the x-axis with an x value of 2
```

switch where

```swift
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
// (1, -1) is on the line x == -y
```


## Guard
guard 语句必须带 else, 条件为真时继续走, 为假时走 else. else 里必须有 return, throw, break, continue, fatalError().
```swift
func greet(person: [String: String]) {
    guard person.count == 2 else {
        print("warning")
        return
    }

    guard let name = person["foo"] else {
        print("foo is required")
        return
    }

    print(name)

    print(person)
}

greet(person: ["foo": "Alice", "bar": "Bob"])                // ["bar": "Bob", "foo": "Alice"]
greet(person: ["foo": "Alice", "bar": "Bob", "baz": "Cat"])  // 打印 warning 然后退出
greet(person: ["dub": "Alice", "bar": "Bob"])                // 打印 foo is required 然后退出
```

## Defer
和 golang 的 defer 一样, 适合用来释放资源, 事务回滚 等
```swift
var score = 3
if score < 100 {
    score += 100
    defer {
        score -= 100
    }
    // 这里可以写其他使用分数和奖励的代码。
    print(score)
}
// 输出 "103"
```


多个 defer 时先声明都 defer 后执行, 类似栈, 和 golang 一样的
```swift
if score < 10 {
    defer {
        print(score)
    }
    defer {
        print("The score is:")
    }
    score += 5
}
// 输出 "The score is:"
// 输出 "6"
```

## #available 检测 API 可用性

```swift
if #available(iOS 10, macOS 10.12, *) {
    // 在 iOS 使用 iOS 10 的 API, 在 macOS 使用 macOS 10.12 的 API
} else {
    // 使用先前版本的 iOS 和 macOS 的 API
}


@available(macOS 10.12, *)
struct ColorPreference {
    var bestColor = "blue"
}


func chooseBestColor() -> String {
    guard #available(macOS 10.12, *) else {
       return "gray"
    }
    let colors = ColorPreference()
    return colors.bestColor
}
```
available 里可以填的平台参考[这里](https://doc.swiftgg.team/documentation/the-swift-programming-language/attributes/#available)
