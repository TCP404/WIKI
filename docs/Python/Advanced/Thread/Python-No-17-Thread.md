---
title: Python【No-17】线程
tags: Python
categories:
  - Python
  - 进阶
  - 进程、线程
thumbnail: 'https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/thumbnail/python.png'
date: 2020-07-18 13:41:48
---

线程，是操作系统调度的单位

<!--more-->

一个程序写完了，放在硬盘里不运行，叫做 **程序**
一个程序写完了，放到内存里跑起来，叫做 **进程**
一个进程中至少有一个 **线程** ，称为 **主线程**
一个线程要干很多事情，有些事情很耗时，就创建个 **子线程** ，让子线程去做，主线程继续下一步

当有多个线程的时候，CPU会轮流切换执行，每个线程执行多长时间叫做 **时间片**
比如 t1 分到了0.001秒的时间片，t2 分到了0.002秒的时间片
显然这么短时间是不够执行的，所以CPU会快速的切来切去，这样就看起来好像能执行 **多任务**
多任务就是你可以同时听歌、写文档、挂QQ等等，但其实同一时刻CPU只执行一个线程
只不过切换的快，时间片也很小，所以人感觉不出来。

线程的执行顺序是由操作系统决定的，开发者左右不了。

> 子线程由主线程创建，子线程关闭后留下的辣鸡由主线程清理
> 一旦主线程挂了，子线程全部挂掉。
> 主线程会等待所有子线程结束后才结束

单线程操作
```python
import time


def say_hi():
    for i in range(5):
        print('Hi!')
        time.sleep(1)    # 耗时操作


def say_yes():
    for i in range(5):
        print('Yes!')
        time.sleep(1)    # 耗时操作


def main():
    say_hi()
    say_yes()


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
Hi!
Hi!
Hi!
Hi!
Hi!
Yes!
Yes!
Yes!
Yes!
Yes!
```

多线程操作
```python
import time
import threading    # 导入 threading 模块


def say_hi():
    for i in range(5):
        print('Hi!')
        time.sleep(1)    # 耗时操作


def say_yes():
    for i in range(5):
        print('Yes!')
        time.sleep(1)    # 耗时操作


def main():
    t1 = threading.Thread(target=say_hi)    # 创建一个线程对象
    t2 = threading.Thread(target=say_yes)   # 创建一个线程对象
    t1.start()    # 启动线程
    t2.start()    # 启动线程


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
Hi!
Yes!
Hi!
Yes!
Hi!
Yes!
Yes!
Hi!
Hi!
Yes!
```

单线程和多线程的区别就像这个栗子

你要来一段热舞rap，
- 单线程的操作是：rap完再热舞，或者热舞完再rap
- 多线程的操作是：边热舞边rap

## 创建子线程并启动
创建线程操作可以分为三步：

1. 导入 threading 模块
2. 创建一个Thread对象
3. 通过Thread对象启动线程

创建一个Thread对象的时候可以传入参数
`threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)`

- group 应该为 None；为了日后扩展 ThreadGroup 类实现而保留。
- target：**线程中需要执行的函数** 。默认是 None，表示不需要调用任何方法。
- name：**线程名称** 。默认情况下，由 "Thread-N" 格式构成一个唯一的名称，其中 N 是小的十进制数。
- args：用于调用目标函数的 **参数元组** 。默认是 ()。`（必须是可迭代对象）`
- kwargs：用于调用目标函数的关键字 **参数字典** 。默认是 {}。`（必须是可迭代对象）`
- `daemon不是None`，线程将被显式的设置为`守护模式`，不管该线程是否是守护模式。
- `daemon=None`，线程将继承当前线程的守护模式属性。
- 如果子类重载了构造函数，必须先调用父类构造器`Thread.__init__()`，再做其他事情。

### 函数方式
函数方式适合于 —— 一个线程中要做的事情比较少，一个函数就能搞定。

1. 导入 threading 模块
2. 创建一个Thread对象并把函数传进去
3. 通过Thread对象启动线程

```python
import threading    # 1. 导入 thrading 模块


def funcName():     # 需要子线程执行的函数
    pass


t1 = threading.Thread(target=funcName)    # 2. 创建一个Thread对象并把函数传进去
t1.start()    # 3. 启动线程
```

### 继承方式
继承方式适合于 —— 一个线程里面做的事情比较复杂，需要分成多个函数来做。

1. 导入 threading 模块
2. 创建一个类继承 Thread 类
3. 重写 run 方法
4. 创建一个继承了 Thread 类的对象
5. 用start方法启动线程

```python
import threading    # 1. 导入 threading 模块


class clsName(threading.Thread):    # 2. 继承Thread类
    def run(self):                  # 3. 重写run方法
        print(self.name)    # self.name 保存的是当前线程的名称


t1 = clsName()    # 4. 创建一个继承了 Thread 类的对象
t1.start()        # 5. 启动线程
```

