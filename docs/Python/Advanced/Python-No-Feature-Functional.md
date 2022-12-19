---
title: Python【Feature】函数式编程
tags:
  - Python
  - 函数式编程
categories:
  - Python
  - 高级特性
thumbnail: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/thumbnail/python.png'
date: 2020-07-25 12:17:48
---

Python的高级特性2

<!--more-->

函数式编程的一个特点就是，允许把函数本身作为参数传入另一个函数，还允许返回一个函数！

## 函数的本质
在学C语言指针的时候，会接触到一个概念：函数名是函数所在内存的地址，类似-数组名是数组的首元素的地址。

即使Python中没有指针一说，但是本质还是一样的。
变量名是一个变量的地址
函数名是一个函数的地址
调用函数的时候其实是把参数传到函数本身所在的那块内存中去，函数名不过是一个媒介而已。
在C中可以说，函数名就是个指针，指向一个函数
在Python中可以说，函数名就是个变量，指向一个函数

既然函数名只是指向函数本身所在地址，那我换个名来指不也可以。

```python
def sum(a, b):
    return a + b

f = sum
f(1, 2)

---------------------------------

# Output:
3
```
如果放到交互模式中，可以更明显的看出来

```python
# 定义一个函数
>>> def sum(a, b):
...     return a + b
...
>>> sum
<function sum at 0x000001F25AF6B280>

>>> f=sum
>>> f
<function sum at 0x000001F25AF6B280>
>>> f(3, 4)
7
```

大概就这么回事儿

![1](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/No-Feature-1.png)

## 高阶函数
> 高阶函数：接受函数作为参数的函数
> `def heigh_fn(a, b, fn)`

既然变量可以指向函数，函数的参数能接收变量，那么一个函数就可以接收另一个函数作为参数，这种函数就称之为高阶函数。

一个最简单的高阶函数：
```python
def abs(num):
    return (num) if num >= 0 else (-num)

def abs_add(a, b, fn):
    return fn(a) + fn(b)

print(abs_add(8, -4, abs))

---------------------------------

# Output:
12
```

这个高阶函数的过程大概是这样的
a = 8
b = -4
fn = abs
fn(a) + fn(b) ==> abs(8) + abs(-4) ==> 12


## 返回函数
> 返回函数：返回值为函数的函数

通常函数都是返回int、str、list 这些变量，但是变量可以指向函数，
那函数可以返回一个变量，不就也可以返回一个函数。

举个栗子
实现一个求和函数。参数可变，可以是1个也可以是多个。
```python
def cal_sum(*args):
    sum = 0
    for n in args:
        sum += n
    return sum


result = cal_sum(1, 2, 3, 4, 5)
print(result)

---------------------------------

# Output:
15
```
好，现在再写一个函数`lazy_sum()`，把这个函数`cal_sum()`作为返回值返回
```python
def lazy_sum(*args):
    def cal_sum():
        sum = 0
        for n in args:
            sum += n
        return sum
    return cal_sum
```
可以看到最后一句`return cal_sum`就把函数返回出来了。
那么函数`lazy_sum()`就是一个返回函数

现在调用一下这个返回函数
```python
def lazy_sum(*args):
    print('I am the return function.')
    def cal_sum():
        sum = 0
        for n in args:
            sum += n
        return sum
    return cal_sum


result = lazy_sum(1, 2, 3, 4, 5)
print(result)

---------------------------------

# Output:
I am the return function.
<function lazy_sum.<locals>.cal_sum at 0x000001C8765CB310>
```
哦嚯，`lazy_sum()`没有进行计算，但也看出来，`lazy_sum()`返回的是一个函数
`result`此时是一个函数，这时候我们要去调用一次返回值`result`，才会进行计算
```python
def lazy_sum(*args):
    print('I am the return function.')
    def cal_sum():
        sum = 0
        for n in args:
            sum += n
        return sum
    return cal_sum


result = lazy_sum(1, 2, 3, 4, 5)
print(result)

res = result()
print(res)

---------------------------------

# Output:
I am the return function.
<function lazy_sum.<locals>.cal_sum at 0x000001E02D07B310>
15
```
这就是为什么说 返回函数是`惰性`的。就是这么来的

### 为什么要返回一个函数
在`lazy_sum()`中，有一句`print`语句。
从输出中可以看出，返回函数本身是立即执行的，但是它返回出来的函数要再进行一次调用才会执行。

单独一个返回函数并没有太大作用，但是返回函数和高阶函数结合起来，是不是就有意思了。
接收一个函数，然后把这个函数返回出去。在返回之前是不是可以进行一些操作？


