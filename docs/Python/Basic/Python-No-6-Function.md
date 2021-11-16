---
title: Python【No-6】函数
tags: Python
categories:
  - Python
  - 基础
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 9049
date: 2020-07-07 11:17:48
---

一等公民：函数

<!--more-->

## 定义函数
```python
def functionName(paramters):
    function body
    [return]
```

## 调用函数
```python
funcName([paras])
```

## 返回值
> 不管有没有 return 函数都会返回
> 有 return 时函数返回相应的值
> 没有 return 时函数返回 None

> `return None` 可以简写为 `return`

### 返回多个值
python 函数可以返回多个值
但其本质其实是返回一个多元素的Tuple，因为语法上tuple可以省略括号，所以看起来像返回多个值
如果显式的返回一个list，dict，tuple，set，则是返回值本身

```python
import math

def move(x, y, step, angle=0):
    nx = x + step * math.cos(angle)
    ny = y - step * math.sin(angle)
    return nx, ny

>>> x, y = move(100, 100, 60, math.pi / 6)
>>> print(x, y)
151.96152422706632 70.0

>>> r = move(100, 100, 60, math.pi / 6)
>>> print(r)
(151.96152422706632, 70.0)
```

## 空函数
> 空函数可以作为占位符，比如现在还没想好怎么写函数的代码，就可以先放一个pass，让代码能运行起来。

```python
def func():
    pass
```
## 参数
### 参数检查
> python解释器会检查参数 `个数` ，但不会检查参数 `类型`
> 类型检查需要自己做
> 可以使用内置函数 `isinstance(obj,class_of_tuple)` 并抛出错误

```python
def my_abs(x):
    return x if x >= 0 else -x

# 添加检查类型
def my_abs(x):
    if not isinstance(x, (int, float)):        # 检查类型
        raise TypeError('bad operand type')    # 抛出错误
    return x if x >= 0 else -x
```

#### isinstance()
> isinstance(被检查对象,检查类型元组)
isinstance() 接受两个参数，第一个是被检查对象，第二个是要求的类型元组
类型元组中只要有一个被满足，则返回True，反之返回False

### 默认参数、可变参数和关键字参数
#### 默认参数 =
> `def funcName( [para1 ... ,] paraN = defaultValue)`
> 当参数的值不确定的时候，就可以使用默认参数
> 
> 默认参数 `para1 = defaultValue ` 必须放在最后面
> 当有多个默认参数的情况下，想指定某个默认参数不使用默认，可以将默认参数的形参名带上
> 默认参数必须指向**不变对象！**

```python
def power(x,n=2):
    s = 1
    while n > 0:
        s *= x
        n -= 1
    return s

>>> power(5)
25
>>> power(5, 2)
25
>>> power(5, 3)
125
```

当有多个默认参数的情况下，想指定某个默认参数不使用默认，可以将默认参数的形参名带上

```python
def enroll(name, gender, age=6, city='SWA'):
    print(name)
    print(gender)
    print(age)
    print(city)

>>> enroll('Boii', 'M', 23, 'BJ')
Boii
M
23
BJ
>>> enroll('Boii', 'M', 23)
Boii
M
23
SWA
>>> enroll('Boii', 'M', city='NJ')
Boii
M
6
NJ
>>> enroll('Boii', 'M')
Boii
M
6
SWA
```

默认参数必须指向**不变对象！**
```python
def add_end(L=[]):
    L.append('END')
    return L

>>> add_end([1,2,3])
[1,2,3,'END']
>>> add_end(['x', 'y', 'z'])
['x', 'y', 'z', 'END']

# 坑来了
>>> add_end()
['END']            # 这里还是正常的
>>> add_end()
['END', 'END']     # 这里就不正常了
```
猜测可能是python内部在使用默认参数的时候会创建对象，比如这个list，但是这个对象不会被销毁导致的

改进：
```python
def add_end(L=None)
    if L is None:
        L = []
    L.append('END')
    return L
    
>>> add_end()
['END']
>>> add_end()
['END']
```
#### 可变参数 *
> 可变参数可以接收 0~N个参数
>
> `def funcName( [para1 ... ,] *paraN)`
> 当参数的个数不确定的时候，就可以使用可变参数 `*paraN`
> 形参和实参都可以使用可变参数。
> `形参` 使用可变参数，则是告诉调用者我可以接受0~N个参数，
> `实参` 使用可变参数，则是告诉函数我已经打包好了。

可变参数的 `实际原理`：
其实可变参数在函数被调用时，将所有参数自动打包成一个tuple，然后传递给函数
函数接收到之后再解构这个tuple，或者通过 `for 迭代` 遍历这个tuple来使用里面的元素

```python
def calc(*numbers):        # 形参 使用可变参数，则是告诉调用者我可以接受0~N个参数
    sum = 0
    for n in numbers:
        sum += n*n
    return sum

>>> calc(1, 2)
5
>>> calc()
0

>>> nums = [1, 2, 3]
>>> calc(nums[0], nums[1], nums[2]) # 这种分散一个个的参数在传递之前也是被自动打包成一个tuple的
14
>>> calc(*nums)            # 实参 使用可变参数，则是告诉函数我已经打包好了
14
```

