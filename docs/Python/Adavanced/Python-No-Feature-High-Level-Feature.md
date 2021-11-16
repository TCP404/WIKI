---
title: Python【Feature】高级特性
tags: Python
categories:
  - Python
  - 高级特性
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 44742
date: 2020-07-24 11:17:48
---

Python的高级特性1

<!--more-->

## 切片 Slice
> [startWith : end : step]
> 切片适用于 list、tuple、string
> startWith是包含，end则不包含
>
> 注意：
> 切片是从左往右切的
> 如果用正索引开始，就要用 正索引 或 逆索引 结束，例如 `s[1:5] 或 s[1:-3]`
>
> ！如果用逆索引开始，就要用 逆索引 结束（`s[-1:1]` 这样只会返回空字符串，**正确**应该是  `s[-5:-1]`）
> ！逆索引记得不要弄颠倒了。（`s[-1:-3]` 这样只会返回空字符串，**正确**应该是`s[-3:-1]`）

### 索引
```python
|  P  |  y  |  t  |  h  |  o  |  n  |
   0     1     2     3     4     5            # 从左往右，下标从0开始
  -6    -5    -4    -3    -2    -1            # 从右往左，下标从-1开始
```

```python
>>> s = 'Python'
>>> s[0]
'P'
>>> s[0:-1]     # 结束索引不被截取
'Pytho'
>>> s[:5]       # 起始索引可以省略不写，代表从第一位开始截取
'Pytho'
>>> s[0:]       # 结束索引可以省略不写，代表截取至最后一位
'Python'
>>> s[:]        # 等于复制了一整个
'Python'

>>> s[1:4]
'yth'
>>> s[1:-2]
'yth'
>>> s[-2:1]     # 逆索引开始，不能以正索引结束
''
>>> s[-5:-2]
'yth'
>>> s[-2:-5]    # 只能从左往右切
''
```

```python
>>> l = list(range(100))

>>> l[10:20]
[10, 11, 12, 13, 14, 15, 16, 17, 18, 19]

>>> l[10:20:3]
[10, 13, 16, 19]

>>> l[80::3]
[80, 83, 86, 89, 92, 95, 98]

>>> l[:10:3]
[0, 3, 6, 9]

>>> l[::10]
[0, 10, 20, 30, 40, 50, 60, 70, 80, 90]

>>> l[::-10]
[99, 89, 79, 69, 59, 49, 39, 29, 19, 9]
```

## 迭代 Iteration
### 判断是否迭代对象
> 可以用过 collections 的 Iterable 类型，用 `isinstance()`判断 `一个对象是否是可迭代对象`

```python
from collections import Iterable

str = 'abc'
l = [1, 2, 3]
t = (1, 2, 3)
d = {1: 1, 2: 2, 3: 3}
s = ({1, 2, 3})

print(isinstance(str, Iterable))    # True
print(isinstance(l, Iterable))      # True
print(isinstance(t, Iterable))      # True
print(isinstance(d, Iterable))      # True
print(isinstance(s, Iterable))      # True
print(isinstance(123, Iterable))    # False  整数字面量不是可迭代对象
```

### 迭代 List 列表类型数据

```python
s = 'abcdefg'
for value in s:
    print(value, end=' ')

# Output：
a b c d e f g
```

```python
# 可以通过 enumerate() 给string，list，tuple 等生成下标

s = 'abcdefg'
for i, value in enumerate(s):
    print(i, value)

# Output：
0 a
1 b
2 c
3 d
4 e
5 f
6 g
```

```python
l = ['a', True, (1, 2), 43]
for i, value in enumerate(l):
    print(i, value)

# Output:
0 a
1 True
2 (1, 2)
3 43
```

```python
# 同时引用两个变量
for x, y in [(11, 12), (21, 22), (31, 32)]:
    print(x, y)

# Output:
11 12
21 22
31 32

######################################################

for x, y, z in [(11, 12, 13), (21, 22, 23), (31, 32, 33)]:
    print(x, y, z)

# Output:
11 12 13
21 22 23
31 32 33
```

### 迭代 Dict 键-值对类型数据
> 默认情况下，Dict迭代的是key，如果要迭代value，可以用 `for value in d.values()`
> 如果要同时迭代key 和 value， 可以用 `for k, v in d.items()`

