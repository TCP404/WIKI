---
title: Python【No-12】私有化
tags: Python
categories:
  - Python
  - OPP
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 45109
date: 2020-07-13 10:41:48
---

类的私有化

<!--more-->

Python 是动态语言，可以在程序运行过程中动态地给class加上属性或方法。
这种不加以节制的特性很容易造成烂代码一堆。对此 Python 提供了一些解决方案。

## __slots__ 变量绑定白名单
> `__slots__ = (attributes_of_tuple)`
> 写在类变量处
> attributes_of_tuple是个元组，元组中每一个元素要用string型。

```python
class ClsName:
    __slots__ = ('name', 'age')
    ...
```

示例，在交互模式中：
```python
# 没有白名单，可以随意绑定属性
>>> class Person:
...     pass
...
>>> p = Person()
>>> p.name = 'Boii'
>>> p.age = 18
>>> p.score = 100

>>> print(p.name, p.age, p.score)
Boii 18 100
```

```python
# 使用了白名单，只能绑定白名单上的属性
>>> class Person:
...     __slots__ = ('name', 'age')
...
>>> p = Person()
>>> p.name = 'Boii'    # 白名单中的属性，可以绑定
>>> p.age = 18         # 白名单中的属性，可以绑定
>>> p.score = 100      # 白名单中没有的属性，绑定失败
# 白名单中没有 score，所以绑定失败
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Person' object has no attribute 'score'
```

示例，在模块文件中：

```python
class Person:

    __slots__ = ('score', 'gender')

    def __init__(self, name, age):
        self.name = name
        self.age = age

# 白名单中没有 name 和 age，所以连创建对象都会失败
person = Person('Boii', 20)

# Output:
AttributeError: 'Person' object has no attribute 'name'
```

```python
class Person:

    __slots__ = ('name', 'age', 'score', 'gender')

    def __init__(self, name, age):
        self.name = name
        self.age = age


p = Person('Boii', 20)

print(p.name, p.age)

p.name = 'Kali'     # 白名单中存在，可以修改
p.age = 30          # 白名单中存在，可以修改
p.score = 100       # 白名单中存在，可以绑定
p.gender = 'male'   # 白名单中存在，可以绑定

print(p.name, p.age, p.score, p.gender)

# Output:
Boii 20
Kali 30 100 male
```

> 使用__slots__要注意，`__slots__`定义的属性仅对当前类实例起作用，对继承的子类是不起作用的
> 除非在子类中也定义`__slots__`，这样，子类实例允许定义的属性就是自身的`__slots__`加上父类的`__slots__`。

## @property
当一个类中有不想给外界随便访问或修改的属性时，可以将该变量变成 private 型的
例如 `__age = 18`

这样的属性只能在类中访问和修改，外界要进行访问和修改只能通过类中的`getter、setter`方法。例如：

```python
class Person:
    def __init__(self, age):
        self.__age = age

    def get_age(self):
        return self.__age

    def set_age(self, value):
        if value > 0 and value < 200:
            self.__age = value

p = Person(20)

print(p.get_age())    # 20
p.set_age(18)
print(p.get_age())    # 18
```

通过 `getter、setter` ，在访问和修改的时候比较麻烦，要通过调用方法。
使用 `装饰器@property` 则可以比较方便，像变量一样去访问和修改

### 使用方法

```python
class ClsName:

    def __init__(self, attributeName):
        self.__attributeName = attributeName

    # 要先写getter
    @property
    def attributeName(self):
        return self.__attributeName

    # 再写setter
    @attributeName.setter
    def attributeName(self, value):
        self.__attributeName = value


cls = ClsName()

print(cls.attributeName)    # 访问
cls.attributeName = value   # 修改
```

现在改写上面 Person 类的例子：
```python
class Person:

    def __init__(self, age):
        self.__age = age

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, value):
        if value > 0 and value < 200:
            self.__age = value


p = Person(18)
print(p.age)    # 18
p.age = 20
print(p.age)    # 20
```
