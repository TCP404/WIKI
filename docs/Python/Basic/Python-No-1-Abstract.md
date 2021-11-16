---
title: Python【No-1】总叙
tags: Python
categories:
  - Python
  - 基础
thumbnail: 'https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 15463
date: 2020-07-02 10:41:48
---

关于Python

<!--more-->


- 语言哲学： 简洁
- 类型：解释型语言、动态语言、面向对象（一切皆对象）
- 缺点：
    1. 速度慢
    2. 代码不能加密

- 语言思想：
    1. 一切皆对象
    2. 函数是一等公民

## 解释器
### CPython
> 默认解释器。

### PyPy
> 目标是执行速度，采用JIT技术，对python代码进行动编译。
> 但是与CPython不同。 [PyPy VS CPython](http://pypy.readthedocs.org/en/latest/cpython_differences.html)

### Jython
> Jython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码执行。

### IronPython
> IronPython和Jython类似，只不过IronPython是运行在微软.Net平台上的Python解释器，可以直接把Python代码编译成.Net的字节码。

## 命令行模式 & 交互模式
### 命令行模式

┌──────────────────────────────────────┐
│Command Prompt                                               - □ x │
├──────────────────────────────────────┤
│Microsoft Windows [Version 10.0.0]                              │
│(c) 2015 Microsoft Corporation. All rights reserved.       │
│                                                                                    │
│C:\> _                                                                           │
│                                                                                    │
└──────────────────────────────────────┘

> 可以切换到文件所在目录下，然后输入 `python 文件名.py` 来执行python文件


### Python交互模式

┌────────────────────────────────────────┐
│Command Prompt - python                                     - □ x │
├────────────────────────────────────────┤
│Microsoft Windows [Version 10.0.0]                                  │
│(c) 2015 Microsoft Corporation. All rights reserved.           │
│                                                                                        │
│C:\> python                                                                     │
│Python 3.7 ... on win32                                                     │
│Type "help", ... for more information.                                │
│>>> 100 + 200                                                                │
│300                                                                                  │
│>>> _                                                                              │
│                                                                                        │
└────────────────────────────────────────┘

> 输入 `python` 进入交互模式， 输入 `exit()` 或 `quit()` 退出交互模式

### 文件名
- 扩展名：.py
- 文件名：英文字母、数字、下划线

## 直接运行.py文件
- [ ] Win
- [x] Linux
- [x] Mac

前提：
> 1)  在hello.py文件的第一行加上一个特殊的注释 `#!/usr/bin/env python3` ， 如
> ```python
> #!/usr/bin/env python3
>
> print("Hello World!")
> ```
> 2)  通过命令给hello.py文件执行权限
> ```shell
> $ chmod a+x hello.py
> ```

## 中文编码
> Python2 默认编码格式是：ASCII，使用中文会出错
> 解决方法：在文件开头加入 `# -*- coding: UTF-8 -*-` 或 `# coding=utf-8`

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

print("放码过来")
```

> Python3 默认编码格式是：UTF-8，所以无需指定编码格式
> 注意：py文件需要存储格式为 UTF-8

```python
#!/usr/bin/env python3

print("放码过来")
```

但最好还是加上

```python
#!/usr/bin/env python3
# -*- coding: UTF-8 -*-

print("放码过来")
```

## python的一切皆对象

```python
# 示例如下
a=2019
b="一切皆对象"
print(type(2019))
print(type(int))
print(type(b))
print(type(str))

class Student:
    pass

stu = Student()
print(type(stu))
print(type(Student))
print(int.__bases__)
print(str.__bases__)
print(Student.__bases__)
print(type.__bases__)
print(object.__bases__)
print(type(object))
print(type(type))
```
运行结果：
```python
<class 'int'>				 # 2019是由int这个类创建的实例
<class 'type'>				 # int这个类是由type这个类创建的实例
<class 'str'>				 # 同上
<class 'type'>
<class '__main__.Student'>   # stu是类Student创建的实例
<class 'type'>				 # 类Student是由type这个类创建的实例
(<class 'object'>,)			 # 类int的基类是object这个类
(<class 'object'>,)			 # 同上
(<class 'object'>,)			 # 同上
(<class 'object'>,)			 # 重点：类type的基类也是object这个基类
()							 # 重点：类object没有基类
<class 'type'>				 # 难点：类object是由类type创建的实例
<class 'type'>				 # 难点：类type是由type类自身创建的实例
```

![一切皆对象](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Py/1-1-一切皆对象.png)


对于上面图片的解读如下：

1. object是一切对象：list、str、dict、tuple的基类，同时object是type的实例
2. 类type是自身的实例，同时type也继承自object类
3. 由结论1和结论2，得出一切皆对象，同时一切皆继承自object类


`类object`是一切对象的基类
`类object`是`类type`的实例

`类type`继承自`类object`
`类type`是`类type`的实例

一切皆对象，一切皆继承自object类