```python
d = {'a': 'Alice', 'b': 'Boii', 'c': 'Cai'}
for i in d:
    print(i)

# Output:
a
b
c

#############################################

d = {'a': 'Alice', 'b': 'Boii', 'c': 'Cai'}
for i in d.values():
    print(i)

# Output:
Alice
Boii
Cai

#############################################

d = {'a': 'Alice', 'b': 'Boii', 'c': 'Cai'}
for k, v in d.items():
    print(k, '-', v)

# Output:
a - Alice
b - Boii
c - Cai
```

## 列表生成式 List Comprehensions
### 列表生成式
> [ i-expression **for** i **in** Iterations ]
> [ i-expression **for** i **in** Iterations **if** expression ]
> [ i-expression **if** expression **else** expression **for** i **in** Iterations ]

```python
[ ··· for ··· in ··· ]
[ ··· for ··· in ··· if ··· ]
[ ··· if ··· else ··· for ··· in ··· ]
```
列表生成式是一种非常简洁的语法，可以大幅度的压缩代码
阅读方式：
- `[i-expression for i in Iterations]`
遍历迭代对象 Iterations，将每个元素 e 放到 i-expression中运算后，作为这个列表 list 的元素
- `[i-expression for i in Iterations if expression]`
遍历迭代对象 Iterations，将每个元素 e 放到 if 中的 expression中运算后，放到 i-expression 中运算，然后作为这个列表 list 的元素
- `[i-expression if expression else expression for i in Iterations]`
遍历迭代对象 Iterations，将每个元素 e 放到 if else 中的 expression中运算后，放到 i-expression 中运算，然后作为这个列表 list 的元素

for 前的 if else 可以看成三目运算符，这样比较好理解
1. 三目运算符：True时执行 **if** expression **else** Flase时执行
2. 对比一下：[ i-expression **if** expression **else** expression for i in Iterations ]
3. 代入一下：[ True时执行 **if** expression **else** Flase时执行 for i in Iterations ]

**一句话：原本有10个，for...if 后不一定有 10 个， if...else...for 以后有10个，不过可能不尽相同。**
**两句话：for 前必 else，for 后不else。**

> 注意：i-expression 的`计算变量`和 for 里的`临时变量`要相同

普通列表生成式
```python
# range(1, 11) 生成1~10 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
# for in 遍历这10个数
# 每次遍历都把当前元素拿到 i-expression 即 x * x 做相乘计算
# 然后作为列表的元素
# 注意 for 的临时变量是 x，i-expression 的变量也是 x，这两个要相同

# 这里是计算出1~10每个数的平方
>>> [x * x for x in range(1, 11)]
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]


# 把一个 List 中所有字符串变小写
>>> L = ['Hello', 'World', 'IBM', 'Apple']
>>> [s.lower() for s in L]
['hello', 'world', 'ibm', 'apple']
```

for 里带 if 筛选的列表生成式
```python
# range(1, 11) 生成1~10 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
# for in 遍历这10个数
# 每次遍历都把当前元素拿到 if 里计算, 得到 [2, 4, 6, 8, 10]
# 然后放到 i-expression 即 x * x 做相乘计算得到 [4, 16, 36, 64, 100]
# 最后作为列表的元素

# 这里是筛选出偶数，然后计算平方

>>> [x * x for x in range(1, 11) if x % 2 == 0]
[4, 16, 36, 64, 100]
```

双循环的列表生成式
```python
# 还可以使用双循环，生成全排列

>>> [m + n for m in 'ABC' for n in 'XYZ']
['AX', 'AY', 'AZ', 'BX', 'BY', 'BZ', 'CX', 'CY', 'CZ']
```

多变量的列表生成式
```python
# 一个循环多个变量也是可以的
>>> d = {'x': 'A', 'y': 'B', 'z': 'C' }
>>> [k + '-' + v for k, v in d.items()]
['y-B', 'x-A', 'z-C']
```

for 前带 if 的列表生成式
```python
# # range(1, 11) 生成1~10 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
# for in 遍历这10个数
# 每次遍历都把当前元素拿到 for 前的 if else三目运算符里计算
# x if x % 2 == 0 else -x ,偶数正常输出, 奇数变负数，得到 [-1, 2, -3, 4, -5, 6, -7, 8, -9, 10]
# 最后作为列表的元素
>>> [x if x % 2 == 0 else -x for x in range(1, 11)]
[-1, 2, -3, 4, -5, 6, -7, 8, -9, 10]

# 再稍微改动下
# 把 x if x % 2 == 0 else x + 10 看作三目运算符就很好理解
>>> [x if x % 2 == 0 else x + 10 for x in range(1, 11)]
[11, 2, 13, 4, 15, 6, 17, 8, 19, 10]
```

