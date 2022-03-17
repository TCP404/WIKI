---
title: Python【No-8】调试
tags: Python
categories:
  - Python
  - 进阶
  - 错误与调试
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 24435
date: 2020-07-09 10:41:48
---

开发的必经之路：调试

<!--more-->


大型项目中，一般都是使用日志来调试程序。不过有时候有一小块代码想做调试的时候，可以用`print`和`assert`。

## print & assert
`print()` 就是常见的打印函数。

凡是用`print()`的地方都可以用 `断言assert语句`代替

```python
# 用print

def foo(s):
    n = int(s)
    if n == 0:
        print("n is zero!")
    else:
        return 10 / n

foo('0')

--------------------------------------------------

# Output:
Traceback (most recent call last):
 ...
ZeroDivisionError: division by zero
```

```python
# 用assert

def foo(s):
    n = int(s)
    assert n == 0 , "n is zero!"
    return 10 / n

foo('0')

--------------------------------------------------

# Output:
Traceback (most recent call last):
 ...
AssertionError: n is zero!
```


当assert后面的表达式为 False 时，则抛出 `AssertionError`错误

`assert expression`等价于
```python
if not expression:
    raise AssertionError
```

`assert expression [,arguments]`等价于
```python
if not expression:
    rasise AssertionError(arguments)
```

例如：断言当前系统为 linux
```python
import sys
assert (linux in sys.platform), "该代码只能在 linux 系统下执行"
```

## 日志模块 logging
Python 标准库中有一个日志模块 `logging`，相当于 java 中的 `log4j`

在**软件开发阶段**或**部署开发环境**时，
为了尽可能详细的查看应用程序的运行状态来保证上线后的稳定性，
我们可能需要把该应用程序所有的运行日志全部记录下来进行分析，这是非常耗费机器性能的。

当**应用程序正式发布**或**在生产环境部署应用程序**时，
我们通常只需要记录应用程序的`异常信息`、`错误信息`等，
这样既可以减小服务器的I/O压力，也可以避免我们在排查故障时被淹没在日志的海洋里。

### logging 日志等级
把日志分为6个等级

|  Level   | value | Description                                                        |
| :------: | :---: | :----------------------------------------------------------------- |
|  NOTEST  |   0   |                                                                    |
|  DEBUG   |  10   | 最详细的日志信息，典型应用场景是 问题诊断                          |
|   INFO   |  20   | 通常只记录关键节点信息，用于确认一切都是按照我们预期的那样进行工作 |
| WARNING  |  30   | 当某些不期望的事情发生时记录的信息，但是此时应用程序还是正常运行的 |
|  ERROR   |  40   | 由于一个更严重的问题导致某些功能不能正常运行时记录的信息           |
| CRITICAL |  50   | 当发生严重错误，导致应用程序不能继续运行时记录的信息               |


日志等级是从下到上依次升高的（值越大等级越高）
即：NOTEST < DEBUG < INFO < WARNING < ERROR < CRITICAL，
而日志的信息量是依次减少的；

开发应用程序或部署开发环境时，可以使用`DEBUG`或`INFO`级别的日志获取尽可能详细的日志信息来进行开发或部署调试；
应用上线或部署生产环境时，应该使用`WARNING`或`ERROR`或`CRITICAL`级别的日志来降低机器的I/O压力和提高获取错误日志信息的效率。

日志级别的指定通常都是在`应用程序的配置文件`中进行指定的。


### logging 配置
> `loggong.basicConfig(**kwargs)`

**kwargs 可以接收关键字参数如下：

