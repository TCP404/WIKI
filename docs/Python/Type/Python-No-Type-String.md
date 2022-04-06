---
title: Python【Type】String
tags: Python
categories:
  - Python
  - 基础
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 32620
date: 2020-07-26 11:17:48
---

关于String的详细笔记

<!--more-->

```python
a = 'hello'
b = "good"
c = """ 哈哈哈 """
d = ''' 呵呵呵 '''

str1 = 'OK'                     # OK
str2 = "OK"                     # OK
str3 = '''I'm OK.'''            # I'm OK.
str4 = """哈哈哈"""              # 哈哈哈
str4 = "I'm ok."                # I'm ok.
str5 = 'I\'m ok.'               # I'm ok.
str6= "I\"m ok."               # I"m ok.
str7 = 'I"m ok.'                # I"m ok.
str8 = """I'm "Iron Man"."""    # I'm "Iron Man".
str9 = r'ab\ncd'                 # ab\ncd
str0 = R'ab\ncd'                 # ab\ncd
```

```python
str_1 = '''这是一个段落，所以可以直接换行，不需反斜杠来声明语句未结束
    直到遇到下一个三单引号，才认为结束
但是换行会跟着换行，空格会跟着空格'''

"""
str_1输出为：

这是一个段落，所以可以直接换行，不需反斜杠来声明语句未结束
    直到遇到下一个三单引号，才认为结束
但是换行会跟着换行，空格会跟着空格
"""
```

```python3
\n 换行    \t 缩进

# 使用转义字符
str_2 = 'ab\ncd'

'''
str_2输出为：
ab
cd
'''


r'字符串' 表示不转义

# 不转义
str_3 = r'ab\ncd'    # ab\ncd
```
转义字符

| 转义字符 |    描述    |
| :------: | :--------: |
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

## 下标
```python
 |  P  |  y  |  t  |  h  |  o  |  n  |
    0     1     2     3     4     5
   -6    -5    -4    -3    -2    -1
```