### 其他生成式
除了列表生成式，还有`字典生成式`、`元组生成式`，其实现都是一样的
```python
{k: v for k, v in d.items()}
{k: v for k, v in d.items() if condition}
{k: v if condition else k: -v for k, v in d.items() }
```

元组生成式 生成一个生成器对象，下面将会讲到。


### 错误示范

```python
# for 后 if 加 else
>>> [x for x in range(1, 11) if x % 2 == 0 else 0]
  File "<stdin>", line 1
    [x for x in range(1, 11) if x % 2 == 0 else 0]
                                              ^
SyntaxError: invalid syntax

# for 前 if 不加 else
>>> [x if x % 2 == 0 for x in range(1, 11)]
  File "<stdin>", line 1
    [x if x % 2 == 0 for x in range(1, 11)]
                     ^
SyntaxError: invalid syntax
```

### 总结
- `for...in...` 最先，`if 或 if else` 随后，`i-expression` 最后
- `for 前 if` 必加 else，`for 后 if` 不加 else
- `for 前 if` 是对 for的输出进行处理，`for 后 if` 是对for的输出进行控制过滤

## 生成器 Generator

### 普通形式
元组生成式 生成一个生成器对象，通过for或者next遍历,遍历后，原生成器对象就不存在了

> ( i-expression **for** i **in** Iterations )
> ( i-expression **for** i **in** Iterations **if** expression )
> ( i-expression **if** expression **else** expression **for** i **in** Iterations )

```python
(··· for ··· in ··· )
(··· for ··· in ··· if ··· )
(··· if ··· else ··· for ··· in ··· )
```

生成器，是一种一边循环一边计算的机制
比如现在需要一个`1到100W的平方`的列表，用列表生成式表示为`[ x * x for x in range(1000000)]`
但如果只需要经常访问前面几个元素，则浪费了很大的空间
而**生成器是保存了一种算法，它不会直接创建100W个数，而是等到调用的时候通过计算获得**
也就是说：列表生成式是一种缩写，生成器是一种算法

> 生成器 vs 列表生成式：
> 列表生成式 是使用 `[]`，生成器 是使用 `()`
> 列表生成式是一个列表，可以直接列出所有元素；生成器要用`next()`或`遍历`来列出所有元素

```python
>>> L = [ x * x for x in range(10)]
>>> L
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

>>> G = ( x * x for x in range(10) )
>>> G
<generator object <genexpr> at 0x00000204807D8D60>
>>> next(G)
0
>>> next(G)
1
>>> next(G)
4
>>> next(G)
9
>>> next(G)
16
>>> next(G)
25
>>> next(G)
36
>>> next(G)
49
>>> next(G)
64
>>> next(G)
81
>>> next(G)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
```

**生成器是算法，是规律，是计算方式**，通过`next(G)`计算出G的下一个元素的值，直到最后一个元素
没有更多元素的时候，抛出`StopIteration`错误
但是这种不断调用`next()`的方法显然不科学，

所以一般都是用`for循环`

```python
G = ( x * x for x in range(10) )
for i in G:
    print(i, end=' ')

# Output:
0 1 4 9 16 25 36 49 64 81
```

### 函数形式
生成器的另一种实现方式是 `函数形式`

```python
def funcName():
    sth
    while True:
        yield sth
        sth
```
> 普通函数通过`return语句`返回
> 生成器通过`yield语句`返回，并在下次调用时从`yield语句`处继续
> `yield`可以看成`return`，都是返回一个值出去
> 只不过使用`yield`在下次调用时会从`yield`处继续。

#### 实例
例如打印杨辉三角
```
          1
         / \
        1   1
       / \ / \
      1   2   1
     / \ / \ / \
    1   3   3   1
   / \ / \ / \ / \
  1   4   6   4   1
 / \ / \ / \ / \ / \
1   5   10  10  5   1
```
期待输出是这样的
```
# [1]
# [1, 1]
# [1, 2, 1]
# [1, 3, 3, 1]
# [1, 4, 6, 4, 1]
# [1, 5, 10, 10, 5, 1]
# [1, 6, 15, 20, 15, 6, 1]
# [1, 7, 21, 35, 35, 21, 7, 1]
# [1, 8, 28, 56, 70, 56, 28, 8, 1]
# [1, 9, 36, 84, 126, 126, 84, 36, 9, 1]
```
实现思路：
先看杨辉三角的规律：头尾都是1，中间是上一层`第n个`+`第n+1个`，每一层都是list
而且第一层比较特殊，是 [1]
可以先把杨辉三角核心的算法搞定。

