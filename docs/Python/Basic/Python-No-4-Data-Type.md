---
title: Python【No-4】数据类型
tags: Python
categories:
  - Python
  - 基础
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 39690
date: 2020-07-05 21:41:48
---

基础知识：数据类型

<!--more-->

## 常见内置类型
!!! note
    - 内置类型：
        - None（全局只有一个）
    - 数值类型：int、float、complax（复数）、bool
    - 迭代类型
        - 序列类型：list、bytes、range、tuple、string、array
        - 映射类型：dict
        - 集合类型：set、frozenset
        - 上下文管理类型：with

    可变类型：list、set、dict

    不可变类型：int、float、string、tuple

## 基本数据类型
### 整型
可以是任意大小的整数

与数学上的表示方法一样 如：1，100，-800，0

可以用十六进制表示法 如：0xFF00, 0xab54f

#### 常用基本运算

!!! note
    加（+）    减（-）    乘（\*）    除（/）    模（%）

```python
>>> 3 + 2
5
>>> 3 - 2
1
>>> 3 * 2
6
>>> 3 / 2
1.5
>>> 2 + 3 * 4
14
>>> (2 + 3) * 4
20
>>> 17 % 3
2
```

!!! note
    除法运算 （/）永远返回浮点型

```python
>>> 9 / 3
3.0
```

!!! note
    乘方（**）    截断除法、整数除（//）

```python
>>> 3 ** 2
9
>>> 3 ** 3
27
>>> 17 // 3
5
```

!!! note
    包含多种混合类型运算数的运算会把整数转换为浮点数

```python
>>> 4 * 3.75 - 1
14.0
```



### 浮点数
即小数 如：1.2，524.33，-9.11

如果是很大或很小的浮点数，必须用科学计数法表示，用`e+指数`代替底数 `10^指数`，如：

!!! example
    1.23×10^9^ 就是 1.23e9 或 12.3e8，  0.000012 就是 1.2e-5

    浮点数运算可能会有四舍五入的误差

### 字符串

```python
str1 = 'OK'        # OK
str2 = "OK"        # OK
str3 = "I'm ok."   # I'm ok.
str4 = 'I\'m ok.'  # I'm ok.
str5 = "I\"m ok."  # I"m ok.
str6 = 'I"m ok.'   # I"m ok.
```

```python
str7 = '''这是一个段落，所以可以直接换行，不需反斜杠来声明语句未结束
    直到遇到下一个三单引号，才认为结束
但是换行会跟着换行，空格会跟着空格'''

"""
str7输出为：

这是一个段落，所以可以直接换行，不需反斜杠来声明语句未结束
    直到遇到下一个三单引号，才认为结束
但是换行会跟着换行，空格会跟着空格
"""
```

```python
\n 换行    \t 缩进

# 使用转义字符
str8 = 'ab\ncd'

'''
str8输出为：
ab
cd
'''


r'字符串' 表示不转义

# 不转义
str9 = r'ab\ncd'    # ab\ncd
```
转义字符

| 转义字符 |    描述    |
|:--------:|:---------:|
|    \     |   续行符   |
|    \\    |   反斜杠   |
|    \'    |   单引号   |
|    \"    |   双引号   |
|    \a    |    响铃    |
|    \b    |    退格    |
|    \e    |    转义    |
|   \000   |     空     |
|    \n    |    换行    |
|    \v    | 纵向制表符 |
|    \t    | 横向制表符 |
|    \r    |    回车    |
|    \f    |    换页    |
|    \o    |   八进制   |
|    \x    |  十六进制  |

### 布尔值
> `True`   `False`

可以用 `and` , `or` , `not` 运算

### 空值

空值是一个特殊的值，用 `None` 表示。`None` 不等于 `0`，`0` 是有意义的.
全局只有一个None


## 复合数据类型
### 列表 List

