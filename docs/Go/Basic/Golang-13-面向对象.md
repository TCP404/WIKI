# 13-面向对象

面向对象三大特性：封装、继承、多态

Golang 没有类的概念，也没有面向对象的概念。
准确的说，面向对象、封装、继承、多态、抽象等等，这些都是编程思想，不同的语言实现这些特性的方式不同。
例如 `Java`，用的是类 `class`，访问修饰符 `public、protected、default、private` 等来实现；
在 Golang 中，用的是结构体 `struct`、标识符首字母大小写 等来实现。

!!! note ""
    面向过程、面向对象、一切皆对象、一切皆文件 等诸如此类的概念，
    在学习之初可能会成为初学者的一道坎，也可能是帮助新手更快入门的好帮手；
    等到学到一定程度以后，这些思想能帮助我们快速解决一些问题，也可能开始禁锢我们的思想；
    善于变通者会慢慢看透本质，脱离这些思想的枷锁，对编程形成自己的认知。
    变通者和不变通者的区别在于：是否愿意深入底层（汇编、组原等等），是否愿意摒弃语言执念。


1. **封装**

    - 封装也叫 **信息隐藏、数据访问保护**。通过暴露有限的访问接口，外部仅能通过类提供的方式来访问内部信息或数据。
    - 需要编程语言提供权限访问控制语法来支持。如：
        - Java 中的 `public、protected、private`
        - Python 中标识符的双下划线前缀 `__xxx`或`__slots__` 白名单
        - Golang 中标识符首字母大小写。
    - 封装存在的意义，
        - 一方面是保护数据不被随意修改，提高代码可维护性；
        - 一方面是仅暴露有限的必要接口，提高易用性。

2. **抽象**

    - 封装讲的是如何隐藏信息、保护数据，抽象讲的就是如何隐藏的具体实现。
    - 抽象可以通过接口类或者抽象类来实现，但不需要特殊的语法机制来支持。
    - 抽象存在的意义，
        - 一方是是提高代码的可扩展性、维护性，修改实现不需要修改定义，减少代码改动范围；
        - 另一方面，抽象也是处理复杂系统的有效手段，能有效过滤掉不必关注的信息。

3. **继承**

    - 继承是用来表示类之间的 `is-a` 和 `has-a` 的关系，分为两种模式：单继承和多继承。
    - 单继承表示一个子类只能继承一个父类，多继承表示一个子类可以继承多个父类。
    - 需要编程语言提供特殊语法机制来支持。如：
        - Java 中的 `extends` 关键字
        - Python 中类名后的括号
        - Go 中的结构体嵌套
    - 继承存在的意义，是用来解决代码复用的问题。

4. **多态**

    - 多态是指子类可以替代父类。在实际代码运行过程中，调用子类的方法实现。
    - 需要编程语言提供特殊语法机制来支持。如：继承、接口、duck-typing。
    - 多态可以提高代码的扩展性和复用性，是很多设计模式、设计原则、编程技巧的代码实现的基础。




## 封装
在 Java 等面向对象中，会将一类事物抽象出`属性`和`行为`，并通过语言层面限定 `访问性`。

属性 使用基本数据类型或复合类型描述；
行为 通过函数描述，并称之为`方法`；
`属性+方法`组成一个类`class`；
访问性各个语言实现不同。



在 Go 中没有类的概念，
而是将事物的 **属性** 使用基本数据类型或复合数据类型 **封装在结构体中**，
而 **行为** 是通过 **给函数限定调用者的方式** 实现。
**访问性** 是通过首字母大写为 `public`，首字母小写为 `private`。

这种限定了调用者的函数，我们称之为 **方法**。

而拥有方法的结构体，我将其称之为 **类**。

!!! info
    Golang 中，**类 = 结构体 + 限定调用者的函数 + 访问性**

```go
// class A

type A struct {
    fieldA
}

func (a A) aMethod() {
    ...
}
```

### 构造函数

Go 不支持像 Java 那样的构造函数，但是可以通过 `标识符首字母大小写 + 工厂函数` 实现构造函数。

一般分为4步：

