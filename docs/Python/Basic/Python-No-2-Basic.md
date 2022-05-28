# 基础
## 输入 & 输出

### 输出
> print('输出内容')

```python
>>> print(300)                         # 300
>>> print(100 + 200)                   # 300
>>> print('100 + 200 =', 100 + 200)    # 100 + 200 = 300
>>> print('Hello World!')              # Hello World!
>>> print('The quick', 'brown for', 'jumps over.')    # The quick brown for jumps over.
```

### 输入

!!! note ""
    承接变量 = input('提示信息')

=== "命令行式"
    ```python
    >>> name = input()
    Boii
    >>> name
    'Boii'
    >>> print(name)
    Boii
    ```

=== "文件方式"

    ```python3
    # name.py

    name = input('Please enter your name:')
    print('Hello', name)
    ```

    ```sh
    > python name.py

    Please enter your name: Boii
    Hello Boii
    ```

## 变量

!!! note

    1. 大小写英文、数字、下划线_
    2. 不能数字开头
    3. 大小写敏感
    4. 不能用关键字
    5. 不需要声明类型
    6. 简洁明了，信达雅原则。变量名用名词，函数名方法名用动词
    7. 由于python是动态语言，不需要声明类型，所以命名尽量体现类型或用前缀体现。如 i_age, fPrice, b_flag
    8. 慎用字母O和I

```python
a = 1        # a是一个整型
b = 1.0      # b是一个浮点型
c = 'abc'    # c是一个字符串
d = True     # d是一个布尔值
e = None     # e是一个空值

# 多个变量同时赋值

a = b = c = 1
# 以上语句, 内存中会创建一个空间,值为1; 再创建三个空间a, b, c, 存放1的地址，即指向1那块内存

x, y, z = 1, 2, "Boii"
# 以上语句, x为1, y为2, z为Boii
```

### 变量实际上是指向

```python
>>> a = 'ABC'
>>> b = a
>>> a = 'XYZ'
>>> print(b)
ABC
```

`a = 'ABC'`

![1](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/2-1.png)


`b = a`

![2](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/2-2.png)


`a = 'XYZ'`

![2](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/Py/2-3.png)


## 数据类型

[数据类型.md](https://tcp404.com/2020/07/09/Python%E3%80%90No-4%E3%80%91%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B/)


## 运算符

[运算符](https://tcp404.com/2020/07/10/Python%E3%80%90No-5%E3%80%91%E8%BF%90%E7%AE%97%E7%AC%A6/)


## 缩进
Python以缩进来区分代码块。连续的相同缩进视为同一代码块，同一作用域，如同C中的花括号。

!!! note
    按照约定俗成，4个空格为一个缩进。在IDE中最好设置好tab自动转换为4个空格

### 多行语句
如果一个语句太长，可以使用 `\` 声明此语句未结束

```python3
total = item_one + \
        item_two + \
        item_three
```

## 注释
```python3
# 这是单行注释

'''
这是多行注释，使用单引号
这是多行注释，使用单引号
这是多行注释，使用单引号
'''

"""
这是多行注释，使用双引号
这是多行注释，使用双引号
这是多行注释，使用双引号
"""
```