!!! note
    一种`有序的、可变的` 元素集合
    
    用 `[ ]` 标识

    可随机添加和删除其中的元素
    
    **可理解为可变的数组**
    
    是一种复合数据类型
    
    List 中的元素可以不同类型
    
    区别于元组Tuple：List 中的元素可变

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> print(name)
['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
```
#### List索引
```python
|  P  |  y  |  t  |  h  |  o  |  n  |
   0     1     2     3     4     5            # 从左往右，下标从0开始
  -6    -5    -4    -3    -2    -1            # 从右往左，下标从-1开始
```


#### List的增删改查
##### 初始化

`#!py listName = [element1, element2, element3, ...]`

##### 增
###### 追加到末尾 append

`#!python listName.append(value)`

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> name.append('Fit')
>>> name
['Alice', 'Boii', 'Chen', 'Dannie', 'Eva', 'Fit']
```

###### 插入到指定位置 insert
`#!py listName.insert(index, value)`

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> name.insert(3, 'Fit')
>>> name
['Alice', 'Boii', 'Chen', 'Fit', 'Dannie', 'Eva']
```

##### 删
###### 删除末尾元素 pop
`#!py listName.pop()`

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> 
>>> name.pop()
'Eva'
>>> name
['Alice', 'Boii', 'Chen', 'Dannie']
```

###### 删除指定位置元素 pop
`#!py listName.pop(index)` 或 `#!py del listName[index]`

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> name.pop(3)
'Dannie'
>>> name
['Alice', 'Boii', 'Chen', 'Eva']
>>> del name[2]
['Alice', 'Boii', 'Eva']
```

##### 改
修改某个元素，直接给该元素赋新值即可。`#!py listName[index] = value`

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> 
>>> name[2] = 'Cai'
>>> name
['Alice', 'Boii', 'Cai', 'Dannie', 'Eva']
```


##### 查
使用下标来访问List中的元素。 `#!py listName[index]`

```python
name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']

print(name[0])    # 'Alice' 访问List第一个元素
print(name[1])    # 'Boii'  访问List第二个元素
print(name[-1])   # 'Eva'   访问List最后一个元素
```

#### 获得长度 len()
`#!py len(List名)`

```python
>>> name = ['Alice', 'Boii', 'Chen', 'Dannie', 'Eva']
>>> len(name)
5
```

#### List中元素可以不同类型

```python
>>> L = ['Boii', 23, True, 'https://tcp404.com']
>>> L
['Boii', 23, True, 'https://tcp404.com']
```

#### List嵌套List

类似于多维数组的概念

```python
>>> l_main = ['Boii', 23, ['https://', 'tcp404', '.com'], 443]
>>> len(l_main)
4
>>> len(l_main[2])
3
```

等价于

```python
>>> l_sub =  ['https://', 'tcp404', '.com']
>>> l_main = ['Boii', 23, l_sub, 443]

#获取 tcp404可以用一下方式

>>> l_main[2][1]
'tcp404'
```
#### 空List

```python
>>> L = []
>>> len(L)
0
>>> li = list()
>>> len(li)
0
```

### 元组 Tuple
!!! note 
    一种 `有序的、不可变的` 元素集合

    用 `( )` 标识

    Tuple中的元素一旦初始化就不可变

    **可理解为不可变的数组**

    是一种复合数据类型

    Tuple中的元素可以不同类型

    区别于List：Tuple 中的元素 `不可变`

#### 初始化
`#!py tupleName = (elem1, elem2, elem3, ...)`

#### Tuple索引
```python
|  P  |  y  |  t  |  h  |  o  |  n  |
   0     1     2     3     4     5            # 从左往右，下标从0开始
  -6    -5    -4    -3    -2    -1            # 从右往左，下标从-1开始
```

#### Tuple的增删改查
Tuple不可变，所以不可以 增加、删除、修改，只能查询

查询与List相同

##### 查
`#!py tupleName[index]`

```python
>>> T = (1, 2, 'Boii')
>>> T[2]
'Boii'
>>> T[0]
1
>>> T
(1, 2, 'Boii')
```

#### 空Tuple ()
```python
>>> tr = ()
>>> tr
()
```

#### 定义单元素的Tuple (element，)

为避免与数学的括号混淆，定义一个元素的Tuple时，要在元素后加上逗号

```python
>>> tr = ('Boii',)
>>> tr
('Boii',)
```

且在python解释中：

- tr = (1) 会被认为是 tr = 1, tr 就变成一个普通的整型变量
- tr = ('Boii') 会被认为是 tr = 'Boii', tr 就变成一个普通的字符串变量

```python
>>> tr = (1)        # 等价于 tr = 1
>>> tr
1
>>> tr = ('Boii')   # 等价于 tr = 'Boii'
>>> tr
'Boii'
>>> 
>>> 
>>> tr = ('Boii',)  # √ 正确定义单元素Tuple
>>> tr
('Boii',)
```
#### Tuple不可变的意义
因为tuple不可变，所以代码更安全。如果可能，能用 tuple 代替 list 就尽量用 tuple。

#### ！！！不可变Tuple中的可变元素

Tuple的不可变指的是Tuple指向的元素不可变

如果元素中有 `可变的List` ，那么 Tuple 依然可以修改

```python
>>> t = ('a', 'b', ['A', 'B'])
>>> t[2][0] = 'X'
>>> t[2][1] = 'Y'
>>> t
('a', 'b', ['X', 'Y'])
```

定义 Tuple 时

![1](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/4-1.png)


修改 Tuple 的元素后

![2](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/4-2.png)


### 字典 Dict

!!! note
    一种 `无序的、可变的` 键-值对集合

    用 `{ }` 标识

    键key 必须是不可变对象 （如字符串、整数。而list，tuple这些不可以作为key）

    键key 不可以重复

    键key 可以不同类型，但不建议

    键key 可以是变量，但是这个变量必须指向字符串、整数这类不可变对象

    Dictionary VS List

    1. Dict 查找和插入的速度极快，不会随着key的增加而变慢；

    2. Dict 需要占用大量的内存，内存浪费多。

    3. List 查找和插入的时间随着元素的增加而增加；

    4. List 占用空间小，浪费内存很少。


#### 初始化
`#!py dictName = { key1 : value1, key2 : value2, key3 : value3, ... }`

```python
d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# 键可以不同类型
dd = {'a':'Alice', 'b':'Boii', 18:'Kk'}
```


#### Dict索引
Dict的索引就是 键key。

#### Dict的增删改查和判断
##### 增
`#!py dictName[key] = value`

```python
>>> d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
>>> d['AC'] = 'diu'
>>> d
{'a': 1, 'b': 2, 'c': 3, 'd': 4, 'AC':'diu'}

# 使用变量作为key
>>> d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
>>> a = 'x'
>>> d[a] = 'what'
>>> d
{'a': 1, 'b': 2, 'c': 3, 'd': 4, 'x':'what'}
```

##### 删 pop、del
`#!py dictName.pop(key)` 或 `#!py del dictName[key]`

```python
>>> d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
>>> d.pop('a')
{'b': 2, 'c': 3, 'd': 4}

>>> del d['c']
{'b': 2, 'd': 4}
```

##### 改
`#!py dictName[key] = newValue`

因为key不能重复，所以如果key不存在，会变成添加，如果key存在，newValue会覆盖oldValue

```python
>>> d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
>>> d['a'] = 25
>>> d
{'a': 25, 'b': 2, 'c': 3, 'd': 4}
>>> d['e'] = 15
>>> d
{'a': 25, 'b': 2, 'c': 3, 'd': 4, 'e': 15}
```


##### 查 get
`#!py dictName.get(key)`

如果key存在，返回对应的value

如果key不存在，返回None

```python
d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
d.get('a')       # 1
d.get('z')       # None
```


`#!py dictName.get(key, return)`

如果key存在，返回对应的value

如果key不存在，返回指定的返回值return，return可以是整型、字符串，甚至是List等

```python
>>> d = {'a': 1, 'b': 2, 'c': 3, 'd': 4}
>>> 
>>> d.get('z', 0)
0
>>> d.get('z', 'No This Key')
'No This Key'
>>> d.get('z', [0, 0])
[0, 0]
>>> d.get('z', {1:'a', 2:'b'})
{1:'a', 2:'b'}
```

##### 判断 in、not in
`#!py key in dictName`

```python
>>> d = {'a': 25, 'b': 2, 'c': 3, 'd': 4}
>>> 'b' in d
True
>>> 'z' in d
False
```

### 集合 Set
!!! note 
    一种 `无序的、不重复的、可变的` 的元素的集合

    用 `set([ ])` 或 `{key, key, ...}` 标识

    类似数学概念中的集合

    可以通过 Dict 来理解：Set 是一种不存储 value 的 Dict（因为key不能重复）

    是一种复合数据类型

    Set 中的元素可以不同类型

#### Set索引
Set 是无序的，所以没有索引，只有元素，或者说只有key

#### 初始化
`#!py setName = set(key_list)` 或 `#!py setName = {key1, key2, key3, ...}`

```python
>>> s1 = set([1, 2, 3, 'a', (32, 'a', False), 55])
>>> s1
{1, 2, 3, 55, (32, 'a', False), 'a'}

>>> s2 = set('abadfgaae')
>>> s2
{'g', 'b', 'e', 'f', 'd', 'a'}

>>> s3 = {1,2,3,4,4,4,5,5}
>>> s3
{1, 2, 3, 4, 5}
```

#### Set的增删改查
##### 增 add、update
`#!py setName.add(key)` 只能添加基本数据类型

`#!py setName.update(key)` 可以添加基本数据类型和复合数据类型

```python
>>> s3 = set([1,2,3,4,4,4,5,5])

>>> s3.add(7)
>>> s3
{1, 2, 3, 4, 5, 7}

>>> s3.update(['a', 'b'])
>>> s3
{1, 2, 3, 4, 5, 7, 'a', 'b'}
```

##### 删除 remove、discard、pop
`#!py setName.remove(key)` 删除指定元素，如果元素不存在会存在错误

`#!py setName.discard(key)` 删除指定元素，如果元素不存在不会存在错误

`#!py setName.pop()` 随机删除一个元素

```python
>>> s = set([1, 2, 3, 4, 7, 4, 15, 4, 5, 5])
>>> s
{1, 2, 3, 4, 5, 7, 15}

>>> s.remove(5)
>>> s
{1, 2, 3, 4, 7, 15}
>>> s.remove(20)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 20

>>> s.discard(15)
>>> s
{1, 2, 3, 4, 7}
>>> s.discard(20)
>>>

>>> s.pop()
4
>>> s
{2, 3, 1, 7}
>>> s.pop()
3
>>> s
{2, 1, 7}
>>> s.pop()
2
>>> s
{1, 7}
```

##### 改 remove + add
Set 中没有改的办法，因为集合是无序的，改一个元素没有意义

只能删除要改的key，然后添加新的key

`#!py  setName.remove(key)` 或 `#!py setName.add(key)`


##### 查


## 强制类型转换

| 函数                  | 描述                                                |
|:----------------------|:--------------------------------------------------|
| int(x [,base])        | 将x转换为一个整数                                   |
| long(x [,base] )      | 将x转换为一个长整数                                 |
| float(x)              | 将x转换到一个浮点数                                 |
| complex(real [,imag]) | 创建一个复数                                        |
| str(x)                | 将对象 x 转换为字符串                               |
| repr(x)               | 将对象 x 转换为表达式字符串                         |
| eval(str)             | 用来计算在字符串中的有效Python表达式,并返回一个对象 |
| tuple(s)              | 将序列 s 转换为一个Tuple                            |
| list(s)               | 将序列 s 转换为一个List                             |
| set(s)                | 转换为可变集合                                      |
| dict(d)               | 创建一个Dict。d 必须是一个序列 (key,value)Tuple。     |
| frozenset(s)          | 转换为不可变集合                                    |
| chr(x)                | 将一个整数转换为一个字符                            |
| unichr(x)             | 将一个整数转换为Unicode字符                         |
| ord(x)                | 将一个字符转换为它的ASCII十进制整数值               |
| hex(x)                | 将一个整数转换为一个十六进制字符串                  |
| oct(x)                | 将一个整数转换为一个八进制字符串                    |


```python
 
数字
    整数 -1 0 1
    浮点 -0.1 0.0 1.0
    二进制 0b11        结果 3
    八进制 0o77        结果 63
    16进制 0xFF        结果 255
 
字符串 <class 'str'>
    纯字符串 'str' "str" '''str''' """str"""
    字符串数字(二进制 0b) '0b0'     转成字符 str(0b10) 结果 '2'     ## 可以前置补零str(0b00000010)
    字符串数字(八进制 0o) '0o0'     转换字符 str(0o77) 结果 '63'    ## 可以前置补零str(0o0077)
    字符串数字(十进制)    '0'       转换字符 str(100)  结果 '100'   ## 不能前置补零
    字符串数字(16进制 0x) '0x0'     转换字符 str(0xFF) 结果 '255'   ## 可以前置补零str(0x00FF)
二进制 <class 'bytes'>
    二进制字节表示 b''    # ASCII 字符 0-9 a-z A-Z 等
```

### 数字 转 字符串
```python
##  255(10进制)  0b11(2进制)  0xFF(16进制)
## (10进制数)
>>> bin(255)            '0b11111111'
>>> oct(255)            '0o377'
>>> hex(255)            '0xff'
## (2进制数)
>>> bin(0b11)           '0b11'
>>> hex(0xFF)           '0xff'
## (16进制数)
>>> bin(0xFF)           '0b11111111'
>>> hex(0xFF)           '0xff'
```


### 字符串 转 数字（十进制数）
```python
##  '123'(以10进制解析)  '10'(以2进制解析)  'a'(以16进制解析)
## (10进制表示的字符串)
>>> int('123')          123         ## 十进制字符转十进制数字
>>> int('123',10)       123         ## 默认是十进制
## (二进制表示的字符串)
>>> int('100',2)        4           ## 二进制的 100 等于 十进制的 4（可以不加前置 0b）
>>> int('0b100',2)      4           ## 二进制的 100 等于 十进制的 4
>>> int('0b0100',2)     4           ## 可以前置补零
## (16进制表示的字符串)
>>> int('a',16)         10          ## 16进制的 a 等于 十进制的 10（可以不加前置 0x）
>>> int('0xa',16)       10          ## 16进制的 a 等于 十进制的 10
>>> int('0x0a',16)      10          ## 16进制的 a 等于 十进制的 10（可以前置补零）
>>> int('10',16)        16          ## 16进制的10 等于 十进制的 16（可以不加前置 0x）
>>> int('0x10',16)      16          ## 16进制的10 等于 十进制的 16
>>> int('0x0010',16)    16          ## 16进制的10 等于 十进制的 16（可以前置补零）
```

### 数字 转 数字
```python
##  0b11  0xFF
## 十进制 转 十进制
>>> int(255)            255         # 无意义操作
>>> 255                 255         # 无意义操作
## 二进制 转 十进制
>>> int(0b11)           3           # 可加前置零 int(0b0011)
>>> 0b11111111          255         # 等价
## 16进制 转 十进制
>>> int(0xFF)           255
>>> 0xff                255         # 等价 且 忽略大小写
>>> 0xFF                255         # 等价 且 忽略大小写
## 十进制 转 二进制（使用 数字 转 字节码/字符）
255 等价 0b11111111
## 十进制 转 16进制（使用 数字 转 字节码/字符）
255 等价 0xff
```

### 字符串 转 字节码
```python
>>> bytes('abc','utf-8')              b'abc'
>>> bytes('编程','utf-8')             b'\xe7\xbc\x96\xe7\xa8\x8b'
>>> bytes('Python3编程','utf-8')      b'Python3\xe7\xbc\x96\xe7\xa8\x8b'
>>> 'Python3编程'.encode('UTF-8')     b'Python3\xe7\xbc\x96\xe7\xa8\x8b'
>>> S = 'Python3编程'                 'Python3编程'
>>> B = bytes(S,'utf-8')              b'Python3\xe7\xbc\x96\xe7\xa8\x8b'
>>> FMT = str(len(B)) + 's'           '13s'
>>> struct.pack(FMT,B)                b'Python3\xe7\xbc\x96\xe7\xa8\x8b'
## 以16进制数字写的字符串，直接转成一样的字节码（2个16进制字符才是一个字节）
>>> bytes.fromhex('01')               b'\x01'                       # 单字节
>>> bytes.fromhex('0001')             b'\x00\x01'                   # 双字节
>>> bytes.fromhex('aabbccddeeff')     b'\xaa\xbb\xcc\xdd\xee\xff'   # 多字节
```

### 字节码 转 字符串
```python
##  取出16进制表示的内容
>>> b'abc'.decode('UTF-8')                                  'abc'
>>> b'Python3\xe7\xbc\x96\xe7\xa8\x8b'.decode('UTF-8')      'Python3编程'
>>> b'\xaa\xbb\xcc\xdd\xee\xff'.hex()                       'aabbccddeeff'
>>> b'0'.hex()                                              '30'    ## 字符0在ASCII码上的数字（数字是16进制表示）== 48（十进制）
>>> b'1'.hex()                                              '31'
>>> b'z'.hex()                                              '7a'
```
### 数字 转 字节码（是二进制，用16进制显示） 

```python
# 10进制数 转 字节码
import struct
>>> struct.pack('B',0)          b'\x00'
>>> struct.pack('B',1)          b'\x01'
>>> struct.pack('B',101)        b'e'                    ## 101 对应 16进制的 0x65（此处返回值是显示为当前整数 101 对应的 ASCII字符 e）
>>> struct.pack('B',255)        b'\xff'                 # 无符号最大单字符可以表示的数字
>>> struct.pack('>i',255)       b'\x00\x00\x00\xff'     # 4字节大端表示的数字
>>> struct.pack('<i',255)       b'\xff\x00\x00\x00'     # 4字节小端表示的数字
# 2进制数 转 字节码
import struct
>>> struct.pack('B',0b11111111)     b'\xff'
>>> struct.pack('>i',0b111)         b'\x00\x00\x00\x07'     # 0b111 等于 7（10进制）
>>> struct.pack('>i',0b1111)        b'\x00\x00\x00\x0f'     # 0b1111 等于 15（10进制）
>>> struct.pack('>i',0b11111)       b'\x00\x00\x00\x1f'     # 0b11111 等于 31（10进制）
# 16进制数 转 字节码
import struct
>>> struct.pack('B',0xff)           b'\xff'
>>> struct.pack('>i',0xfff)         b'\x00\x00\x0f\xff'
```

### 字节码 转 数字
```python
import struct          16进制表现                10进制等值
>>> struct.unpack('B', b'\xff')                      (255,)          # 单字节
>>> struct.unpack('>i', b'\x00\x00\x00\xff')         (255,)          # 4字节，大端模式
>>> struct.unpack('<i', b'\x00\x00\x00\xff')         (-16777216,)    # 4字节，小端模式
## 手动 转换
字节码 -> 字符串
>>> B = b'\xe9'
>>> S = B.hex()
>>> S                                               # 值 'e9'
字符串（16进制格式）-> 数字（10进制）
>>> int(S,16)                                       # 值 233
```

### ASCII 字符 和 数字

- 字节    b'\x05'
- 字符串   '\x05'
- 将一个整数 (0-1114111) 转换为 一个字符（整数对应的 ASCII 字符）
- ValueError: chr() arg not in range(0x110000)

```python

>>> chr(0)                  '\x00'
>>> chr(1)                  '\x01'
>>> chr(97)                 'a'
>>> chr(1114111)            '\U0010ffff'
>>> len(chr(101))           1  # 长度为 1个字符
>>> len(chr(1114111))       1  # 长度为 1个字符
```

### 将一个 ASCII字符 转换为 一个整数
```python
>>> ord('\x00')             0
>>> ord('\x01')             1
>>> ord('a')                97
>>> ord('0')                48
>>> ord('1')                49
>>> ord('A')                65
>>> ord('Z')                90
>>> ord('\U0010ffff')       1114111
```
### ASCII 字符 和 bin(字节)
```python
from binascii import b2a_hex, a2b_hex
>>> a2b_hex('ab')
b'\xab'

>>> b2a_hex(b'ab')
b'6162'
>>> a2b_hex(b'6162')
b'ab'
```

## 总结 (摘自 [羋虹光](https://www.jianshu.com/p/b3157c9751d0))
### int 类型解析

较小的整数会很频繁的被使用，所以python将这些对象放置到了一个池子中，每次需要这些对象的时候就到池子中获取这个值，避免多次的重复创建对象引起的许多不必要的开销。这个池子内的数字范围是[-5, 257), 所以都是从池子里面取值，自然id不变。

### float类型解析

对于float类型的使用自然没有int那么频繁，并且float类型也不好定义哪些常用，也就没有池子给到这个类型，所以每次重新创建即可。

### tuple类型解析

对于tuple类型，与float类型的思维相似，所以也是每次重新创建。

### string类型解析

单词类型的str由于被重复使用的概率比较大，所以在python中为单词类型的str做了一个缓存，也就是说如果是单词类型的str， 会被存储到一个字典(dict)中，字典的内容是字符串为key， 地址为value。当有一个字符串需要创建，就先去访问这个字典，如果存在则返回字典中字符串的地址，如果不存在，则返回新创建的地址，并将这个字符串添加进入字典。这是字符串的intern机制。
