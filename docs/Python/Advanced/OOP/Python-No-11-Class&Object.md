# Class&Instance

封装、继承、多态

## Class

!!! note ""
    类就像模具，用模具制作出来的产品叫做对象

```python
# 不带继承的类
class ClsName:    # 类名
    # 静态字段，通过类访问 print(ClsName.name), 在内存中只保存一份
    name = ''
    age = 0        # public类变量，任何区域都能访问
    _weight = 0    # protected类变量，当前类和子类和同一模块中才能访问
    __height = 0   # private类变量，只有当前类才能访问

    # 构造方法
    def __init__(self,[para1[, para2[, paraN]):
        """ 这里是类文档 """
        # 普通字段，通过实例来访问 print(clsInstance.para1), 每个实例中都保存
        self.para1 = para1
        self.para2 = para2

    # 类方法
    def method(self):
        """ 这里是方法注释 """
        <statement>

    # 类的私有方法
    def __private_method(self):
        <statement>

# 继承了别的类的类
class ClsName(BaseClsName1[, Base2[, Base3[, BaseN]):    # 括号内是这个类所继承的父类
    <statement>
```

### self

这是`方法`和`函数`的区别之处：不管是什么方法都要有`self`这个参数，再写其他参数。

这个`self`是唯一的不可缺少的，它等价于java 和 C++中的`this`。

但是调用方法的时候不需要给它赋值。

当这样调用对象的方法时：`myobject.method(arg1,arg2)`，解释器会解释为`MyClass.method(myobject, arg1, arg2)`，**这就是方法的自动传值**





### 类变量 VS 对象变量

类变量或者说静态字段都是一个东西。（类变量 == 静态字段、类属性）

对象变量或者说普通字段也都是一个东西。（对象变量 == 普通字段、实例属性）

顾名思义，`类变量`就是属于`类`的变量，`对象变量`就是属于`对象（实例）`的变量

#### 差别