1. 将结构体、字段的首字母小写
2. 结构体所在的包提供一个工厂模式的函数，首字母大写，模拟一个构造函数。按照规范，构造函数的名字以 `new` 或 `New` 开头。**注意返回值必须是结构体指针**，因为结构体是值类型。
3. 提供首字母大写的 Get 方法，用于获取属性的值，建议命名规则：属性名首字母大写，如属性 `sex` 的 Get 方法为 `Sex()`、属性 `name` 的 Get 方法为 `Name()`。
4. 提供首字母大写的 Set 方法，用于设置属性的值，建议命名规则：Set+属性名首字母大写，如属性 `sex` 的 Set 方法为 `SetSex()`、属性 `name` 的 Set 方法为 `SetName()`。

eg：
```go
type person struct {
    name string
    age  int
}

// 写一个工厂函数，首字母大写，其他方就可以访问，相当于构造函数
func NewPerson (name string, age int) *person {
    if age <= 0 || name == "" { return nil }
    return &person{name, age}
}

// Get 方法
func (p person) Name() string{
    return p.name
}

func (p person) Age() int {
    return p.age
}
// Set 方法
func (p person) SetName(name string) {
    if name == "" { return }
    p.name = name
}

func (p person) SetAge(age int) {
    if age <= 0 { return }
    p.age = age
}


func main() {
    // 然后这样创建对象：
    p := NewPerson("Boii", 18)
    fmt.Println(p)    // &{Boii 18}

    p.Name()           // Boii
    p.SetAge(20)    // p.age == 20
}
```

### toString
在面向对象中，每个类默认继承自 Object，打印一个对象的时候，会调用这个对象的 `toString()` 方法，如果这个对象没有重写 `toString()`，会找其父类，一层层往上，找到了执行 `toString()`，找不到就执行 Object 的 `toString()`。

想要打印对象信息，
在 Java 中是 `.toString()`，
在 Python 中是 `.__str__()`

!!! info
    **在 Go 中是 `.String()`**

在 Go 中，可以通过 `%v` 打印结构体信息。

`fmt.Printf("%v \n", t)` 这句话等价于 `fmt.Printf("%v \n", t.String())`，也等价于 `fmt.Println(t)`。

那么想要打印结构体信息时按照自己想法来，就可以为结构体写一个 `String()` 方法。

```go
type car struct {
    band  string
    model string
}

func (c car) String() string {
    return c.band + "-" + "c.model"
}

func main() {
    c1 := &car{"Benz", "S600"}
    fmt.Printf("%v \n", c1)		      // Benz-S600
    fmt.Printf("%v \n", c1.String())  // Benz-S600
    fmt.Println(c1)                   // Benz-S600
}
```


## 继承

### 面向对象中的继承性

如果两个类 class 存在继承关系，其中一个是子类，另一个作为父类，那么：

1. 子类可以直接访问父类的属性和方法
2. 子类可以新增自己的属性和方法
3. 子类可以重写父类的方法（override，就是将父类已有的方法，重新实现）

Golang 语法上不支持继承，但是通过结构体嵌套却可以实现继承，而且可以多继承，
且通过 匿名字段 和 非匿名字段 还可以进一步区分 `is-a` 继承关系 和 `has-a` 聚合关系

### Golang 的结构体嵌套

#### 模拟继承性：is - a
```go
type Base struct {
    fieldB
}
type Son struct {
    fieldS
    Base // 匿名字段，模拟的是 继承关系
}

func (b Base) baseMethod() { // A的方法
    fmt.Println("base method")
}

func (s Son) sonMethod() { // B的方法
    fmt.Println("son method")
}

func main() {
    base := Base{}
    son := Son{}

    base.fieldB     // 正确使用
    son.fieldS      // 正确使用
    son.fieldB      // 正确使用
    son.Base.fieldB // 正确使用
    base.fieldS     // !报错
    base.Son.fieldS // !报错

    base.baseMethod()     // 正确使用
    son.sonMethod()       // 正确使用
    son.baseMethod()      // 正确使用
    son.Base.baseMethod() // 正确使用
    base.sonMethod()      // !报错
    base.Son.sonMethod()  // !报错
}
```