`t1.start()`只会去调用`run`方法。如果类中还有其他方法，只能在类中调用，不能用 `t1`去调用。
也就是说，继承了 Thread 的类的之类，除了run方法，其他方法都可以定义成私有的（公有的外面也不能调用啊）

```python
import threading


class MyThread(threading.Thread):
    def run(self):
        print(self.name)
        self.__say_hi()
        self.__say_yes()

    def __say_hi(self):
        print('Hi!')

    def __say_yes(self):
        print('Yes!')


t1 = MyThread()
t1.start()

--------------------------------------------------

# Output:
Thread-1
Hi!
Yes!
```

## 插队 join()
> `join([timeout])`
> 当某个线程对象成功调用了 join() 方法之后，会优先获得CPU的资源，让其他线程等待
> 知道这个线程对象执行结束后，才把资源让给别的线程

```python
import threading


def f1(n):
    for i in range(n):
        print(threading.current_thread().getName())


def f2(n):
    for i in range(n):
        print(threading.current_thread().getName() + "------" + str(i))


def main():
    t1 = threading.Thread(target=f1, args=(5,))
    t2 = threading.Thread(target=f2, args=(5,))
    t1.start()
    t2.start()

    for i in range(5):
        print(threading.current_thread().getName())


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
Thread-1
Thread-1
MainThread
Thread-2------0
MainThread
Thread-2------1
MainThread
Thread-2------2
MainThread
MainThread
Thread-2------3
Thread-2------4
Thread-1
Thread-1
Thread-1
```
可以看到，没有加 join 方法，所以主线程 和 t1 和 t2 相互争抢资源。
如果现在需要 t1 这个线程执行完再给其他线程执行，也就是说要求 t1 独占一段时间
那就让 t1 调用 join() 方法

```python
import threading


def f1(n):
    for i in range(n):
        print(threading.current_thread().getName())


def f2(n):
    for i in range(n):
        print(threading.current_thread().getName() + "------" + str(i))


def main():
    t1 = threading.Thread(target=f1, args=(5,))
    t2 = threading.Thread(target=f2, args=(5,))
    t1.start()
    t1.join()
    t2.start()

    for i in range(5):
        print(threading.current_thread().getName())


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
Thread-1
Thread-1
Thread-1
Thread-1
Thread-1
Thread-2------0
Thread-2------1
MainThread
Thread-2------2
MainThread
Thread-2------3
MainThread
Thread-2------4
MainThread
MainThread
```
这样就实现了 t1 独占资源，完成后才给其他线程



## 查看所有线程
> `threading.enumerate()`
> 以列表形式返回当前所有存活的 Thread 对象。
> 该列表包含守护线程，current_thread() 创建的虚拟线程对象和主线程。
> 它不包含已终结的线程和尚未开始的线程。

```python
import time
import threading


def say_hi():
    for i in range(3):
        print('Hi!')
        time.sleep(1)


def say_yes():
    for i in range(3):
        print('Yes!')
        time.sleep(1)


def main():
    t1 = threading.Thread(target=say_hi)
    t2 = threading.Thread(target=say_yes)

    t1.start()
    t2.start()

    while True:
        print(threading.enumerate())
        if len(threading.enumerate()) <= 1:
            break
        time.sleep(1)


if __name__ == '__main__':
    main()

--------------------------------------------------

# Output:
Hi!
Yes!
[<_MainThread(MainThread, started 1292)>, <Thread(Thread-1, started 7968)>, <Thread(Thread-2, started 3796)>]
[<_MainThread(MainThread, started 1292)>, <Thread(Thread-1, started 7968)>, <Thread(Thread-2, started 3796)>]
Hi!
Yes!
Hi!
Yes!
[<_MainThread(MainThread, started 1292)>, <Thread(Thread-1, started 7968)>, <Thread(Thread-2, started 3796)>]
[<_MainThread(MainThread, started 1292)>, <Thread(Thread-1, started 7968)>]
```

## 多线程共享全局变量
假设现在有个任务，去抓取某个网站上所有的照片，然后发送到某个邮箱。
这里就可以分成两个子任务，一个抓取照片，一个发送照片
于是我们可以创建一个线程负责抓取照片，一个线程负责发送照片

那么可以发现，`照片`是两个线程都要用到的数据。
所以我们应该把`照片`设置为 **全局变量** ，这样两个线程才可以进行合作。

> 在多线程任务中，`全局变量`是可以给每一个线程共享的。

```python
import threading

g_num = 10    # 全局变量


def add1():
    global g_num
    g_num += 5
    print("add1----------- %d" % g_num)


def add2():
    print("add2----------- %d" % g_num)


def main():
    t1 = threading.Thread(target=add1, name='add1')
    t2 = threading.Thread(target=add2, name='add2')

    t1.start()
    t2.start()
    print("main----------- %d" % g_num)


if __name__ == "__main__":
    main()

--------------------------------------------------

# Output:
add1----------- 15
add2----------- 15
main----------- 15
```
从栗子中可以看出，线程`t1`中的`add1`对全局变量`g_num`进行了修改
在线程`t2`的`add2`对`g_num`进行访问的时候访问到的是修改后的值
在主线程的`main`方法中访问也是修改后的值。