假如现在有一个需求：求和；实现完了之后，产品经理又加需求：除了`求和`还要打印一下求和用了多长时间，也就是`计时功能`。
那么现在你有两个办法
- 一是在`求和函数`里实现这个`计时功能`，但是这样就在一个函数里做了两件事情。即不优雅，也不好维护。这里举例是简单的求和，实际开发中可能是跟复杂的功能。
- 二是把`计时函数`写出来，然后把`求和函数`传给`计时函数`，在`计时函数`中给`求和函数`加上`计时功能`，之后再把这个`有计时功能的求和函数`返回出去。

`计时函数`这种`接收函数 -> 增强函数 -> 返回函数`的函数就叫`闭包`

不仅可以用在`计时功能`，还能用在`日志功能`，我只要把`求和函数`写好，传给`日志函数`就可以获得`日志功能`。
函数只做一件事情，其他事情由别的函数做，需要什么功能把函数往里面一丢，还你一个增强版的。Amazing！这就叫`面向切面编程`


## 闭包、装饰器、面向切面编程
### 闭包
> 闭包 = 高阶函数 + 返回函数
> 即，一个`接收函数作为参数，又返回一个函数`的函数，就叫做闭包
> 作用：增强函数功能、面向切面编程（AOP）

#### 无参闭包
```python
import time

def print_hello():
    print('hello!')

def cal_time(fn):
    def improve_fn():
        start_time = time.time()
        fn()
        end_time = time.time()
        print('耗时：{}'.format(end_time - start_time))

    return improve_fn

improve_print_hello = cal_time(print_hello)
improve_print_hello()

# 也可以写做 cal_time(print_hello)()

---------------------------------

# Output:
hello!
耗时：0.0009620189666748047
```
栗子中，`print_hello()`只负责打印hello，`cal_time()`负责把传进来的函数增强然后返回出去
所以`cal_time`就是一个闭包

#### 带参闭包
上面的栗子，`print_hello()`不需要参数，所以在无参闭包中可以顺利通过
但是如果一个函数有参，那无参闭包就会抛出`TypeError`错误。

想要带参很简单。

{% codeblock lang:python line_number:true mark:9,11 %}
import time


def add(a, b):
    print(a + b)


def cal_time(fn):
    def improve_fn(*args, **kwargs):    # 在这里写下形参
        start_time = time.time()
        fn(*args, **kwargs)        # 调用的时候传进去
        end_time = time.time()
        print('耗时：{}'.format(end_time - start_time))

    return improve_fn


cal_time(add)(1, 2)

---------------------------------

# Output:
3
耗时：0.00800633430480957
{% endcodeblock %}


#### 接收的函数有返回值
接收的函数有返回值的话，应该在闭包里对函数调用的时候把参数接收下来，然后进行返回。

```python
import time


def add(a, b):
    return a + b


def cal_time(fn):
    def improve_fn(*args, **kwargs):
        start_time = time.time()
        count = fn(*args, **kwargs)     # 调用函数之后用一个变量接收返回值
        end_time = time.time()
        print('耗时：{}'.format(end_time - start_time))
        return count                    # 然后把返回值返回出去

    return improve_fn


result = cal_time(add)(5, 12)
print(result)

---------------------------------

# Output:
耗时：0.0009620189666748047
17
```

### 装饰器

> 装饰器是闭包的语法糖——`@闭包名`或`@闭包名(参数)`
> 语法糖只是一个更方便的写法，完全可以等价转换为非语法糖的代码

装饰器、闭包，本质上也是函数，说装饰器的同时也是在说闭包

#### 保留原函数的元信息
装饰器（闭包）的原理就是把原函数当作参数传进装饰器中，经过增强后返回一个新的`增强版原函数`
这说明返回出来的`增强版函数`和原来的函数不是同一个了（在内存中创建了一个新的）

既然不是在原函数上进行改造，这就导致函数的元信息（文档字符串、注解、参数签名等）都丢失了
解决方法就是使用`functools`库中的`@wraps`装饰器来把原函数的元信息复制过来。

先看看`不`保留元信息的样子
```python
import time


# 装饰器函数
def cal_time(fn):
    '''
    Decorator that report the execution time.
    '''
    def improve_fn(*args, **kwargs):
        start = time.time()
        result = fn(*args, **kwargs)
        end = time.time()
        print(fn.__name__, end - start)
        return result

    return improve_fn


# 原函数
@cal_time
def countdown(n):
    '''
    Count down
    '''
    while n > 0:
        n -= 1


if __name__ == '__main__':
    countdown(10_000)
    print(countdown.__doc__)    # 没有使用wrap，所以元信息都丢失了

---------------------------------

# Output:
countdown 0.0010008811950683594
None
```
再看看使用了`@wraps`的样子
```python
import time
from functools import wraps

# 装饰器函数
def cal_time(fn):
    '''
    Decorator that report the execution time.
    '''
    @wraps(fn)
    def improve_fn(*args, **kwargs):
        start = time.time()
        result = fn(*args, **kwargs)
        end = time.time()
        print(fn.__name__, end - start)
        return result

    return improve_fn


# 原函数
@cal_time
def countdown(n):
    '''
    Count down
    '''
    while n > 0:
        n -= 1


if __name__ == '__main__':
    countdown(10000)
    print(countdown.__doc__)    # 使用了wrap，所以元信息没有丢失

---------------------------------

# Output:
countdown 0.0010025501251220703

    Count down

```