## 查找 find、index、rfind、rindex
> s.find(sub[, start[, end]) -> int
> s.rfind(sub[, start[, end]) -> int
> s.index(sub[, start[, end]) -> int
> s.rindex(sub[, start[, end]) -> int

没r找第一个，有r找最后一个
find找不到返回-1，index找不到报错

```python
s = 'abcdefghijkl abundance gg'
print(s.find('g'))     # 返回字符串中第一个 g 的下标：6
print(s.find('z'))     # 找不到则返回 -1
print(s.index('g'))    # 返回字符串中第一个 g 的下标：6
print(s.index('z'))    # 找不到则报错

print(s.rfind('g'))    # 返回字符串中最后一个 g 的下标：24
print(s.rfind('z'))    # 找不到则返回 -1
print(s.rindex('g'))   # 返回字符串中最后一个 g 的下标：24
print(s.rindex('z'))   # 找不到则报错
```

## 判断

| 方法名            | 描述                              |
| :---------------- | :-------------------------------- |
| startswith(value) | 判断是否以 value 开始             |
| endswith(value)   | 判断是否以 value 结束             |
| isalpha()         | 判断是否全是字母                  |
| isdigit()         | 判断是否全是数字                  |
| isalnum()         | 判断是否全是字母或数字或字母+数字 |
| isdecimal()       | 判断是否全是浮点数                |
| isnumeric()       | 判断是否全是数值                  |
| islower()         | 判断是否全小写                    |
| isupper()         | 判断是否全大写                    |
| isspace()         | 判断是否全空格或\t或\n或\r        |
| istitle()         | 判断是否所有单词都是首字母大写    |

### isdigit() VS isdecimal() VS isnumeric()

**isdigit()**
True: Unicode数字，byte数字（单字节），全角数字（双字节），罗马数字
False: 汉字数字
Error: 无

**isdecimal()**
True: Unicode数字，，全角数字（双字节）
False: 罗马数字，汉字数字
Error: byte数字（单字节）

**isnumeric()**
True: Unicode数字，全角数字（双字节），罗马数字，汉字数字
False: 无
Error: byte数字（单字节）

## 计数 count
> s.count(sub[, start[, end]) -> int

```python
s = 'abcdefghijkl abundance gg'
print(s.count('g'))        # 3. 字母 g 一共出现了3次
```

## 替换 replace
> s.replace(old, new[, count]) -> str
替换字符串中指定的内容，如果指定次数count， 则替换不会超过count次。

```python
s = 'abcdefghijkl abundance gg'
print(s.replace('g', '*'))        # abcdef*hijkl abundance **
print(s.replace('g', '*', 2))     # abcdef*hijkl abundance *g
```

## 分割 split、rsplit、partition、rpartition、splitlines
> `s.split(sep[, maxsplit]) -> list[str]`：按 sep 分割，从左边开始，只分割两次
> `s.rsplit(sep[, maxsplit]) -> list[str]`：按 sep 分割，从右边开始，只分割两次
> `s.patition(sep) -> tuple(str, str, str)`：按 sep 分割，从左边开始，分成 (前部，分割符号，后部)
> `s.rpartition(sep) -> tuple(str, str, str)`：按 sep 分割，从右边开始，分成 (前部，分割符号，后部)

```python
s1 = 'Alice,Boii,Candy,Dannis,Eva,Fiona'

print(s1.split(','))                    # ['Alice', 'Boii', 'Candy', 'Dannis', 'Eva', 'Fiona']
print(s1.split(',', 2))                 # ['Alice', 'Boii', 'Candy,Dannis,Eva,Fiona']
print(s1.rsplit(',', 2))                # ['Alice,Boii,Candy,Dannis', 'Eva', 'Fiona']


s2 = '2020.8.8拍摄.mp4'

print(s2.partition('.'))                # ('2020', '.', '8.8拍摄.mp4')
print(s2.rpartition('.'))               # ('2020.8.8拍摄', '.', 'mp4')
```

> `s.splitlines([keepends]) -> list[str]`：按换行符（\n，\r，\r\n）分割
keepends -- 在输出结果里是否保留换行符('\r', '\r\n', \n')，默认为 False，不包含换行符，如果为 True，则保留换行符。

```python
str1 = 'ab c\n\nde fg\rkl\r\n'
print(str1.splitlines())        # ['ab c', '', 'de fg', 'kl']

str2 = 'ab c\n\nde fg\rkl\r\n'
print(str2.splitlines(True))    # ['ab c\n', '\n', 'de fg\r', 'kl\r\n']
```

## 切片
从字符串里截取一段指定的内容，生成一个新的字符串
切片都是**包头不包尾**

具体参考 [高级特性->切片]()

## 转换大小写

```python
s = 'hello woRlD.good mornIng'

# 将第一个字符转成大写，其他字符转成小写
print(s.capitalize())   # Hello world.good morning
# 将全部字符转成大写
print(s.upper())        # HELLO WORLD.GOOD MORNING
# 将全部字符转成小写
print(s.lower())        # hello world.good morning
# 将每个单词首字母转成大写
print(s.title())        # Hello World.Good Morning
```


## 填充
> `s.ljust(weight, fillchar='') -> str`
> `s.rjust(weight, fillchar='') -> str`
> `s.center(weight, fillchar='') -> str`

不足 weight 的话，用 fillchar 填充，fillchar默认空格

```python
print('|' + 'Monday'.ljust(10) + '|')        # |Monday    |
print('|' + 'Monday'.ljust(10, '^') + '|')   # |Monday^^^^|
print('|' + 'Monday'.rjust(10) + '|')        # |    Monday|
print('|' + 'Monday'.rjust(10, '#') + '|')   # |####Monday|
print('|' + 'Friday'.center(20, '*') + '|')  # |*******Friday*******|
print('|' + 'Friday'.center(4, '*') + '|')   # |Friday|
```

## 修剪
> `s.lstrip(chars) -> str`
> `s.rstrip(chars) -> str`
> `s.strip(chars) -> str`

删除字符串 左边 / 右边 / 左边+右边 的 chars 字符

```python
print('|' + '   Couldy Sunny   '.lstrip() + '|')      # |Couldy Sunny   |
print('|' + '   Couldy Sunny   '.rstrip() + '|')      # |   Couldy Sunny|
print('|' + '   Couldy Sunny   '.strip() + '|')       # |Couldy Sunny|

print('|' + '   Couldy+Sunny+++'.lstrip('+') + '|')   # |   Couldy+Sunny+++|
print('|' + '   Couldy+Sunny+++'.rstrip('+') + '|')   # |   Couldy+Sunny|
print('|' + '   Couldy+Sunny+++'.strip('+') + '|')    # |   Couldy+Sunny|
```

## 拼接
> `s.join(iterable) -> str`
> 将 s 拼接到 iterable 的每一个元素上

```python
li = ['Alice', 'Boii', 'Candy', 'Dannis', 'Eva', 'Fiona']
print(' - '.join(li))       # Alice - Boii - Candy - Dannis - Eva - Fiona

print('+'.join('Hello'))    # H+e+l+l+o

dic = {'name1': 'Alice', 'name2': 'Boii'}
print(' - '.join(dic))      # name1 - name2
```

## 编码

- **ASCII**
    0~127共128个，占用 1byte 即 8bit 的长度，共使用了7位的长度，从0000 0000 到 0111 1111。
    48-57 -> 0-9，65-90 -> A-Z，97-122 -> a-z
- **ISO-8859-1**
    0~255共256个，占用 1byte 即 8bit 的长度，共使用了8位的长度，从 0000 0000到1111 1111。
    0~127与ASCII完全兼容，128~255是一些欧洲字母
- **Unicode**
    万国码，汉字每个占 3byte
- **GBK**
    国标扩，汉字每个占 2byte

## 长度

## 格式化





## 不可变性
字符串是不可变对象。
当执行一个 `str = 'abc'` 的语句时，
python在字符串池中创建了一个字符串 `'abc'` ，
然后在内存中创建一个变量 `str`指向字符串池中的 `'abc'`

如果对`str`进行修改 `str = 'xyz'`
python会在字符串池中再创建一个字符串 `'xyz'`，
然后让变量 `str` 指向字符串池中的 `'xyz'`
此时字符串池其实有 `'abc'` 和 `'xyz'`，只不过`'abc'`没有被指向， `'xyz'` 被`str`指向

>所以，对于不变对象来说，
>调用对象自身的任意方法，也不会改变该对象自身的内容。
>相反，这些方法会创建新的对象并返回
>这样，就保证了不可变对象本身永远是不可变的。