所以：全局变量对于任何一个线程，包括主线程，来说都是共享的。

## 锁
全局变量对于任何一个线程来说都是共享的，那就一定会引发争抢资源的情况，这是由CPU的运行机制导致的。
如果对同一个全局变量，两个线程一个写一个读倒还好，但是如果两个同时写就会引发各种问题。
特别是在银行、金融领域，会引发更大的问题。比如一边转了帐还没收到突然出现问题，转账的扣钱了，收账的没收到，那就凉凉了~

所以为了解决这种缺陷，引入了锁的概念。
举个栗子，很多人上一个厕所，前面的进去了得锁门吧？办完事了开锁出来后面的才能进去，上锁，办事儿。
锁就是这样的概念。哪个线程先抢到锁哪个就先执行，执行完了释放锁，下一个线程抢到了就锁上.....
这叫做 **同步控制**



互斥锁
> 创建锁：`mutex = threading.Lock()`，默认没有上锁的  
> 锁定：`mutex.acquire(blocking=True, timeout=1)`
> 解锁：`mutex.release()`

1. `acquire(blocking=True, timeout=1)`    成功获得锁返回True，否则返回Flase
- blocking：设置为`True`时，会一直等到前面的锁释放然后锁定，设置为`Flase`则不会。
- timeout：浮点型，单位——秒；
    - 值为正数时，等待时长为设定的秒数
    - 值为-1时，将无限等待
    - `blocking=Flase`时，timeout不起作用。

2. `release()`    锁被锁定时，重置为未锁定。锁没被锁定时，引发`RuntimeError`错误。

```python
import threading
import time

g_num = 0
mutex = threading.Lock()


def add1(num):
    global g_num
    mutex.acquire()
    for i in range(num):
        g_num += 1
    mutex.release()
    print("add1----------- %d" % g_num)


def add2(num):
     global g_num
    mutex.acquire()
    for i in range(num):
        g_num += 1
    mutex.release()
    print("add2----------- %d" % g_num)


def main():
    t1 = threading.Thread(target=add1, args=(1_000_000, ))
    t2 = threading.Thread(target=add2, args=(1_000_000, ))

    t1.start()
    t2.start()
    time.sleep(2)
    print("main----------- %d" % g_num)


if __name__ == "__main__":
    main()

--------------------------------------------------

# Output:
add1----------- 1000000
add2----------- 2000000
main----------- 2000000
```

### 死锁
锁并非只有一个，可以有多个。有多个锁，就可能发生死锁。

举个栗子，
电影里经常能看见的镜头——俩人打架不分上下，A锁住了B的脚踝，B锁住了A的脖子。
A：你放开我
B：你先放开我
A：你放开我我就放开你
B：你先放开我，我再放开你
A：凭什么我先放
B：那我也不放
...2000 year later...
两人死了。
The end.

这就是死锁，互不相让，互相锁了对方需要的资源，都等着对方先释放

线程A上了A锁干活，线程B上了B锁，这是两把锁，所以相安无事。
运行着运行着线程A需要一份资源，这份资源被线程B上了锁用着呢；线程B也需要一份资源，这份资源被线程A上了锁用着呢。
这时候线程A和线程B都在等对方释放锁，就这么一直等一直等

```python
import time
import threading

class ThreadLockA(theading.Thread):
    def run(self):
        mutexA.acquire()    # 使用A锁上锁

        print(self.name + '-----------doA--up--')
        time.sleep(1)

        # 这里会上锁失败，因为上面 time.sleep(1)的时候B锁已经被上锁了
        # 或者此线程后执行，A锁已经被上锁了，会上锁失败。
        # 所以死锁会发生在这
        mutexB.acquire()    # 请求B锁上锁
        print(self.name + '-----------doA--down--')
        mutexB.release()

        mutexA.release()    # 对A锁进行解锁

class ThreadLockB(threading.Thread)；
    def run(self):
        mutexB.acquire()    # 使用B锁上锁

        print(self.name + '-----------doB--up--')
        time.sleep(1)

        # 这里会上锁失败，因为上面 time.sleep(1)的时候A锁已经被上锁了
        # 或者此线程后执行，A锁已经被上锁了，会上锁失败。
        # 所以死锁会发生在这
        mutexA.acquire()
        print(self.name + '-----------doB--down--')
        mutexA.release()

        mutexB.release()    # 对B锁进行解锁

mutexA = threading.Lock()
mutexB = threading.Lock()

if __name__ == '__main__':
    t1 = ThreadLockA()
    t2 = ThreadLockB()
    t1.start()
    t2.start()

--------------------------------------------------

# Output:
-----------doA--up--
-----------doB--up--

···
这里已经卡死了
```

为了避免这种情况，又两种方法
1. 程序设计时要尽量避免（使用银行家算法）
2. 添加超时时间等