#### 模拟聚合关系：has - a
```go
type C struct {
    fieldC
}
type D struct {
    fieldD
    c C    // 非匿名字段，模拟的是 聚合关系
}

func (c C) cMethod() {
    fmt.Println("C method")
}

d := D{...}

d.fieldC      // !报错
d.C.fieldC    // !报错
d.c.fieldC    // 正确使用
d.c.cMethod() // 正确使用
```

### 小结
在结构体中嵌套了其他结构体，会出现两种情况，一种是 is - a 的关系，一种的 has - a 的关系。

is - a 是一种继承关系，子类可以直接使用父类（被嵌套类）的变量，如上面例子中的 `b.fieldA`
has - a 是一种聚合关系，当前类使用聚合类（被嵌套类）的变量必须通过聚合类的名字，如上面例子中的 `d.c.fieldC`，其他两种方式会报错。


## 多态
Golang 中的多态是通过 接口 和 Duck-typing 实现的。

`Duck-typing` 也是一种编程思想：只要一个东西看起来像鸭子，走路像鸭子，吃起来像鸭子...，那它就是鸭子。
反映在编程语言中就是，只要一个结构体或者一个类，具有某个方法的具体实现，那它就可以被我这个函数/方法接受。

Golang 中的接口是非侵入式的，不像`Java` 那样需要显式的在类声明中加上 `implement xxer`，
Golang 不需要结构体显示的声明实现某个接口，只要你这个结构体有我这个接口所有方法的具体实现，那这个结构体就实现是我这个接口，就是我的实现类，就是我这种类型。


```go
type Aer interface {
    show() string
}

type X struct {}
type Y struct {}
type Z struct {}

func (x X) show() string {
    return "I'm X"
}

func (x X) add(a, b int) int {
    return a + b
}

func (x Y) show() string {
    return "I'm Y"
}

func PrintStruct(a Aer){
    fmt.Println(a.show())
}

func main() {
    x := X{}
    y := Y{}
    z := Z{}

    PrintStruct(x)    // I'm X
    PrintStruct(y)    // I'm Y
    PrintStruct(z)    // !报错, 因为 Z 没有实现 Aer 的方法 show()，不是 Aer 类型
}
```

### 重写
**重写**，说大白话就是：爹有的，儿子不满意，儿子自己来。

Golang 中通过嵌套结构体模拟继承，如果子结构体有和父结构体 **同名同参的方法**，则称作重写。

Golang 中不支持同名不同参。

```go
type base struct {    // 父类
    name string
}

type son struct {    // 子类
    base             // 继承了父类
    age int
}

func (b base) say() {    // 父类方法
    fmt.Println("Base said.")
}

func (b base) run() {    // 父类方法
    fmt.Println("Base ran.")
}

func (s son) say() {    // 子类方法，重写了父类方法
    fmt.Println("Son said.")
}


func main() {
    b := base{"Eva"}            // 实例化父类
    s := son{base{"Boii"}, 64}  // 实例化子类
    b.say()         // Base said.
    s.say()         // Son said.
    b.run()         // Base ran.
    s.run()         // Base ran.
}
```



## 举个栗子

我们通过一个例子看看

