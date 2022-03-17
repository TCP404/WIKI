---
title: Python【No-7】错误处理
tags: Python
categories:
  - Python
  - 进阶
  - 错误与调试
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 63024
date: 2020-07-08 10:41:48
---

语言必修课：错误处理

<!--more-->

## 错误处理
> `try...except1...[exceptN...[finally...]`

```python
try:
    可能会出现异常的代码
except 异常类型 [as 别名]：
    出现异常后的处理
finally:
    不管是否出现异常最后都会执行的代码
```

示例：
```python
# 出现 除零的异常 示例

try:
    print("Try---------")
    r = 10/0
    print("result: ", r)
except ZeroDivisionError as e:
    print("Except------", e)
finally:
    print("Finally-----")

print("END")

--------------------------------------------------

# Output:
Try---------
Except------ division by zero
Finally-----
END
```

try 语句块中的代码遇到异常后会跳转到 except 语句块执行
而 finally 语句块不管有没有异常最后都会执行。当然，也可以没有 finally 语句

```python
# 没有异常的示例

try:
    print("Try---------")
    r = 10/2
    print("result: ", r)
except ZeroDivisionError as e:
    print("Except------", e)
finally:
    print("Finally-----")

print("END")

--------------------------------------------------

# Output:
Try---------
result: 5
Finally-----
END
```
### 捕获多个异常

捕获异常不止可以捕获一个，还可以`捕获多个`
```python
try:
    print("Try---------")
    r = 10 / int('a')
    print("result: ", r)
except ValueError as e:
    print('ValueError: ', e)
except ZeroDivisionError as e:
    print("ZeroDivisionError: ", e)
finally:
    print("Finally-----")

print("END")

--------------------------------------------------

# Output:

Try---------
ValueError:  invalid literal for int() with base 10: 'a'
Finally-----
END
```

### else 语句
在 except 后面还可以加一个 `else 语句块`

```python
# else 语句在没有异常的情况下 示例

try:
    print("Try---------")
    r = 10 / int('2')
    print("result: ", r)
except ValueError as e:
    print('ValueError: ', e)
except ZeroDivisionError as e:
    print("ZeroDivisionError: ", e)
else:
    print("Else--------No Error!")
finally:
    print("Finally-----")

print("END")

--------------------------------------------------

# Output:

Try---------
result: 5.0
Else--------No Error!
Finally-----
END
```
else 语句不同于finally 语句。finally 语句是有无异常都会执行，else 语句会在没有异常的情况下执行
```python
# else 语句在有异常的情况下 示例

try:
    print("Try---------")
    r = 10 / int('a')
    print("result: ", r)
except ValueError as e:
    print('ValueError: ', e)
except ZeroDivisionError as e:
    print("ZeroDivisionError: ", e)
else:
    print("Else--------No Error!")
finally:
    print("Finally-----")

print("END")

--------------------------------------------------

# Output:
Try---------
ValueError:  invalid literal for int() with base 10: 'a'
Finally-----
END
```

### 优点：跨越多层调用
例如现在 `A()` 调用 `B()`，`B()` 调用 `C()`，在 `C()` 出错了，只要 `A()` 捕获到了，就可以处理。

```python
def C(s):
    return 10 / int(s)

def B(s):
    return C(s) * 2

def A():
    try:
        B('0')
    except Exception as e:
        print("Error:", e)
    finally:
        print("Finally------")

A()

--------------------------------------------------

# Output:

Error: division by zero
Finally------
```
虽然错误是在 `C()` 出现，但是在 `A()` 也能捕获。
如果错误没有被捕获，它就会一直往上抛，最后被Python解释器捕获，打印一个错误信息，然后程序退出。

### 错误是个class
所有的错误类型都继承自 `BaseException`，如果捕获到了一个父类错误，其子类错误不会被捕获。
例如：
```python
try:
    foo()
except ValueError as e:
    print('ValueError')
except UnicodeError as e:
    print('UnicodeError')
```
第二个except永远捕获不到 `UnicodeError`，因为`UnicodeError`是`ValueError`的子类。
即使有`UnicodeError`，也被第一个except捕获了。

#### 错误类继承关系
```
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration 
      +-- AttributeError     属性错误
      +-- AssertionError     断言错误
      +-- BufferError        缓冲错误
      +-- EOFError           文件结束符错误
      +-- MemoryError        内存错误
      +-- ReferenceError
      +-- SystemError        系统错误
      +-- TypeError          类型错误
      +-- ArithmeticError    算术错误
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- ImportError        导入错误
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- NameError          命名错误
      |    +-- UnboundLocalError
      +-- OSError            操作系统错误
      |    +-- BlockingIOError      IO阻塞错误
      |    +-- ChildProcessError    子进程错误
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- RuntimeError       运行时错误
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError        同步错误
      |    +-- IndentationError
      |         +-- TabError
      +-- ValueError         值错误
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning            警告
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

### 抛出错误
错误是 class， 捕获错误实质上就是捕获到该 class 的一个实例。因此，错误是可以有意创建并抛出的。

`try...except...`是承接错误，而`raise`是抛出错误

```python
def register():
    username = input("请输入用户名：")
    if len(username) < 6:
        raise Exception("用户名长度必须6位以上")
    else:
        print("输入的用户名是:", username)


register()

--------------------------------------------------

# Output1:
请输入用户名：admin
Traceback (most recent call last):
  File "d:/---Programming---/Python/Project/mo2.py", line 9, in <module>
    register()
  File "d:/---Programming---/Python/Project/mo2.py", line 4, in register
    raise Exception("用户名长度必须6位以上")
Exception: 用户名长度必须6位以上

# Output2:
请输入用户名：administor
输入的用户名是: administor
```
在 if 中主动`raise`抛出错误，但是调用`register()`的地方没有`try...except`来捕获错误，所以最后由解释器处理，打印出了错误信息


```python
def register():
    username = input("请输入用户名：")
    if len(username) < 6:
        raise Exception("用户名长度必须6位以上")
    else:
        print("输入的用户名是：", username)

try:
    register()
except Exception as e:
    print(e)
    print("注册失败")
else:
    print("注册成功")

--------------------------------------------------

# Output1:
请输入用户名：admin
用户名长度必须6位以上
注册失败

# Output2:
请输入用户名：administor
输入的用户名是: administor
注册成功
```
在 if 处 `raise`抛出错误，在`register()`使用了`try...except`捕获错误，然后进行了处理。

### 自定义错误
主动抛出错误，这个错误可以是自己定义的 class，因为错误也是 class。
```python
class NameUndercutting(Exception):
    pass

def register():
    username = input("请输入用户名：")
    if len(username) < 6:
        raise Exception("用户名长度必须6位以上")
    else:
        print("输入的用户名是：", username)

try:
    register()
except NameUndercutting as e:
    print(e)
    print("注册失败")
else:
    print("注册成功")

--------------------------------------------------

# Output1:
请输入用户名：admin
用户名长度必须6位以上
注册失败

# Output2:
请输入用户名：administor
输入的用户名是: administor
注册成功
```