---
title: Python【Feature】工具类
tags:
  - Python
  - 函数式编程
categories:
  - Python
  - 高级特性
thumbnail: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/thumbnail/python.png'
abbrlink: 18591
date: 2020-07-24 13:17:48
---

Python的高级特性3

<!--more-->

## map
将传入的函数作用于可迭代对象的每一个元素上

> `map(function, iterable)`
参数：map 可以接受两个参数，第一个是函数，第二个是可迭代对象
作用：将可迭代对象的每一个元素放在 function 中进行处理
返回：map 返回的是一个 map 对象，这是一个可迭代对象

![map](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/No-Feature-map.png)

举个栗子：
```python
# 使用map示例
alpha = ['a', 'b', 'c', 'x', 'y', 'z']
m = map(lambda x: x + 'i', alpha)
print(m)
print(list(m))    # 返回的map对象用m接收，然后转换成列表，打印出来

--------------------------------------------------

# Output:
<map object at 0x000001D50CEBAD00>
['ai', 'bi', 'ci', 'xi', 'yi', 'zi']
```

如果不使用 map 函数，应该这样写
```python
# 不适用map的示例
alpha = ['a', 'b', 'c', 'x', 'y', 'z']

def add_alpha(list1):
    """ 遍历list1，每个元素都加上 i 然后放进 list2 中 """
    list2 = list()
    for alpha in list1:
        list2.append(alpha + 'i')
    return list2

m = add_alpha(alpha)
print(m)
print(type(m))

--------------------------------------------------

# Output:
['ai', 'bi', 'ci', 'xi', 'yi', 'zi']
<class 'list'>
```
效果是一样的，但是并没有像 map 那么简洁

同样，上面的示例还可以使用更简洁的办法，用列表生成式
前提是其逻辑比较简单，如果复杂还是要另外编写 操作函数，然后传给 map()
```python
alpha = ['a', 'b', 'c', 'x', 'y', 'z']
print([x + 'i' for x in alpha])

--------------------------------------------------

# Output:
['ai', 'bi', 'ci', 'xi', 'yi', 'zi']
```

此外，因为返回出来的已经不是原本放进去的可迭代对象了，所以即使元组不能修改，map 也能进行操作
```python
# 对元组进行操作
m = (12, 23, 28, 19, 16, 30)
print('m source:  ', id(m))    # 打印原元组的地址
m = map(lambda x: x * x, m)    # 对元组进行操作
print('m operated:', id(m))    # 打印修改后元组的地址
print('m is a', m)             # 打印结果显示 m 是个 map对象
print(list(m))                 # 把 m 转成列表打印出来

--------------------------------------------------

# Output:
m source:   2599500087840
m operated: 2599500555024
m is a <map object at 0x0000025D3E4EA310>
[144, 529, 784, 361, 256, 900]
```


## filter
对可迭代对象进行过滤
Python2 中 filter 是个内置函数, Python3 中 filter 是个内置类

> `filter(condition_function, iterable)`
参数：filter 可以接收两个参数，第一个是函数，第二个是可迭代对象
作用：将 可迭代对象 的每一个元素放在 function 中进行过滤，满足 function 中的过滤规则便返回
返回：filter 返回的是一个 filter对象，这是一个可迭代对象。

![filter](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/No-Feature-filter.png)

```python
# filter 示例
f = [12, 23, 28, 19, 16, 30]
print('f source:  ', id(f))             # 打印原元组的地址
f = filter(lambda x: x >= 18, f)        # 对 列表进行过滤，过滤出大于等于18的元素
print('f operated:', id(f))             # 打印修改后元组的地址
print('f is a', f)                      # 打印结果显示 f 是个 filter对象
print(list(f))                          # 把 f 转成列表打印出来

--------------------------------------------------

# Output:
f source:   2379695132608
f operated: 2379696090368
f is a <filter object at 0x0000022A10F0AD00>
[23, 28, 19, 30]
```
同样，filter 也可以对元组操作，简单的过滤规则也可以用列表生成式代替

```python
# 用列表生成式实现简单过滤
f = [12, 23, 28, 19, 16, 30]
print([x for x in f if x >= 18])

--------------------------------------------------

# Output:
[23, 28, 19, 30]
```

## reduce
reduce 不是内置函数，必须导入
> `from functools import reduce`
> `reduce(function, sequence, initial=_initial_missing)`

