# 测试

测试分为 `单元测试`、`组件测试`、`系统测试`、`性能测试`，逐级上升。

所谓的`测试驱动开发(TDD：Test-Driven Development)`，就是说每写完一个小功能，就要做一个完整的单元测试，每次进行改动以后都要进行一次单元测试，以确保功能正常。

每个单元测试都通过以后就可以进行组件测试，所有组件测试都通过就可以进行系统测试，系统测试通过就可以进行性能测试，性能测试类似于“烤机”，测试系统的最大承受能力，承受峰值等等。

## 单元测试
单元测试就是对一个模块、一个函数或者一个类进行正确性检验的检测工作
举个栗子：

对一个求绝对值函数 `abs(num)`，可以编写一下几个测试用例：

1. 输入正数 如 `1`、`4.1`、`5.9`，期待返回正整数 `1`、`4.1`、`5.9`
2. 输入负数 如 `-1.5`、`-3.7`、`20`，期待返回正整数 `1.5`、`3.7`、`20`
3. 输入 `0`，期待返回 `0`
4. 输入非数值类型 如 `None`、`[]`、`{}`，期待抛出`TypeError`。

将上面4个测试用例放到一个测试模块里，就是一个完整单元测试。

单元测试能通过了，说明这个函数功能正常，如果不通过，要么函数有问题，要么测试用例有问题。

所以要修复直到单元测试能够通过。

这样做的好处是，如果我们对`abs()`做了修改，只要再进行一次单元测试，可以通过，就说明修改没有影响函数的功能。如果不通过则要找出问题，修改到通过单元测试为止。极大程度上确保该模块行为仍然是正确的。

## 文档测试

一个函数、模块、类的第一个匿名字符串，就是文档。如

```python
def func():
    '''
    这里是文档
    '''
    pass


class cls:
    '''
    这里是文档
    '''
    pass
```

文档的内容可以作为测试集来进行测试。

- 第一步：编写文档。文档中 `>>>`的行会被执行，这与在交互模式下执行语句的相同的。
- 第二步：`if __name__ == '__main__': import doctest; doctest.testmod()`
- 第三步：运行文件

例如上面的绝对值函数`abs()`可以这样写

```python
def abs(num):
    '''
    The function will return the absolute value of num.

    >>> abs(11)
    11
    >>> abs(-41)
    41
    >>> abs(0)
    0
    >>> abs('a')
    Traceback (most recent call last):
        ...
    TypeError: '>=' not supported between instances of 'str' and 'int'
    '''
    return n if n >= 0 else (-n)


if __name__ == '__main__':
    import doctest
    doctest.testmod()
```
文档中以 `>>>` 表示测试用例，并在下行写出结果。如果测试结果正确，则什么也不会输出。

### 小结
doctest非常有用，不但可以用来测试，还可以直接作为示例代码。通过某些文档生成工具，就可以自动把包含doctest的注释提取出来。用户看文档的时候，同时也看到了doctest。

## Unittest
1. 环境搭建：Python已内置Unittest框架，直接`import unittest`
2. 四大组件：
    1. test fixture：`setUp`（前置条件）、`tearDown`（后置条件），用于初始化测试用例及清理和释放资源
    2. test case：测试用例，通过继承`unittest.TestCase`实现用例的继承。在Unittest中，测试用例都是通过test前缀或后缀来识别的。
        `def test_xxx(self)`、`def xxx_test(self)`
    3. test suite：测试套件，即测试用例的集合。
    4. test runner：运行器，一般通过`runner`来调用`suite`去执行测试。
3. 运行机制：通过在main函数中，调用`unittest.main()`运行所有内容


```python
import unittest

class forTest(unittest.TestCase):
    # 初始化
    def setUp(self) -> None:
        print('setUp')

    # 释放
    def tearDown(self) -> None:
        print('tearDown')

    # 测试用例1
    def test_a(self):
        print('a')

    # 测试用例2
    def test_b(self):
        print('b')


if __name__ == '__main__'；
    unittest.main()

--------------------------------------------------

# Output:
setUp
a
tearDown
setUp
b
tearDown
```