# 模块 Module

!!! note ""
    一个 `.py`文件就称之为一个`模块 module`

好处：

1. 提高了代码的可维护性；
2. 可以被其他地方应用；
3. 可以避免命名冲突。

## import

import加载的模块分为四个通用类别：

1. 使用python编写的代码（`py文件`）；
2. 已被编译为共享库或DLL的`C或C++扩展`；
3. 包好一组模块的`包`
4. 使用C编写并链接到python解释器的`内置模块`.

## 包 Package

!!! note ""
    模块的上一级称为`包 package`

- 包是一个文件夹，可以通过包来组织模块，避免冲突。
- 一个包里必须含有一个`__init__.py`
    1. `__init__.py`可以是空文件，也可以有Python代码
    2. `__init__.py`本身就算一个模块，它的模块名就是包名
- 包里面还可以有包

```python
MyPackage
 ├─ web
 │  ├─ __init__.py
 │  ├─ utils.py
 │  └─ test.py
 ├─ dirc
 │  ├─ utils.py
 │  └─ test.py
 │
 ├─ __init__.py
 ├─ main.py
 └─ utils.py
```

- 顶层包为 MyPackage
- 子包为 web
- 普通目录 dirc （因为它没有`__init__.py`文件）
- `test.py`的模块名为`MyPackage.web.test`
- `utils.py`有两个，模块名分别为`MyPackage.web.utils`和`MyPackage.utils`

!!! note ""
    自己创建模块时注意命名不要与系统模块冲突

    要检查系统中是否存在该模块，可以在交互环境下执行 `import abc`，若成功说明系统存在此模块。

```python
>>> import aabbccc
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'aabbccc'
```

## 使用模块

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"Here is the module documentation"

__author__ = 'Boii'

import sys


def test():
    args = sys.argv
    if len(args) == 1:
        print('Hello, world!')
    elif len(args) == 2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')


if __name__ == '__main__':
    test()

```

### 标准模块文件模板

- `#!/usr/bin/env python3` 是向Unix / Linux / Mac 系统声明本文件是Python3文件
- `# -*- coding: utf-8 -*-` 表示本文件使用标准utf-8编码，虽然Python3之后默认支持中文，但为保稳妥还是写上
- ` "Here is the module documentation" `是模块文档注释，任何模块的第一个字符串都被视为模块的文档注释。可以通过`模块名.__doc__`获得。
- `__author__ = 'Boii'`是作者。可以通过`模块名.__author__`获得。

标准模块文件模板也可以不写，并不影响。

### 导入、参数列表、__name__

- `import sys`引入了内置模块`sys`
- `args = sys.argv`，这里`sys.argv`是参数列表，是一个 list，保存的是`通过命令执行本文件的时候所带的参数`。但该 list的第一个元素永远是文件名。
    - 例如上面的文件名为`Hello.py`，则运行命令`python Hello.py`后sys,argv为`['Hello.py']`.
    - 如果执行的命令为`python Hello.py Boii`，则sys.argv为`['Hello.py', 'Boii']`
- `if __name__ == '__main__': test()`如果这个文件是独立运行的，则 if 为 True，如果这个文件是被导入的，则 if 为 False。

```shell
$ python Hello.py
Hello, world!
$ python Hello.py Boii
Hello, Boii!
```
↑ 通过命令独立运行这个文件，所以 if 为 True，执行了 test() 函数

```shell
$ python
Python 3.8.3
>>>
>>> import Hello
>>>
>>> Hello.test()
Hello, world!
```
↑ 通过导入的方式运行这个文件，所以 if 为 False，没有执行 test() 函数，等到 Hello.test() 主动调用时才执行了 test() 函数


## 总结

一个 .py 文件就是一个模块 module
一个带有 `__init__.py` 的文件夹就是一个包 package