!!! example 
    === "Java"

        现在我们用 Java 来定义一个类
        ```java
        public class Dog{
            // 属性
            private String name;
            private int age;

            // 构造函数
            public Dog(String name, int age){
                this.name = name;
                this.age = age;
            }

            // getter
            public String getName() {
                return this.name;
            }
            public int getAge() {
                return this.age;
            }

            // setter
            public void setName(String newName) {
                this.name = newName;
            }
            public void setAge(int newAge){
                this.age = newAge;
            }

            // toString
            @Override
            public String toString() {
                return this.name + "-" + this.age;
            }
        }
        ```
        以上就是用 Java 定义的一个最简单的类了。类中有`属性`、`构造函数`、`getter 和 setter` 和 `toString()`。

        使用的时候是这样子的：
        ```java
        Dog doge = new Dog("Doge", 2);
        System.out.println(doge.getName());    // Doge
        System.out.println(doge.getAge());     // 2

        doge.setAge(3);
        System.out.println(doge.getAge());     // 3
        System.out.println(doge)               // Doge-3
        ```

    === "Go"

        下面我们来看看用 Go 怎么做：
        ```go
        //$GOPATH/src/dog/myDog.go

        package dog

        import (
            "fmt"
            "strconv"
        )

        // 定义 Dog 结构体，包含了属性
        type dog struct {
            name string
            age  int8
        }

        // 构造函数
        func NewDog(name string, age int8) *dog {
            return &dog{name, age}
        }

        // getter
        func (d *dog) Name() string {
            return d.name
        }
        func (d *dog) Age() int8 {
            return d.age
        }

        // setter
        func (d *dog) SetName(newName string) {
            d.name = newName
        }
        func (d *dog) SetAge(newAge int8) {
            d.age = newAge
        }

        // toString
        func (d *dog) String() string {
            return d.name + "-" + strconv.Itoa(int(d.age))
        }
        ```
        同样实现了`属性`、`构造函数`、`getter 和 setter`。

        使用的时候是这样子的：
        ```go
        //$GOPATH/src/Hello/main.go

        package main

        import (
            "dog"
            "fmt"
        )

        func main() {
            doge := NewDog("Doge", 2)
            fmt.Println(doge.Name()) // Doge
            fmt.Println(doge.GetAge())  // 2

            doge.SetAge(3)
            fmt.Println(doge.Age()) // 3
            fmt.Printf("%v \n", doge)  // Doge-3
            fmt.Println(doge) // Doge-3
        }
        ```


## 抽象类

抽象类其实和接口的性质是一样的，但是又多了一些具体的实现。

当一个接口中的某些方法，所有子类的实现都一样时，可以换成抽象类来实现，将这些共同的实现写在抽象类中，剩下不同的实现续集保持抽象。

### 以 Java 为栗
eg：

```java
// 定义一个抽象类
public abstract class Animal {
    // 实现共同方法
    public void run() {
        System.out.println(this.name() + " is running!");
    }

    // 定义抽象方法
    public abstract String kind();
}

// 继承抽象类
public class cat extends Animal {
    // 实现抽象方法
    public String kind() {
        return "cat";
    }
}

// 继承抽象类
public class dog extends Animal() {
    // 实现抽象方法
    public String kind() {
        return "dog";
    }
}
```

上面抽象了一个动物类 `Animal`，我们实现了共同的方法 `run()`，并定义了需要子类自己实现的抽象方法 `kind()`。

接着定义了两个具体类 `cat` 和 `dog` 继承 `Animal`，并各自具体实现抽象方法 `kind()`。


### Go 实现抽象类

Go 并没有抽象类的概念，但是通过 `struct` 和 `interface` 可以实现出抽象类。

思考一下：Java 中抽象类和接口的区别在哪？

其实就是抽象类中需要具体实现一些 **公共方法**，剩下的那些抽象方法，用 Java 中的接口实现也是一样的。

只不过 Java 有抽象类的概念，可以优雅的实现。

那么我们也可以在 Golang 中，定义一个接口IAer，接口中定义一些抽象方法；

然后定义一个 Aer 作为公共的结构体（类），由它来实现公共的部分

然后其他结构体嵌套这个公共结构体，并实现接口 IAer 的方法，这样就能达到与抽象类相同的效果。