#### 关键字参数 **
> 这种方式，调用者可以传入 0~N 个关键字参数，关键字 key 随意
>
> `def funcName( [para1 ... ,] **key_paraN)`
> 即通过`关键字`来指定参数，或者说通过`键-值对`的方式来传实参
> 而在函数里，形参 关键字参数 `**paraN` 将传过来的这些键-值对打包成一个dict
>
> 其原理跟可变参数类似
> - 可变参数 `*paraN` 是把 `除了必选参数外的所有参数` 打包成一个tuple传给函数
> - 关键字参数 `**paraN` 是把 `除了必选参数外的所有参数` 打包成一个dict传给函数
>
> 之后函数内部可以通过 `for 迭代` 遍历这个dict来使用里面的键-值对
> 因为是使用键-值对的方式，所以传参时的 `传给关键字参数的`参数顺序是无所谓的

```python
def person(name, age, **kv):
    print('name:', name, '--' , 'age:', age, '--' , 'other:' kv)


# 只填必选参数的调用方法
>>> person('Boii', 23)
name: Boii -- age: 23 -- other: {}


# 填了必选参数和自定义参数的调用方法
>>> person('Boii', 23, city='SWA')
name: Boii -- age: 23 -- other: {'city':'SWA'}

>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam -- age: 45 -- other: {'gender': 'M', 'job': 'Engineer'}

# 关键字实参的顺序无所谓
>>> person('Adam', 45, job='Engineer', gender='M')
name: Adam -- age: 45 -- other: {'job': 'Engineer', 'gender': 'M'}

# 还可以自己把参数打包成dict，然后传给函数, 原理和可变参数类似
>>> d = {'job': 'Engineer', 'gender': 'M'}
>>> person('Adam', 45, **d)
name: Adam -- age: 45 -- other: {'job': 'Engineer', 'gender': 'M'}
```


#### 命名关键字参数 *，
> `def funcName( [para1 ... ,] *, key_paraN)`
> 对比关键字参数 ↓
> `def funcName( [para1 ... ,] **key_paraN)`
>
> 区别于关键字参数：
> - 如果想要求调用者在调用的时候必须用你给的 key，则可以用 命名关键字参数 `*, key_paraN`
> - 这种方式，调用者必须 遵循个数 和 关键字

```python
def person(*, city, nationality):
    print('city:', city)
    print('nationality:', nationality)

>>> person(city='SWA', nationality='China')
city: SWA
nationality: China

>>> person(nationality='China', city='SWA')
city: SWA
nationality: China

# 错误示例
>>>  person()
TypeError: person() missing 2 required keyword-only arguments: 'city' and 'nationality'

>>>  person('SWA', 'China')
TypeError: person() takes 0 positional arguments but 2 were given
```

```python
def person(name, age, *, city, nationality):
    print('name:', name)
    print('age:', age)
    print('city:', city)
    print('nationality:', nationality)

>>> person('Boii', 23, city='SWA', nationality='China')
name: Boii
age: 23
city: SWA
nationality: China

>>> person('Boii', 23, nationality='China', city='SWA')
name: Boii
age: 23
city: SWA
nationality: China

# 错误示例
>>> person(city='SWA', nationality='China')
TypeError: person() missing 2 required positional arguments: 'name' and 'age'
```

#### 参数组合
> 在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数
> 这5种参数都可以组合使用。
> 但是请注意，参数定义的顺序必须是：
> `必选参数`、`默认参数`、`可变参数`、`命名关键字参数`和`关键字参数`。

```python
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)

>>> f1(1, 2)
a = 1 b = 2 c = 0 args = () kw = {}

>>> f1(1, 2, c=3)
a = 1 b = 2 c = 3 args = () kw = {}

>>> f1(1, 2, 3, 'a', 'b')
a = 1 b = 2 c = 3 args = ('a', 'b') kw = {}

>>> f1(1, 2, 3, 'a', 'b', x=99)
a = 1 b = 2 c = 3 args = ('a', 'b') kw = {'x': 99}


>>> f2(1, 2, d=99, ext=None)
a = 1 b = 2 c = 0 d = 99 kw = {'ext': None}
```

最神奇的是通过一个tuple和dict，你也可以调用上述函数：

```python
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)


>>> args = (1, 2, 3, 4)
>>> kw = {'d': 99, 'x': '#'}

>>> f1(*args, **kw)
a = 1 b = 2 c = 3 args = (4,) kw = {'d': 99, 'x': '#'}



>>> args = (1, 2, 3)
>>> kw = {'d': 88, 'x': '#'}

>>> f2(*args, **kw)
a = 1 b = 2 c = 3 d = 88 kw = {'x': '#'}
```

> 所以，对于任意函数，都可以通过类似`func(*args, **kw)`的形式调用它，无论它的参数是如何定义的。
**！！虽然可以组合多达5种参数，但不要同时使用太多的组合，否则函数接口的可理解性很差。**

## 递归函数
> 自己调用自己
> 递归要素：
> 1. 自己调用自己
> 2. 有判断条件        （没判断就死循环了，会导致溢出）
> 3. 有变量迭代        （没变量迭代就死循环了）

```python
def fact(n):
    return 1 if n == 1 else n * fact(n-1)

>>> fact(5)
120
>>> fact(10)
3628800
```

计算 fact(5) 的时候
> ===> fact(5)
> ===> 5 * fact(4)
> ===> 5 * (4 * fact(3))
> ===> 5 * (4 * (3 * fact(2)))
> ===> 5 * (4 * (3 * (2 * fact(1))))
> ===> 5 * (4 * (3 * (2 * 1)))
> ===> 5 * (4 * (3 * 2))
> ===> 5 * (4 * 6)
> ===> 5 * 24
> ===> 120