**定义**：
```python
class Person:
    # 类变量、静态字段、类属性
    eyes = 2
    nose = 1

    def __init__(self, name, age):
        # 对象变量、普通字段、实例属性
        self.name = name
        self.age = age
```
![](![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902105833108_3400.png)


**实例化**：
```python
# 创建两个对象（实例化）
p1 = Person('Boii', 20)
p2 = Person('Cai', 18)


# 访问静态变量
print(Person.eyes)    # 2 通过类名访问
print(p1.eyes)        # 2 通过对象访问
print(p2.nose)        # 1 通过对象访问

# 访问对象变量
print(p1.name)   # Boii
print(p2.name)   # Cai
```
![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902105816461_6402.png)



**修改类变量以后**：
```python
# 修改静态变量
Person.eyes = 1

print(p1.eyes)    # 1
print(p2.eyes)    # 1
```
![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902105922748_5672.png)


```python
# 修改对象变量
p1.age = 50

print(p1.age)    # 50
print(p2.age)    # 18, 改了对象A的，对象B是不受影响的
```
![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902110710748_5406.png)


类变量，是属于类的，存在类那块内存中，类变量被该类创建出来的对象共享。

例如上图中，最上面的表格中就是类本身占的内存，其中就有类变量 eyes 和 nose

下面两个小表格就是对象 p1 和 p2 各自占的空间，其中就保存着对象变量 name，age

修改类变量 eyes 和 nose 后，对象 p1 和 p2 去访问 eyes 和 nose 就会访问到修改后的值

修改对象变量 name 或 age 后，对象之间互不影响。



假设一个类创建了两个对象A 和 B

这时内存中其实是有三块空间的，一块是类的，一块是对象A的，一块是对象B的。

当通过`类名.类变量`访问的时候，是访问`类那块内存`里的类变量的

当通过`类名.类变量`修改的时候，是修改`类那块内存`里的类变量的

当通过`对象.类变量`访问的时候，是系统跑去`类那块内存`里访问类变量的

当通过`对象.类变量`修改的时候，是系统跑去`类那块内存`里 **复制** 类变量到`对象那块内存`里的

此时通过`类名.类变量`修改，再通过`对象.类变量`访问，其实访问的是`对象自己内存`里的那个类变量

```python
class Person:
    # 类变量、静态字段
    eyes = 2

    def __init__(self, name, age):
        self.name = name
        self.age = age

# 创建两个对象
p1 = Person('Boii', 20)
p2 = Person('Cai', 18)


# 通过类 访问静态变量
print(Person.eyes)        # 2
# 通过对象 访问静态变量
print(p1.eyes)       # 2
print(p2.eyes)       # 2


### 通过类 修改静态变量
Person.eyes = 1
# 再通过类 访问静态变量
print(Person.eyes)        # 1
# 再通过对象 访问静态变量
print(p1.eyes)       # 1
print(p2.eyes)       # 1



### 通过对象 修改对象变量
p1.eyes = 50

# 再通过类 访问静态变量
print(Person.eyes)        # 1
# 再通过对象 访问静态变量
print(p1.eyes)       # 50    此时personA的内存里已经有eyes这个变量了
print(p2.eyes)       # 1
```
![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902111556373_11112.png)

注意 p1 中 已经多了一个 eyes 的变量了

#### 另外
**但是注意，如果静态变量是字典 dict，则不管怎么访问怎么修改，得到的都是一致的**

看下面的例子，类变量中有一个字典类型 d
```python
class Person:
    # 类变量、静态字段
    eyes = 2
    d = {1: 'A', 2: 'B'}

    def __init__(self, name, age):
        # 对象变量、普通字段
        self.name = name
        self.age = age
```

![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902142346472_20282.png)


创建两个对象，然后通过对象修改类变量，和通过对象修改字典类型类变量

```python
# 创建两个对象
p1 = Person('Boii', 20)
p2 = Person('Cai', 18)

# 通过类 修改静态变量, 改动的是类空间里的eyes
Person.eyes = 1

# 通过对象 修改静态变量, 是复制一份eyes到对象空间里并修改
p1.eyes = 3

# 通过对象 修改静态字典变量, 并不会复制一份d到对象空间里
p1.d[1] = 50
```
![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/IMG/20200902142543383_3873.png)

可以看到通过类名修改后，类空间里的eyes被修改了

通过对象修改类变量之后，对象空间里多了一个 eyes 变量并且已经修改了

通过对象修改字典类型类变量之后，没有复制一份，而是修改了类空间里字典的值

#### 小结

1. 类变量，又称静态字段、类属性
2. 对象变量，又称普通字段、实例属性
3. 类变量是所有对象共有的，对象变量是对象自己独有的
4. 类变量可以通过 `类名.类变量`、`对象.类变量`访问
5. 类变量可以通过 `类名.类变量`、`对象.类变量`修改
6. 通过 `对象.类变量` 修改时，除非该类变量是字典类型，否则都会把类变量复制一份到对象中去


### 实例方法、类方法、静态方法
#### 实例方法

!!! note ""
    最普通最常用的方法。类中定义的非私有的方法，每个对象在被创建以后都有自己的实例方法。

- 定义：第一个参数必须是实例对象，该参数名一般约定为“self”，通过它来传递实例的属性和方法（也可以传类的属性和方法）；
        如`p1.instanceMethod()`，Python 解释器会把对象 p1 传给self参数
- 调用：只能由实例对象调用。

```python
class Person:
    ...
    ...
    # 定义实例方法
    def walk(self, step):
        return f"I walked {step} steps."

p1 = Person()
print(p1.walk(5))        # 调用实例方法

-------------------------------
Output:

I walked 5 steps.
```
#### 类方法 @classmethod

!!! note ""
    属于类的方法，和类变量一样，所有对象共享类方法

- 定义：使用装饰器`@classmethod`。第一个参数必须是当前类对象，该参数名一般约定为“cls”，通过它来传递类的属性和方法（不能传实例的属性和方法）；

    如`p1.clsMethod` 或 `Person.clsMethod`，Python解释器会把类 Person 传给 cls 参数

- 调用：类对象或实例对象都可以调用。


```python
class Person:
    __count = 0

    def __init__(self):
        Person.__count += 1
    ...
    ...
    @classmethod
    def get_count(cls):
        return cls.__count
```

#### 静态方法 @staticmethod

用来存放逻辑性的代码，逻辑上属于类，但是和类本身没有关系，即，在静态方法中，不会涉及到类中的属性和方法的操作。

可以理解为静态方法是个 **独立的、单纯的** 函数，仅仅托关于某个类的名称空间中，便于使用和维护。

- 定义：使用装饰器`@staticmethod`。参数随意，没有“self”和“cls”参数，但是方法体中不能使用类或实例的任何属性和方法；
- 调用：类对象或实例对象都可以调用。

```python
class Person:
    __count = 0

    def __init__(self):
        Person.__count += 1
    ...
    ...
    @staticmethod
    def get_time():
        return time.time()
```


### 构造方法

!!! note ""
    `def __init__(self)`就是类的构造方法

构造方法的形参可以有1~N个。而`参数self`是必选的首个的，这个`self`相当于`this`。

`self`可以换成其他，但是为避免争议和歧义，最好用`self`

对于一个对象，可以自由的增加属性或者说对象变量、普通字段，但是通过构造方法，可以强制要求`在实例化对象时传入必须的参数`。

```python
class Animal:
    def __init__(self):
        pass

animalA = Animal()    # self 不需要传入
animalA.weigth = 180  # 可以自由的增加对象属性

class Person:
    def __init__(self, name, age):
        pass

personA = Person('Boii', 20)    # 创建对象时必须给 name 和 age

personB = Person()    # ！！错误
```




### 访问限制
    
!!! note ""
    不加下划线，仅变量名/方法名 = public，任何区域都可以访问

    一条下划线+变量名/方法名 = protected，`当前类`或`子类`或`同一模块`才可以访问

    两条下划线+变量名/方法名 = private， `当前类`才可以访问

!!! example

    === "public 示例"

        ```python
        # public 示例

        class Person:
            def __init__(self, name):
                self.name = name    # public 属性

            def show(self):
                print(self.name)    # 类内部可以访问


        person = Person('Boii')

        print(person.name)    # Boii 类外部也可以访问
        ```

    === "protected 示例"

        ```python
        ## Person.py
        class Person:
            def __init__(self, name):
                self._name = name    # protected 属性

            def show(self):
                print(self._name)     # 当前类中可以访问


        person = Person('Boii')

        person.show()                 # Boii
        ```

        ```python
        ## Student.py 
        from Person import Person


        class Student(Person):
            def __init__(self, name, age):
                super().__init__(name)    # 在子类中通过 super().__init__() 访问
                self.age = age

            def show(self):
                print(f'姓名：{self._name}, 年龄：{self.age}')


        student = Student('Boii', 20)

        student.show()            # 姓名：Boii, 年龄：20
        ```

    === "private 示例"
            
        ```python
        # private 示例

        class Person:
            def __init__(self, name):
                self.__name = name

            def show(self):
                print(self.__name)

            def get_name(self):
                return self.__name

            def set_name(self, name):
                self.__name = name

        person = Person('Boii')

        print(person.__name)        # 错误！！只能在类中访问私有变量

        print(person.get_name())    # Boii, 通过get方法访问

        person.set_name('Alice')    # 通过set方法改变

        print(person.get_name())    # Alice
        ```

private变量实际上是因为 python解释器对外把 `__name` 改成了 `_Person__name`，依然可以通过`person._Person__name`来访问，但是强烈建议不要这么做。

点击查看更多关于[访问限制、私有化]()的问题



## 封装、继承和多态

    
    面向对象三大特性：封装、继承、多态
    
    - **封装**

        把一类东西共通的属性、行为定义在一个类中，就是封装。


    
    - **继承**
    
        一个类，继承了别的类以后，这个类叫做`子类`
        
        被继承的类，叫做`基类、父类、超类`
        
        继承以后，子类就拥有了父类的全部 `非private 功能`
        
        Python中，子类可以同时继承多个父类


    
    - **多态**
    
        在子类中编写`与父类同名`的方法，叫做`方法重写`，
        
        适用于父类功能不能满足子类要求时。这称之为**多态**
        
        当父类子类的方法相同时，总是会优先调用子类的方法

```python
class Animal:
    def run(self):
        print('Running---')

class Dog(Animal):    # 继承父类 Animal
    pass


class Cat(Animal):    # 继承父类 Animal
    def run(self):    # 方法重写
        print('Cat is Running---')


dog = Dog()
dog.run()    # Running---

cat = Cat()
cat.run()    # Cat is Running---
```
### 继承

!!! note ""
    `class ClsName(BaseClass)`

在继承中，父类和子类都有的方法（同名的方法），会优先调用子类的；子类没有的，才调用父类的

子类中不定义构造方法`__init__()`，会调用父类的构造方法`__init__()`

#### 单继承

在 **单继承** 中：如果父类构造方法有参数，则子类必须有构造方法，并调用父类的构造方法且传参

```python
# 子类没有__init__，默认调用父类的__init__
# 父类的__init__没有参数，所以子类可以不写__init__

class Person:
    def __init__(self):
        self.name = "Anonymity"
        self.age = 18


class Student(Person):
    pass

s = Student()

```

```python
# 父类的__init__有参数，子类必须有__init__，并调用父类的__init__且传参

class Person:
    def __init__(self, name):
        self.name = name


class Student(Person):
    def __init__(self, name, age):
        self.age = age
        super().__init__(name)


s = Student("Boii", 18)
```

#### 多继承

在 **多继承** 中：

- 如果子类没有`__init__()`，会调用第一个父类的`__init__()`
- 如果第一个父类没有`__init__()`，会找第二个父类，以此类推...

其中，只要任何一个父类的`__init__()`有参数，子类就必须有`__init__()`来调用父类的`__init__()`

如果有超过一个父类的`__init__()`有参数，则应该写作 `BaseClsName.__init__(self, paras)` 或 `BaseClsName(type, obj).__init__(paras)`

```python
class BaseA:
    def __init__(self, name, age):
        self.name = name
        self.age = age


class BaseB:
    def __init__(self, gender, nationality):
        self.gender = gender
        self.nationality = nationality


class Student(BaseA, BaseB):
    def __init__(self, name, age, gender, nationality):

        BaseA(Student, self).__init__(name, age)

        BaseB.__init__(self, gender, nationality)


s = Student("Boii", 18, "male", "China")
```

#### 钻石继承
    
!!! note ""
    菱形继承是指：子类 sub 继承了 父类 A，B，而父类 A，B又共同继承了祖父类 Base

```
     [Base]
    ↗     ↖
  [A]      [B]
    ↖     ↗
     [sub]
```

由于Python的机制，解释器在创建 sub类对象时会找到 A 的 构造方法，接着找到 Base 的构造方法

然后再找 B 的构造方法，接着找到 Base 的构造方法，这样等于重复调用了 Base 的构造方法

```python
class Base:
    def __init__(self):
        print("Base.__init__")


class A(Base):
    def __init__(self):
        Base.__init__(self)
        print("A.__init__")


class B(Base):
    def __init__(self):
        Base.__init__(self)
        print("B.__init__")


class sub(A, B):
    def __init__(self):
        A.__init__(self)
        B.__init__(self)
        print("sub.__init__")


sub()

# Output:
Base.__init__
A.__init__
Base.__init__
B.__init__
sub.__init__
```
Base 的 `__init__()` 被调用了两次

这时候可以改写 sub，A，B 的 `__init__()`为`super().__init__`

```python
class Base:
    def __init__(self):
        print("Base.__init__")


class A(Base):
    def __init__(self):
        super().__init__()
        print("A.__init__")


class B(Base):
    def __init__(self):
        super().__init__()
        print("B.__init__")


class sub(A, B):
    def __init__(self):
        super().__init__()
        print("sub.__init__")


sub()

# Output:
Base.__init__
B.__init__
A.__init__
sub.__init__
```



### 多态的好处

如上示例，多态使得继承之后还可以进行扩展，但是多态还有另一个好处。

示例中，Animal 是父类，Dog 和 Cat 是子类。

Animal 是 Animal 类型；Dog 是 Dog 类型，也是 Animal 类型；Cat 同理

```python
>>> animal = Animal()
>>> dog = Dog()
>>> cat = Cat()

>>> isinstance(animal, Animal)
True
>>> isinstance(dog, Dog)
True
>>> isinstance(dog, Animal)
True
>>> isinstance(cat, Cat)
True
>>> isinstance(cat, Animal)
True
>>> isinstance(animal, Dog)
False
```

!!! note ""
    **一句话总结就是，对象的类型是`自身类+父类`，而所有类都自动继承自 `object类`**

因为这个特性，使得python更加灵活

假设现在有个函数，这个函数接受一个animal类型对象

```python
def run_twice(animal):
    animal.run()
    animal.run()
```

在调用的时候：
```python
>>> animal = Animal()
>>> dog = Dog()
>>> cat = Cat()

>>> run_twice(animal)
Running---
Running---

>>> run_twice(cat)
Cat is Running---
Cat is Running---

>>> run_twice(dog)
Running---
Running---
```

可以发现，这个接受Animal类型的函数，**不仅可以接受Animal类及其子类的对象，还可以根据传入对象的不同实现不同的效果**

即使再定义一个类继承Animal，然后创建对象传入 `run_twice()`，依然可以实现相同的效果，而且不需要改动`run_twice()`

同理，所有的类都继承自`object`，如果是`run_twice(object)`，则可以接受任何类型的对象

所以，对于一个变量，只需要知道其父类`Animal`，就可以放心的使用，在调用时`animal.run()`是作用在animal还是dog还是cat，由运行时该对象的确切类型决定。

调用方只管调用，不管细节。每当新增一种`Animal子类`时，只要确保`run()`方法编写正确，不用管`run_twice()`怎么实现。

这就是`<开闭原则>`:

- 对扩展开放：允许新增`Animal`子类；
- 对修改封闭：不需要修改`依赖Animal类型`的`run_twice()`等函数

#### 多态在静态语言和动态语言中的区别

多态的这种特性在`动态语言`和`静态语言`中还有些区别.

像java这种静态语言，`run_twice(Animal)`传入的对象必须是`Animal类型或其子类`，否则无法调用`run()`方法。

而python这种动态语言，则不一定要传入Animal类型，只需要保证传入的对象有一个`run()`方法就可以。

**这就是动态语言的`“鸭子类型”`，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。**


```python
class Timer:
    def run(self):
        print('Start---')
```

像这个类，没有继承Animal，但依然可以传给`run_twice(Animal)`。

示例：
```python
class Animal(object):   # 编写Animal类
    def run(self):
        print("Animal is running...")

class Dog(Animal):      # Dog类继承Amimal类，没有run方法
    pass

class Cat(Animal):      # Cat类继承Animal类，有自己的run方法
    def run(self):
        print('Cat is running...')
    pass

class Car(object):      # Car类不继承，有自己的run方法
    def run(self):
        print('Car is running...')

class Stone(object):    # Stone类不继承，也没有run方法
    pass

def run_twice(animal):
    animal.run()
    animal.run()

run_twice(Animal())
run_twice(Dog())
run_twice(Cat())
run_twice(Car())
run_twice(Stone())
```

输出：
```
Animal is running...
Animal is running...
Animal is running...
Animal is running...
Cat is running...
Cat is running...
Car is running...
Car is running...

AttributeError: 'Stone' object has no attribute 'run'
```