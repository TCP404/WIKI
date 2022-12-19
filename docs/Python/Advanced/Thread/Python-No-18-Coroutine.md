---
title: Python【No-18】协程
tags: Python
categories:
  - Python
  - 进阶
  - 进程、线程
thumbnail: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/thumbnail/python.png'
date: 2020-07-19 14:41:48
---

协程，时间管理大师

<!--more-->

> **线程**是系统级别的，由操作系统调度
> **协程**是程序级别的，由程序根据需要自己调度

协程可以想象成线程里的线程，也就是将一个线程里多个任务分拆给多个协程。

使用线程的时候是：线程执行遇到耗时（阻塞）的地方，再开一个线程，去执行这个耗时的任务，原本的线程继续执行。
就好比我在算题，算着算着计算量比较大比较耗时，我就叫了小明帮我算，我继续算下面的。

使用协程的时候是：利用别的协程执行耗时操作时的那点时间切换去别的协程继续执行，也就是在协程之间反复横跳。
就好比我在算题，算着算着计算量有点大，就先做别的题，回头自己再继续把题算完

协程的优点：
1. 无需线程上下文切换的开销，因为都在同一个线程里，避免了无意义的调度，由此可以提高性能。
2. 无需原子操作锁定及同步的开销
3. 方便切换控制流，简化编程模型
4. 高并发+高扩展性+低成本：一个CPU支持上万个协程都不是问题。所以很适合高并发处理。


缺点：
1. 因为协程都是程序调度的，所以需要程序员承担调度的责任，同时也失去了标准线程使用多CPU的能力
2. 无法利用多核资源：协程的本质是个单线程，不能同时用上单个CPU的多个核。协程需要和进程配合才能运行在多CPU上，不过绝大部分应用都没有这个必要，除非是CPU密集型应用
3. 进行阻塞操作（如IO）会阻塞掉整个程序。

## 使用 yield 实现并发

```python
import time 

def task_1():
    while True:
        print("{:-^16s}".format("task_1"))
        time.sleep(0.1)
        yield

def task_2():
    while True:
        print("{:-^16s}".format("task_2"))
        time.sleep(0.1)
        yield

def main():
    t1 = task_1()
    t2 = task_2()
    # 先让t1运行一会儿，当t1中遇到yield的时候，再返回到main()的while循环
    # 然后执行t2，当它遇到yield的时候，再次切换到t1中
    # 就这样t1/t2/t1/t2...交替运行，最终实现了多任务...协程
    while True:
        next(t1)
        next(t2)


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
...
```

## 使用 greenlet 实现并发

\>_: pip install greenlet

```python
from greenlet import greenlet
import time

def task_1():
    while True:
        print("{:-^16s}".format("task_1"))
        gr2.switch()        # 切换到 gr2中，也就是 task_2() 中运行
        time.sleep(0.5)


def task_2():
    while True:
        print("{:-^16s}".format("task_2"))
        gr1.switch()        # 切换到 gr1中，也就是 task_1() 中运行
        time.sleep(0.5)

gr1 = greenlet(task_1)
gr2 = greenlet(task_2)

# 切换到 gr1中，也就是 task_1() 中运行
gr1.switch

--------------------------------------------------

# Output:
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
-----task_1-----
-----task_2-----
...
```

## 使用 gevent 实现并发

> `import gevent`
> `gevent.spawn(funcName, args)`

先看看没有阻塞操作的时候：
```python
# 没有阻塞
import gevent


def task_1(n):
    for i in range(n):
        print(gevent.getcurrent(), i)


def task_2(n):
    for i in range(n):
        print(gevent.getcurrent(), i)


def task_3(n):
    for i in range(n):
        print(gevent.getcurrent(), i)


def main():
    g1 = gevent.spawn(task_1, 5)
    g2 = gevent.spawn(task_2, 5)
    g3 = gevent.spawn(task_3, 5)

    g1.join()
    g2.join()
    g3.join()


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
<Greenlet at 0x1f6eaf0cd00: task_1(5)> 0
<Greenlet at 0x1f6eaf0cd00: task_1(5)> 1
<Greenlet at 0x1f6eaf0cd00: task_1(5)> 2
<Greenlet at 0x1f6eaf0cd00: task_1(5)> 3
<Greenlet at 0x1f6eaf0cd00: task_1(5)> 4
<Greenlet at 0x1f6eaf0ce10: task_2(5)> 0
<Greenlet at 0x1f6eaf0ce10: task_2(5)> 1
<Greenlet at 0x1f6eaf0ce10: task_2(5)> 2
<Greenlet at 0x1f6eaf0ce10: task_2(5)> 3
<Greenlet at 0x1f6eaf0ce10: task_2(5)> 4
<Greenlet at 0x1f6eaf0cbf0: task_3(5)> 0
<Greenlet at 0x1f6eaf0cbf0: task_3(5)> 1
<Greenlet at 0x1f6eaf0cbf0: task_3(5)> 2
<Greenlet at 0x1f6eaf0cbf0: task_3(5)> 3
<Greenlet at 0x1f6eaf0cbf0: task_3(5)> 4
```
gevent 是利用协程阻塞的时候去执行别的协程
这里没有阻塞操作，所以并不会发生什么变化，跟普通函数调用一样，更提不上并发



