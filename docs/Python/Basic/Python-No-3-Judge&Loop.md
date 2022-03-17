---
title: Python【No-3】判断和循环
tags: Python
categories:
  - Python
  - 基础
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 6930
date: 2020-07-04 20:41:48
---

程序结构：判断与循环

<!--more-->


# Judgment&circulation

!!! warning
    **！！不要用浮点数比较相等！！**


## 判断

```python
if condition:
    print()
elif condition:
    print()
else:
    print()
```

例如
```python
age = 20

if age > = 6:
    print('teenager')
elif age >= 18:
    print('adult')
else:
    print('kid')
```

还可以简写
```python
if x:
    print('True')

或

if x: print('True')
```
**只要 x 是非零数值、非空字符串、非空list、非空tuple，也不是None，就判断为True，否则为False**

## 循环

### for
```python
for x in Iterable:
   do sth
```

例如
```python
Li = [1, 2, 3, 4, 5, 6]
sum = 0
for x in Li:
    sum += x
print(sum)

# 21
```
```python
sum = 0
for x in [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]:
    sum = sum + x
print(sum)

#55
```


### range()

```python
>>> range(5)
0, 1, 2, 3, 4
>>> list(range(5))
[0, 1, 2, 3, 4]
```
```python
# 从1加到100

sum = 0
for x in range(101):
    sum = sum + x
print(sum)

# 5050
```

### while

```python
while condition:
    do sth
```

```python
sum = 0
n = 99

while n > 0:
    sum = sum + n
    n = n - 2
print(sum)
```

### break
> 跳出整个循环，且不执行else子句

### continue
> 跳出本次循环

### else
> else 子句会在循环不满足条件退出时执行
> 但如果 break 跳出循环时，不会执行else子句

```python
sum = 0
n = 99

while n > 0:
    sum = sum + n
    n = n - 2
else:
    print('else')    # 循环结束后会执行这句话
print(sum)

---------------------------------

# Output:
else
2500
```

```python
sum = 0
n = 99

while n > 0:
    sum = sum + n
    n = n - 2
    if n < 10:
        break   # n 小于 10的时候会跳出循环，并且不会执行 else 子句
else:
    print('else')
print(sum)

---------------------------------

# Output:
2475
```

### pass
> pass 语句什么也不做。当语法上需要一个语句，但程序需要什么动作也不做时，可以使用它。

```python
>>> while True:
...     pass  # Busy-wait for keyboard interrupt (Ctrl+C)

>>> class MyEmptyClass:
...     pass

>>> def initlog(*args):
...     pass   # Remember to implement this!
```
