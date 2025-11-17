# Collection Type

## Array 数组

声明
```swift
var foo: Array[Int] = [1, 2, 3] // 完整写法 Array[Int]
var bar: [Int] = [1, 2, 3]      // 简写 [Int]
var far = [1, 2, 3]             // 最简写法, 自动类型推导
var baz = Array(repeating: 0, count: 3) // [0, 0, 0] 使用默认值创建数组
```

空数组
```swift
var foo: [Int] = []
var bar = [Int]()
```

数组相加
```swift
var foo = [1, 2, 3]
var bar = [7, 8, 9]
var baz = foo + bar // [1, 2, 3, 7, 8, 9]
```

数组访问
```swift
var foo = [1, 2, 3]
foo[0] = 4
print(foo) // [4, 2, 3]
```

范围修改
```swift
var foo = [0, 1, 2, 3, 4, 5, 6, 7, 8]
foo[3...5] = [9, 10]
print(foo) // [0, 1, 2, 9, 10, 6, 7, 8]
```

遍历
```swift
var foo = ["a", "b", "c", "d", "e"]
for f in foo {
    print(f)
}
// a
// b
// c
// d
// e
```

枚举遍历
```swift
for i, f in foo.enumerated() {
    print("\(i): \(f)")
}
// 0: a
// 1: b
// 2: c
// 3: d
// 4: e
```


## Set 集合
一个类型必须是 Hashable 的才能存储在集合中.

> Swift 的所有基本类型（如 String、Int、Double 和 Bool）默认都是可哈希的，可以用作集合的值类型或字典的键类型。没有关联值的枚举 case 值（如 枚举 中描述的那样）默认也是可哈希的。

声明
```swift
var foo = Set<Character>()
var bar: Set<String> = ["foo", "bar", "baz"]
var baz: Set = ["foo", "bar", "baz"]
```

访问
```swift
var foo: Set = ["foo", "bar"]
print("count: \(foo.count)")              // count: 2
print("is empty: \(foo.isEmpty)")         // is empty: false
print("contains: \(foo.contains("foo"))") // contains: true
```

修改
```swift
var foo: Set = ["foo", "bar"]
foo.insert("baz")
foo.remove("baz")
```

集合运算

- 使用 `intersection(_:)` 方法创建一个只包含两个集合共有值的新集合。
- 使用 `union(_:)` 方法创建一个包含两个集合中所有值的新集合。
- 使用 `subtracting(_:)` 方法创建一个不包含指定集合中值的新集合。
- 使用 `symmetricDifference(_:)` 方法创建一个包含两个集合中存在但不同时存在的值的新集合。

```swift
let odd: Set = [1, 3, 5, 7, 9]
let even: Set = [2, 4, 6, 8, 10]
var foo: Set = [1, 2, 6, 7]

odd.union(even).sorted()               // odd | even : [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
odd.intersection(even).sorted()        // odd & even : []
odd.subtracting(foo).sorted()          // odd - foo: [3, 5, 9]
odd.symmetricDifference(foo).sorted()  // (odd | foo) - (odd & foo): [3, 4, 5, 8, 9, 10]
```

- 使用 “等于” 运算符 （==）判断两个集合是否包含相同的所有值。
- 使用 isSubset(of:) 方法判断一个集合的所有值是否包含在指定集合中。
- 使用 isSuperset(of:) 方法判断一个集合是否包含指定集合中的所有值。
- 使用 isStrictSubset(of:) 或 isStrictSuperset(of:) 方法判断一个集合是否是指定集合的子集或超集（但不相等）。
- 使用 isDisjoint(with:) 方法判断两个集合是否没有共同的值。

```swift
var a: Set = [1, 2, 3, 4, 5, 6]
var sa: Set = [1, 2, 3]
var b: Set = [7, 8, 9, 10]

sa.isSubset(a)   // true
a.isSuperSet(sa) // true
a.isDisjoint(b)  // true
```

## Dictionary 字典

声明
```swift
var foo: Dictionary<Int: String>
var foo: [Int: String] = [:]
var a = [1: "foo", 2: "bar", 3: "baz"]
var b: [Int: String] = [1: "foo", 2: "bar", 3: "baz"]
var c: [Int, Any] = [1: "foo", 2: 0.1]
```

访问
```swift
var foo: [Int: String] = [1: "foo", 2: "bar"]
print("count: \(foo.count)")              // count: 2
print("is empty: \(foo.isEmpty)")         // is empty: false
print("foo[2]: \(foo[2])")                // foo[2]: bar
```

修改
```swift
var foo: [Int: String] = [1: "foo", 2: "bar"]
foo[1] = "baz"
print("\(foo[1])").  // baz

foo.updateValue("cat", forKey: 1)
print("\(foo[1])")  // cat

if let oldVal = foo.updateValue("dub", forKey: 2) {
    print("foo[2]: \(oldVal)")
}
// foo[2]: baz


if let removeVal = foo.removeValue(forKey: 1) {
    print("\(foo)")
}
// [2: "dub"]
```

遍历
```swift
var foo = [1: "foo", 2: "bar", 3: "baz"]

for (i, v) in foo {
    print("\(i): \(v)")
}
// 1: foo
// 2: bar
// 3: baz
```

```swift
var foo = [1: "foo", 2: "bar", 3: "baz"]
for i in foo.keys {
    print("\(i)")
}
// 1
// 2
// 3
```

```swift
var foo = [1: "foo", 2: "bar", 3: "baz"]
for v in foo.values {
    print("\(v)")
}
// foo
// bar
// baz
```

```swift
var foo = [1: "foo", 2: "bar", 3: "baz"]
let ks = [Int](foo.keys)
let vs = [String](foo.values)
```