| 参数名称 | 描述                                                                                              | 取值范围                                                             |
| :------- | :------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------- |
| filename | 指定日志输出目标文件的文件名，指定该设置项后日志信息就不会被输出到控制台了                        | string                                                               |
| filemode | 指定日志文件的打开模式，默认为'a'。该选项要在filename指定时才有效                                 | a、w                                                                 |
| format   | 指定日志格式字符串，即指定日志输出时所包含的字段信息以及它们的顺序                                | 详见下表                                                             |
| datefmt  | 指定日期/时间格式，该选项要在format中包含时间字段%(asctime)s时才有效                              | [datefmt](https://docs.python.org/3/library/time.html#time.strftime) |
| level    | 指定日志器的日志级别                                                                              | logging.DEBUG、logging.INFO...                                       |
| stream   | 指定日志输出目标stream。                                                                          | sys.stdout、sys.stderr以及网络stream                                 |
| style    | 指定format格式字符串的风格，可取值为，默认为'%'                                                   | %、{、$                                                              |
| handlers | 该选项如果被指定，它应该是一个创建了多个Handler的可迭代对象，这些handler将会被添加到root logger。 | 一个创建了多个Handler的可迭代对象                                    |

`filename`、`stream`和`handlers`这三个配置项只能有一个存在，不能同时出现2个或3个，否则会引发ValueError异常。

format 参数的取值表

| 名称            | 使用格式            | 描述                                                                                  |
| :-------------- | :------------------ | :------------------------------------------------------------------------------------ |
| asctime         | %(asctime)s         | 日志事件发生的时间--**人类可读时间**，如：2003-07-08 16:49:45,896                     |
| created         | %(created)f         | 日志事件发生的时间--**时间戳**，就是当时调用time.time()函数返回的值                   |
| relativeCreated | %(relativeCreated)d | 日志事件发生的时间相对于logging模块加载时间的**相对毫秒数**（目前还不知道干嘛用的）   |
| msecs           | %(msecs)d           | 日志事件发生的时间的**毫秒部分**                                                      |
| --------------- | ------------------- | ----------------------------------------------------------------------------          |
| levelname       | %(levelname)s       | 该日志记录的文字形式的**日志级别**（'DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL'） |
| levelno         | %(levelno)s         | 该日志记录的数字形式的**日志级别**（10,20,30,40,50）                                  |
| lineno          | %(lineno)d          | 调用日志记录函数的**源代码所在的行号**                                                |
| funcName        | %(funcName)s        | 调用日志记录函数的**函数名**                                                          |
| name            | %(name)s            | 所使用的**日志器名称**，默认是'root'，                                                |
| message         | %(message)s         | 日志记录的**文本内容**，通过 msg % args计算得到的                                     |
| --------------- | ------------------- | ----------------------------------------------------------------------------          |
| pathname        | %(pathname)s        | 调用日志记录函数的源码文件的**全路径**                                                |
| filename        | %(filename)s        | pathname的文件名部分，包含文件后缀                                                    |
| module          | %(module)s          | filename的名称部分，不包含后缀                                                        |
| --------------- | ------------------- | ----------------------------------------------------------------------------          |
| process         | %(process)d         | 进程ID                                                                                |
| processName     | %(processName)s     | 进程名称，Python 3.1新增                                                              |
| thread          | %(thread)d          | 线程ID                                                                                |
| threadname      | %(thread)s          | 线程名称                                                                              |

```python
import logging

LOG_FORMAT = "%(asctime)s - %(levelname)s - %(message)s"    # 日志打印格式
DATE_FORMAT = "%Y-%m-%d %H:%M:%S"    # 时间打印格式

# 配置日志
logging.basicConfig(filename='my.log', level=logging.DEBUG, format=LOG_FORMAT, datefmt=DATE_FORMAT)

logging.debug("This is a debug log.")
logging.info("This is a info log.")
logging.warning("This is a warning log.")
logging.error("This is a error log.")
logging.critical("This is a critical log.")
```
以上代码 配置了日志文件，级别设置为 DEBUG，输出格式为 `日志发生时间 - 日志等级 - 日志信息`

执行后结果如下：

![1](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/8-1.png)


执行第二次后结果如下：

![2](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/8-2.png)


如果要让日志格式好看点，还可以指定宽度。
比如执行 日志等级 的打印宽度为 8，在 s 前面加上宽度即可：`%(levelname)8s`
如果要左对齐，则写上减号 -：`%(levelname)-8s`

```python
# 导入模块
import logging

LOG_FORMAT = "%(asctime)s - %(levelname)-8s - %(message)s"    # 日志打印格式
DATE_FORMAT = "%Y-%m-%d %H:%M:%S"    # 时间打印格式

# 配置日志
logging.basicConfig(filename='my.log', level=logging.DEBUG, format=LOG_FORMAT, datefmt=DATE_FORMAT)

# 打日志
logging.debug("This is a debug log.")
logging.info("This is a info log.")
logging.warning("This is a warning log.")
logging.error("This is a error log.")
logging.critical("This is a critical log.")
```

左对齐：%(levelname)-8s

![3](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/8-3.png)


右对齐：%(levelname)8s

![4](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/8-4.png)


### logging 模块使用
> 用法：
> 1. 导入 logging
> 2. 配置(可选)
> 3. 打日志


logging 模块提供了两种记录日志的方式：
- 使用 logging 提供的`模块级别`的函数
- 使用 logging 日志系统的`四大组件`

logging所提供的`模块级别`的日志记录函数也是对logging`日志系统相关类`的封装而已。

#### 函数用法

常用模块级别函数

| Function                                  | Description                            |
| :---------------------------------------- | :------------------------------------- |
| logging.debug(msg, \*args, \*\*kwargs)    | 创建一条严重级别为 DEBUG 的日志记录    |
| logging.info(msg, \*args, \*\*kwargs)     | 创建一条严重级别为 INFO 的日志记录     |
| logging.warning(msg, \*args, \*\*kwargs)  | 创建一条严重级别为 WARNNING 的日志记录 |
| logging.error(msg, \*args, \*\*kwargs)    | 创建一条严重级别为 ERROR 的日志记录    |
| logging.critical(msg, \*args, \*\*kwargs) | 创建一条严重级别为 CRITICAL 的日志记录 |
| logging.log(msg, \*args, \*\*kwargs)      | 创建一条严重级别为 LOG 的日志记录      |
| logging.basicConfig(\*\*kwargs)           | 对root logger进行一次性配置            |

```python
import logging    # 导入模块

# 打日志
logging.debug("This is a debug log.")
logging.info("This is a info log.")
logging.warning("This is a warning log.")
logging.error("This is a error log.")
logging.critical("This is a critical log.")


或

import logging    #导入模块

# 打日志
logging.log(logging.DEBUG, "This is a debug log.")
logging.log(logging.INFO, "This is a info log.")
logging.log(logging.WARNING, "This is a warning log.")
logging.log(logging.ERROR, "This is a error log.")
logging.log(logging.CRITICAL, "This is a critical log.")
--------------------------------------------------

# Output:
WARNING:root:This is a warning log.
ERROR:root:This is a error log.
CRITICAL:root:This is a critical log.
```
这里只输出 warning 级别以上的信息。这是因为默认的日志等级是 warning ，所以 debug 和 info 级别的信息就被忽略了。
logging 只输出 大于等于 所设置级别以上的日志信息。
这点可以在 `logging.basicConfig()`中设置