```go
// -- 抽象类 -- start
type IAbsClass interface {
    absMethod1()
    absMethod2()
}

func AbsClass struct {}	// 公共结构体
func (a AbsClass) commonMethod1() {	// 公共结构体实现公共方法1
    fmt.Println("AbsClass commonMethod1")
}
func (a AbsClass) commonMethod2() {	// 公共结构体实现公共方法2
    fmt.Println("AbsClass commonMethod2")
}
// -- 抽象类 -- end



// -- 子类继承抽象类
type subClass1 struct {
    AbsClass	// 继承抽象类
}
// 子类实现抽象方法
func (s subClass1) absMethod1() {	// 子类实现抽象方法1
    fmt.Println("subClass1 absMethod1")
}
func (s subClass1) absMethod2() {	// 子类实现抽象方法2
    fmt.Println("subClass1 absMethod2")
}

// 此时子类 subClass1 拥有 4 种方法：
// absClass.commonMethod1()
// absClass.commonMethod2()
// subClass1.absMethod1()
// subClass1.absMethod2()



// -- 子类继承抽象类
type subClass2 struct {
    AbsClass	// 继承抽象类
}
// 子类实现抽象方法
func (s subClass2) absMethod1() {	// 子类实现抽象方法1
    fmt.Println("subClass2 absMethod1")
}
func (s subClass2) absMethod2() {	// 子类实现抽象方法2
    fmt.Println("subClass2 absMethod2")
}
// 子类重写公共方法2
func (s subClass2) commonMethod2() {
    fmt.Println("subClass2 commonMethod1")
}

// 此时子类 subClass2 拥有 4 种方法：
// absClass.commonMethod1()
// subClass2.commonMethod2()
// subClass2.absMethod1()
// subClass2.absMethod2()
```

接口定义 **抽象方法**；
公共结构体实现 **公共方法**；
其他要继承抽象类的子类只要 **匿名嵌套** 公共结构体即可。

调用：

```go
func main() {
    c := &cat{}
    c.run(c)

    d := &dog{}
    d.run(d)
    fmt.Println("kind(): ", d.kind())
    
    s1 := subClass1{}
    s1.commonMethod1()	// 打印：AbsClass commonMethod1
    s1.commonMethod2()	// 打印：AbsClass commonMethod2
    s1.absMethod1()		// 打印：subClass1 absMethod1
    s1.absMethod2()		// 打印：subClass1 absMethod2
    
    s2 := subClass2{}
    s2.commonMethod1()	// 打印：AbsClass commonMethod1
    s2.commonMethod2()	// 打印：subClass2 commonMethod2
    s2.absMethod1()		// 打印：subClass2 absMethod1
    s2.absMethod2()		// 打印：subClass2 absMethod2
}
```

### 小结

Golang 中实现抽象类：

1. **抽象的方法** 放在 **接口** 中
2. **公共的方法** 定义一个 **公共结构体** 去实现，需要用到 `this` 的地方使用接口变量
3. 继承抽象类的子类要做两件事：
    - 匿名嵌套公共结构体
    - 实现接口中的所有方法
4. 在调用公共方法时，需要 `this` 的地方，将子类自己传进去。



## 泛型
据说在 Go 1.18 出



## 单例模式

=== "饿汉式"
    ```go
    type singleton struct{}
    
    var single *singleton = new(singleton)
    
    func GetPersonInstance() *singleton {
        return single
    }
    ```
    测试一下
    ```go
    func main() {
        s3 := GetPersonInstance()
        s4 := GetPersonInstance()
        fmt.Printf("%p\n", s3) // 0x104d5b8
        fmt.Printf("%p\n", s4) // 0x104d5b8
    }
    ```

=== "懒汉式"
    ```go
    type singleton struct{}
    
    var single *singleton
    
    func GetPersonInstance() *singleton {
        if single == nil {
            single = new(singleton)
        }
        return single
    }
    ```
    测试一下
    ```go
    func main() {
        s3 := GetPersonInstance()
        s4 := GetPersonInstance()
        fmt.Printf("%p\n", s3) // 0x104d5b8
        fmt.Printf("%p\n", s4) // 0x104d5b8
    }
    ```

=== "使用 sync.Once 实现的懒汉式"
    ```go
    type singleton struct{}
    
    var (
        single *singleton
        once   sync.Once
    )
    
    func GetPersonInstance() *singleton {
        once.Do(func() {
            single = new(singleton)
        })
        return single
    }
    ```
    测试一下：
    ```go
    func main() {
        s1 := GetPersonInstance()
        s2 := GetPersonInstance()
        fmt.Printf("%p\n", s1) // 0xdbd5b8
        fmt.Printf("%p\n", s2) // 0xdbd5b8
    }
    ```