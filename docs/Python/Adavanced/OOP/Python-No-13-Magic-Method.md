---
title: Python【No-13】魔术方法
tags: Python
categories:
  - Python
  - OPP
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 39743
date: 2020-07-14 10:41:48
---

魔术方法即类的内置方法

<!--more-->

## `__str__()`
> 触发时机:使用print(对象)或者str(对象)的时候触发
> 参数：一个self接收对象
> 返回值：必须是字符串类型
> 作用：print（对象时）进行操作，得到字符串，通常用于快捷操作
> 调用方式：`print(obj)`
>
> 另外，还有一个与`__str__()`相同的子方法：`__repr__()`，效果一样，适用于调试时，区别在于
> `>>> print(obj)` 时调用的是 `__str__()`
> `>>> obj` 时调用的是 `__repr__()`

```java
// Java

public class Person{

    String name = "";
    int age = 0;

    public Person(String name, int age){
        this.name = name;
        this.age = age;
    }

    ...

    public String toString() {
        return "name: " + this.name + ", age: " + this.age;
    }
}


Person p = new Person("Boii", 18);
system.out.println(p);

// Output:
// name: Boii, age: 18
```

```python
# Python

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return "name: " + self.name + ", age: " + self.age

    __repr__ = __str__    # 偷懒写法


p = Person("Boii", 18)
print(p)

# Output:
# name: Boii, age: 18
```

## `__call__()`
> 调用对象的魔术方法
> 触发时机:将对象当作函数调用时触发 对象()
> 参数:至少一个self接收对象，其余根据调用时参数决定
> 返回值：根据情况而定
> 作用：可以将复杂的步骤进行合并操作，减少调用的步骤，方便使用
> 调用方式：`obj()`

```python
class Athlete:
    def prepare(self):
        print('The athlete is preparing...')

    def warm_up(self):
        print('The athlete is warming up...')

    def attend(self):
        print('The athlete attended.')

    def run(self):
        print('The athlete ran.')

    def __call__(self):
        self.prepare()
        self.warm_up()
        self.attend()
        self.run()

athlete = Athlete()
athlete()

# Output:
The athlete is preparing...
The athlete is warming up...
The athlete attended.
The athlete ran.
```

示例中只是简单调用自身函数，实际过程中可能会有更多如开启线程，调用别的类等复杂操作

很多时候，需要判断一个对象能否被调用，能被调用的对象就是一个`Callable`
函数，和带有 `__call__()`的类对象就是Callable
可以通过 `callable(obj)`来判断一个对象是否可以被调用

## `__iter__()`
> 调用对象的魔术方法
> 触发时机：对象被用于 `for...in`
> 参数：至少一个self接收对象
> 返回值：根据情况而定
> 作用：使类具有可迭代的能力
> 调用方式：`for...in obj`

如果一个类想被用于for ... in循环，类似list或tuple那样，就必须实现一个__iter__()方法，
该方法返回一个迭代对象，
然后，Python的for循环就会不断调用该迭代对象的__next__()方法拿到循环的下一个值，
直到遇到StopIteration错误时退出循环。

以斐波那契数列为例，写一个Fib类，可以作用于for循环：

```python
class Fib(object):
    def __init__(self):
        self.a, self.b = 0, 1   # 初始化两个计数器a，b

    def __iter__(self):
        return self             # 实例本身就是迭代对象，故返回自己

    def __next__(self):
        self.a, self.b = self.b, self.a + self.b # 计算下一个值
        if self.a > 100000:     # 退出循环的条件
            raise StopIteration()
        return self.a           # 返回下一个值


>>> for n in Fib():
...     print(n)
...
1
1
2
3
5
...
46368
75025
```

## `__getitem__()`


## `__getattr__()`
> 触发时机：获取不存在的对象成员时触发
> 参数：1接收当前对象的self，一个是获取成员名称的字符串
> 返回值：必须有值
> 作用:为访问不存在的属性设置值
> 注意：getattribute无论何时都会在getattr之前触发，触发了getattribute就不会在触发getattr了