##### 核心算法
```python
L = [1]
L = [1] + [L[n] + L[n+1] for n in range(len(L)-1)] +[1]
```
`[1] + [L[n] + L[n+1] for n in range(len(L)-1)] +[1]` 就是核心算法了
把他拆开来看，可以发现这个表达式是由中间段加上头尾的 `[1]` 组成的

再看中间段 `[L[n] + L[n+1] for n in range(len(L)-1)]`
这是一个列表生成式
`range(len(L)-1)`会生成 0 1 2 ...

假设当前为第5层，准备生成 [1, 4, 6, 4, 1]
当前的 L 是 [1, 3, 3, 1]，那么`len(L)`则是4，
`range(len(L) - 1)`则等价于`range(3)`,会生成 0 1 2

`[L[n] + L[n+1] for n in range(len(L)-1)]`则等价于
`[L[0]+L[1], L[1]+L[2], L[2]+L[3]]` 则等价于
`[  1 + 3,     3 + 3,     3 + 1  ]`则等价于
`[4, 6, 4]` 正好就是中间段
再加上头尾两个 [1]： `[1, 4, 6, 4, 1]`

##### 做成生成器
因为第一层 特殊，所以用小括号列表生成式`()`做不出来，改用函数形式

```python
def triangles():
    L = [1]
    while True:
        yield L
        L = [1] + [L[x]+L[x+1] for x in range(len(L)-1)] + [1]
```

第一次迭代生成器triangles时，会执行 `L = [1]`，然后进入循环，遇到`yield L`把L返回出去

第二次迭代生成器triangles时，会从`yield L`处继续，
然后执行核心算法 `L = [1] + [L[x]+L[x+1] for x in range(len(L)-1)] + [1]`
之后继续循环，又遇到`yield L`，把更新过的 L 返回出去

接着第三次，第四次...

其实有点类似 C 语言中被`static`修饰的局部变量，函数调用完不会被销毁，程序结束才销毁

##### 遍历输出

```python
for n, t in enumerate(triangles()):
    results.append(t)
    n += 1
    if n == 10:
        break
```

生成器也是一个可迭代对象，所以可以用 `for...in...`来迭代输出
`enumerate()`赋予了对应的下标
这个生成器可以无限循环下去，可以获得无穷层，这里打印10层作为示范即可

最后可以得到期待输出：

```python
[1]
[1, 1]
[1, 2, 1]
[1, 3, 3, 1]
[1, 4, 6, 4, 1]
[1, 5, 10, 10, 5, 1]
[1, 6, 15, 20, 15, 6, 1]
[1, 7, 21, 35, 35, 21, 7, 1]
[1, 8, 28, 56, 70, 56, 28, 8, 1]
[1, 9, 36, 84, 126, 126, 84, 36, 9, 1]
```

##### 完整代码

```python
def triangles():
    L = [1]
    while True:
        yield L
        L = [1] + [L[x]+L[x+1] for x in range(len(L)-1)] + [1]


results = []
for n, t in enumerate(triangles()):
    results.append(t)
    n += 1
    if n == 10:
        break

for t in results:
    print(t)

if results == [
    [1],
    [1, 1],
    [1, 2, 1],
    [1, 3, 3, 1],
    [1, 4, 6, 4, 1],
    [1, 5, 10, 10, 5, 1],
    [1, 6, 15, 20, 15, 6, 1],
    [1, 7, 21, 35, 35, 21, 7, 1],
    [1, 8, 28, 56, 70, 56, 28, 8, 1],
    [1, 9, 36, 84, 126, 126, 84, 36, 9, 1]
]:
    print('测试通过!')
else:
    print('测试失败!')
```

### 总结
普通形式-生成器：把列表生成式的`[]`换成`()` 
函数形式-生成器：函数中带`yield`，下一次调用时会从`yield`处继续

## 迭代器 Iterator
> 可以被`next()`函数调用并不断返回下一个值的对象称为迭代**器**：`Iterator`。


> 可迭代**对象**：
> 可以直接作用于 for 循环的对象统称为可迭代**对象**`Iterable`
> 可迭代**对象**`Iterable`：
> 1. List、Tuple、Dict、Set、String；
> 2. 生成器Generator、带`yield`的Generator Function
> 
> 可以用`isinstance(obj, Iterable)`判断是否是可迭代**对象**