参数：reduce 第一个参数接收一个带两个形参的函数，第二个参数接收一个序列
作用：把函数作用在一个序列上，函数必须接收两个参数，reduce 把结果继续和序列的下一个元素做累计运算
返回：一个计算结果

![reduce](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/No-Feature-reduce.png)

听起来有点绕，用示例解释

```python
from functools import reduce

def foo(x, y):
    return x + y

scores = [100, 89, 76, 87]
r = reduce(foo, scores)
print(r)

--------------------------------------------------

# Output:
352
```
在 reduce 中会一次次调用scores中的元素放到foo函数中进行计算
第一轮：x = 100, y = 89, result = 189
第二轮：x = result, y = 76, result = 265 (即 189 + 76)
第三轮：x = result, y = 87, result = 352 (即 265 + 87)
最后把 result 也就是 352 返回出去


上面的例子中的序列还是比较简单的，如果遇到复杂的数据对象，则不能这么简单处理

```python
from functools import reduce

dic = [
    {'name': 'Alice', 'age': 18, 'score': 78},
    {'name': 'Boii', 'age': 17, 'score': 89},
    {'name': 'Candy', 'age': 19, 'score': 65},
    {'name': 'Dannis', 'age': 22, 'score': 99}
]


def foo(x, y):
    return x['score'] + y['score']    # 第二轮就会出错


r = reduce(foo, dic)
print(r)

--------------------------------------------------

# Output:
Traceback (most recent call last):
  File "d:/---Programming---/Python/Project/练习/mo2.py", line 58, in <module>
    r = reduce(foo, dic)
  File "d:/---Programming---/Python/Project/练习/mo2.py", line 55, in foo
    return x + y['score']
TypeError: unsupported operand type(s) for +: 'dict' and 'int'
```
哦嚯，报错。
报错信息为：unsupported operand type(s) for +: 'dict' and 'int'
不支持 字典对象和整型做 相加操作

既然说相加，看来问题出在foo函数身上
以上面的方式再进行一次推导
第一轮：
x = {'name': 'Alice', 'age': 18, 'score': 78}, y = {'name': 'Boii', 'age': 17, 'score': 89}, 
则 x['score'] = 78, y['score'] = 89, result = 167
第二轮：
x = result, y = {'name': 'Candy', 'age': 19, 'score': 65}, 
则 x['score'] =...     **问题就出在这**

第一次计算之后result 给了x，x就成了整型，当然取不出 score

解决方法，改为 `return x + y['score']`

```python
from functools import reduce

dic = [
    {'name': 'Alice', 'age': 18, 'score': 78},
    {'name': 'Boii', 'age': 17, 'score': 89},
    {'name': 'Candy', 'age': 19, 'score': 65},
    {'name': 'Dannis', 'age': 22, 'score': 99}
]


def foo(x, y):
    return x + y['score']    # 一个字典 + 一个整型，依然报错


r = reduce(foo, dic)
print(r)

--------------------------------------------------

# Output:
Traceback (most recent call last):
  File "d:/---Programming---/Python/Project/练习/mo2.py", line 58, in <module>
    r = reduce(foo, dic)
  File "d:/---Programming---/Python/Project/练习/mo2.py", line 55, in foo
    return x + y['score']
TypeError: unsupported operand type(s) for +: 'dict' and 'int'
```
依然报错，这回问题出在第一轮
x = {'name': 'Alice', 'age': 18, 'score': 78}，y['score'] = 89
一个是字典一个是整型

**解决方法，设置初始值**

```python
from functools import reduce

dic = [
    {'name': 'Alice', 'age': 18, 'score': 78},
    {'name': 'Boii', 'age': 17, 'score': 89},
    {'name': 'Candy', 'age': 19, 'score': 65},
    {'name': 'Dannis', 'age': 22, 'score': 99}
]


def foo(x, y):
    return x + y['score']


r = reduce(foo, dic, 0)
print(r)

--------------------------------------------------

# Output:
331
```
第一轮：
x = 0, y = {'name': 'Alice', 'age': 18, 'score': 78}, 则 y['score'] = 78, result = 78
第二轮：
x = result, y ={'name': 'Boii', 'age': 17, 'score': 89}, 则 y['score'] = 89, result = 167
第三轮：
x = result, y ={'name': 'Candy', 'age': 19, 'score': 65}, 则 y['score'] = 65, result = 232
第四轮：
x = result, y ={'name': 'Dannis', 'age': 22, 'score': 99}, 则 y['score'] = 99, result = 331
最后把 331 返回出去