现在来加点阻塞操作看看
```python
# 有阻塞
import gevent


def task_1(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        # time.sleep(0.1)    # 在gevent中，time.sleep()这种阻塞操作是不起作用的
        gevent.sleep(0.1)


def task_2(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        gevent.sleep(0.1)


def task_3(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        gevent.sleep(0.1)


def main():
    g1 = gevent.spawn(task_1, 5)
    g2 = gevent.spawn(task_2, 5)
    g3 = gevent.spawn(task_3, 5)

    g1.join()
    g2.join()
    g3.join()



if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
<Greenlet at 0x1433a62b370: task_1(5)> 0
<Greenlet at 0x1433a62b590: task_2(5)> 0
<Greenlet at 0x1433a62b480: task_3(5)> 0
<Greenlet at 0x1433a62b370: task_1(5)> 1
<Greenlet at 0x1433a62b590: task_2(5)> 1
<Greenlet at 0x1433a62b480: task_3(5)> 1
<Greenlet at 0x1433a62b370: task_1(5)> 2
<Greenlet at 0x1433a62b590: task_2(5)> 2
<Greenlet at 0x1433a62b480: task_3(5)> 2
<Greenlet at 0x1433a62b370: task_1(5)> 3
<Greenlet at 0x1433a62b590: task_2(5)> 3
<Greenlet at 0x1433a62b480: task_3(5)> 3
<Greenlet at 0x1433a62b370: task_1(5)> 4
<Greenlet at 0x1433a62b590: task_2(5)> 4
<Greenlet at 0x1433a62b480: task_3(5)> 4
```


上述代码 建立了三个 gevent 对象，里面分别转载了 task_1、task_2、task_3 三个执行函数。
接着三个 gevent 对象都使用 join 方法运行了执行函数
在三个执行函数中又用了 gevent.sleep() 模拟阻塞操作（在实际开发中并不会专门用sleep去阻塞，而是在执行到 IO 等耗时操作时，gevent自动切换。）

在执行 task_1 的时候，打印了第一句，然后遇到了 gevent.sleep() 的阻塞操作，切换到 task_2 ，打印了第二局，又遇到阻塞，又切换...一直到全部执行完毕

另外，task_1 中的 time.sleep() 在 gevent 管理的协程中是不起作用的，需要使用 gevent.sleep() 才行。

这就带来一个问题：如果我的代码是开发完了才加入了 gevent ，那岂不是要把很多地方手动改到 gevent 能接受。还好 gevent 提供了一个补丁

### gevent 补丁
> `from gevent import monkey`
> `monkey.patch_all()`
> 一定要写在最上方

```python
import time
import gevent
from gevent import monkey

monkey.patch_all()    # 打补丁


def task_1(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        time.sleep(0.1)


def task_2(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        time.sleep(0.1)


def task_3(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        time.sleep(0.1)


def main():
    g1 = gevent.spawn(task_1, 5)
    g2 = gevent.spawn(task_2, 5)
    g3 = gevent.spawn(task_3, 5)

    g1.join()
    g2.join()
    g3.join()


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
<Greenlet at 0x1433a62b370: task_1(5)> 0
<Greenlet at 0x1433a62b590: task_2(5)> 0
<Greenlet at 0x1433a62b480: task_3(5)> 0
<Greenlet at 0x1433a62b370: task_1(5)> 1
<Greenlet at 0x1433a62b590: task_2(5)> 1
<Greenlet at 0x1433a62b480: task_3(5)> 1
<Greenlet at 0x1433a62b370: task_1(5)> 2
<Greenlet at 0x1433a62b590: task_2(5)> 2
<Greenlet at 0x1433a62b480: task_3(5)> 2
<Greenlet at 0x1433a62b370: task_1(5)> 3
<Greenlet at 0x1433a62b590: task_2(5)> 3
<Greenlet at 0x1433a62b480: task_3(5)> 3
<Greenlet at 0x1433a62b370: task_1(5)> 4
<Greenlet at 0x1433a62b590: task_2(5)> 4
<Greenlet at 0x1433a62b480: task_3(5)> 4
```

### joinall

有没有发现

g1.join()
g2.join()
g3.join()

写了三个join
不如给他来个一次性

> `gevent.joinall(spawn_list, timeout=None, raise_error=False, count=None)`

```python
import time
import gevent
from gevent import monkey

monkey.patch_all()


def task_1(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        time.sleep(0.1)


def task_2(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        time.sleep(0.1)


def task_3(n):
    for i in range(n):
        print(gevent.getcurrent(), i)
        time.sleep(0.1)


def main():
    # joinall 接受一个 spawn 列表
    gevent.joinall([
        gevent.spawn(task_1, 5),
        gevent.spawn(task_2, 5),
        gevent.spawn(task_3, 5)
    ]， timeout=5)


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
<Greenlet at 0x12e8cba9d00: task_1(5)> 0
<Greenlet at 0x12e8cba9e10: task_2(5)> 0
<Greenlet at 0x12e8cba9bf0: task_3(5)> 0
<Greenlet at 0x12e8cba9d00: task_1(5)> 1
<Greenlet at 0x12e8cba9e10: task_2(5)> 1
<Greenlet at 0x12e8cba9bf0: task_3(5)> 1
<Greenlet at 0x12e8cba9d00: task_1(5)> 2
<Greenlet at 0x12e8cba9e10: task_2(5)> 2
<Greenlet at 0x12e8cba9bf0: task_3(5)> 2
<Greenlet at 0x12e8cba9d00: task_1(5)> 3
<Greenlet at 0x12e8cba9e10: task_2(5)> 3
<Greenlet at 0x12e8cba9bf0: task_3(5)> 3
<Greenlet at 0x12e8cba9d00: task_1(5)> 4
<Greenlet at 0x12e8cba9e10: task_2(5)> 4
<Greenlet at 0x12e8cba9bf0: task_3(5)> 4
```

只管在 joinall 里创建 spawn 对象就行，其他不用管，gevent 自己会处理好的
这也是最常用的方法