> 迭代**器**：
> 不但可以作用于 for 循环，还可以被`next()`函数调用并不断返回下一个值则称为迭代**器**：`Iterator`。
> 生成器Generator、带`yield`的Generator Function就是迭代**器**
> 可以用`isinstance(obj, Iterator)`判断是否是迭代**器**

所以：Generator、Generator Function 既是 `Iterator`，也是 `Iterable`
而 List、Tuple、Dict、Set、String 仅仅只是 `Iterable`，**但是**，使用`iter()`函数可以使他们变成迭代**器**`Iterator`

```python
>>> isinstance(iter([]), Iterator)
True
>>> isinstance(iter('Boii'), Iterator)
True
```

### 迭代器的本质
Python的`Iterator对象`表示的是一个数据流，`Iterator对象`可以被`next()函数`调用并不断返回下一个数据，直到没有数据时抛出`StopIteration`错误。
可以把这个数据流看做是一个有序序列，但我们却不能提前知道序列的长度
只能不断通过`next()函数`实现按需计算下一个数据，所以`Iterator`的计算是**惰性**的，只有在需要返回下一个数据时它才会计算

### 自定义可迭代类
只要类中实现了`__iter__()`方法，这个类的对象就是可迭代**对象**`Iterable`。
只要类中实现了`__iter__()`和`__next__()`方法，这个类就是个迭代**器**`Iteratorn`。

`__iter__()`方法必须返回一个迭代**器**。
那么代码就是这样子：

```python
# 1. 创建一个类
class Classmate:
    def __init__(self):
        self.names = list()

    def add(self, value):
        self.names.append(value)
    
    # 2. 实现__iter__方法，返回一个迭代器
    def __iter__(self):
        """只有实现了__iter__方法，才能可迭代"""
        return ClassmateIterator(self)  # __iter__ 必须返回一个迭代器对象


# 3. 创建一个迭代器
class ClassmateIterator:
    """实现了__iter__和__next__方法，才是一个迭代器"""
    def __init__(self, obj):
        self.obj = obj
        self.current_num = 0

    # 4. 迭代器需要实现 __iter__, __next__方法
    def __iter__(self):
        pass

    def __next__(self):
        """实现了__iter__和__next__方法，才是一个迭代器"""
        if self.current_num < len(self.obj.names):
            res = self.obj.names[self.current_num]
            self.current_num += 1
            return res
        else:
            raise StopIteration


if __name__ == '__main__':
    clsm = Classmate()
    clsm.add(123)
    clsm.add(456)
    clsm.add(789)

    # 通过循环检查classmate对象是否可迭代
    for i in clsm:
        print(i)

--------------------------------------------------

# Output:
123
456
789
```
总结起来就是：
创建一个类并实现`__iter__()`方法去返回一个迭代器
创建一个类并实现`__iter__()`和`__next__()`方法，使其成为迭代器。

这样看起来好像为了实现一个迭代器又生成了一个迭代器
既然我是想造一个迭代器，`__iter__()`方法有需要返回一个迭代器对象，那我返回自己不行吗？

可以！
```python
# 1. 创建一个类
class Classmate:
    def __init__(self):
        self.names = list()
        self.current_num = 0

    def add(self, value):
        self.names.append(value)

    # 2. 实现__iter__方法，返回一个迭代器
    def __iter__(self):
        """只有实现了__iter__方法，才能可迭代"""
        return self  # __iter__ 必须返回一个迭代器对象，这里就返回自己就行

    # 3. 实现__next__方法，这个方法会在外部使用next(Iterator)时调用。
    def __next__(self):
        """实现了__iter__和__next__方法，才是一个迭代器"""
        if self.current_num < len(self.names):
            res = self.names[self.current_num]
            self.current_num += 1
            return res
        else:
            raise StopIteration


if __name__ == '__main__':
    clsm = Classmate()
    clsm.add(123)
    clsm.add(456)
    clsm.add(789)

    # 通过循环检查classmate对象是否可迭代
    for i in clsm:
        print(i)

--------------------------------------------------

# Output:
123
456
789
```

### 总结
1. 可迭代对象和迭代器，通过函数`iter()`转换
2. 迭代器：数据流，变长，惰性的 
3. 继承关系：Iterable>Iterator>Generator(参考python doc)
4. Iterable：list、tuple、dict、set、str、Iterator、Generator等
5. Iterator转化成Iterator：Iterator=iter(Iterable)
6. 自定义迭代类时必须实现`__iter__()`和`__next__()`方法
7. `__iter__()`方法必须返回一个迭代器