#### 无参装饰器

```python
import time
from functools import wraps


def cal_time(fn):
    '''
    Decorator that report the execution time.
    '''
    @wraps(fn)
    def improve_fn(*args, **kwargs):
        start = time.time()
        result = fn(*args, **kwargs)
        end = time.time()
        print(fn.__name__, 'used time:', end - start)
        return result

    return improve_fn


# 装饰器里不需要携带参数
@cal_time
def add(a, b):
    return a + b

res = add(5, 6)
print(res)

# 等价于
#
# def add(a, b):
#     return a + b
#
# improve_add = cal_time(add)
# res = improve_add(5, 6)
# print(res)

---------------------------------

# Output:
add used time: 0.0
11
```

#### 带参装饰器
```python
import time
from functools import wraps

def cal_time(msg):    # 第一层，接收装饰器参数
    print('First floor:', msg)

    def decorator(fn):    # 第二层，接收函数
        print('Second floor:', msg)

        @wraps(fn)
        def improve_fn(*args, **kwargs):    # 第三层，增强函数
            print('Third floor:', msg)

            start = time.time()
            result = fn(*args, **kwargs)
            end = time.time()

            print('Used time:', end - start)
            return result

        return improve_fn

    return decorator


@cal_time('I am the message.')
def countdown(time=10000):
    while time > 0:
        time -= 1


if __name__ == '__main__':
    countdown()

---------------------------------

# Output:
First floor: I am the message.
Second floor: I am the message.
Third floor: I am the message.
Used time: 0.000997781753540039
```
有参装饰器和无参装饰器的区别仅仅只是多了一层函数而已
也就是在无参装饰器的基础上多一层函数来接收装饰器参数而已。**注意带参装饰器即使不填参数也要写上括号**

> 装饰器只在第一次被调用去装饰函数时对原函数进行增强
> - 增强时机：第一调用之前
> - 增强次数：只增强一次





### 面向切面编程

如果说类里面的继承是纵向继承，那面向切面编程可以理解为横向继承

纵向继承中，子类继承了父类非私有属性和方法；同属一个父类的子类都有相同的父类属性和方法
横向继承中，是给子类增加了一些功能；同属一个父类的子类不见得有相同的功能。

就好比，一个爹生了两个娃，两个娃都有手有脚的，这是继承自爹那里的（纵向继承）
但是一个娃点亮了游泳的技能，另一个娃点亮了跳舞的技能（横向继承）
这俩娃生了俩孙子，肯定也是有手有脚的，但不一定会游泳跳舞。

所以面向切面编程中，这种闭包、装饰器之类的通常是用在编写通用工具类上。


## 匿名函数
> `lambda [parameters]: expression`
冒号前面的`parameters`表示匿名函数的形参列表，冒号后面的`expression`表示要return的语句
`parameters`可以有也可以没有。
匿名函数有个限制——**只能有一个表达式**，不用写`return`，返回值就是该表达式的结果。

匿名函数类似于 es6 中的简头函数

举个栗子：
```python
def add(a, b):
    return a + b
```

写成匿名函数就是
```python
lambda a, b: a + b
```
写成简头函数就是
```javascript
(a, b) => a + b
```

匿名函数可以作为实参、返回值，在这些场景中应用的比较多。

匿名函数作为实参
```python
# 普通写法
def abs(num):
    return (num) if num >= 0 else (-num)

def abs_add(a, b, fn):
    return fn(a) + fn(b)

print(abs_add(8, -4, abs))

---------------------------------
# 匿名函数写法
def abs_add(a, b, fn):
    return fn(a) + fn(b)

result = abs_add(8, -4, lambda x: x if x >= 0 else -x)    # 匿名函数作为实参
print(result)

---------------------------------

# Output:
12
```

匿名函数作为返回值
```python
def build(x, y):
    return lambda: x * x + y * y        # 匿名函数作为返回值

build(2, 3)

---------------------------------

# Output:
13
```

## 总结
1. 函数名的本质：函数起始地址
2. 高阶函数：接收函数作为参数的函数
3. 返回函数：返回一个函数（即返回一个函数地址）的函数，是惰性的
4. 闭包：接收函数作为参数，并返回一个函数的函数（闭包=高阶函数+返回函数）
5. 装饰器：闭包的语法糖，@闭包名字
6. 匿名函数：`lambda 参数: 返回值`