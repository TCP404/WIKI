# 枚举类型
## 创建枚举

有两种方法创建枚举

1. 基于 class 语法创建
2. 基于 Function API 创建

第一步，导入Enum类

```python
from enum import Enum
```

### 基于 class 语法创建

```python
from enum import Enum

class Weekend(Enum)；
    Mon = 1
    Tue = 2
    Wed = 3
    Thr = 4
    Fri = 5
    Sat = 6
    Sun = 7
```
上面例子

1. 定义了 Weekend 枚举类型
2. 定义了 Weekend 的枚举成员 Weekend.Mon，Weekend.Tue...
3. 为每一个枚举成员赋值。如Weekend.Mon 的值为1。值可以指定为其他类型，不是必须整型

枚举成员包含两个属性：`name`和`value`

```python
>>> Weekend.Mon.name
Mon
>>> Weekend.Mon.value
1
```

定义string类型的值

```python
from enum import Enum

class Weekend(Enum)；
    Monday = 'Mon'
    Tuesday = 'Tue'
    Wednesday = 'Wed'
    Thursday = 'Thu'
    Friday = 'Fri'
    Saturday = 'Sat'
    Sunday = 'Sun'

>>> Weekend.Monday.value
'Mon'
```

### 基于 Function API 创建

!!! Abstract
    `Enum(enum name, enumerators)`
    
    第一个参数 enum name 表示枚举名称，第二个参数enumerators 表示枚举成员列表
    
    枚举成员列表有三种方式：

    1. 使用字符串表示，各成员名使用空格隔开。成员的值从1开始自动递增

        `Enum('enum_name', 'member1 member2 member3 ... memberN')`

    2. 使用元组表示，成员的值从1开始自动递增

        `Enum('enum_name', ('member1', 'member2', 'member3', ... , 'memberN'))`
    
    3. 使用字典表示，字典可以指定枚举成员的值，其中字典的键为枚举成员名，值为枚举成员的值

        `Enum('enum_name', {'member1_key': memberl_value, 'member_key2': member2_value,...})`

```python
from enum import Enum

# 以下三句表达式互相等价
weekend = Enum('week','Mon Tue Wed Thu Fri Sat Sun')
weekend = Enum('week',('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun')
weekend = Enum('week',{'Mon':1, 'Tue':2, 'Wed':3, 'Thu':4, 'Fri':5, 'Sat':6, 'Sun':7})

>>> weekend.Mon.value
1
```

## 访问枚举成员

访问枚举成员有三种方式：

1. 使用点号(".")引用
2. 使用value获取，值对应的枚举成员
3. 使用枚举成员名


```python
from enum import Enum

class Weekend(Enum):
    Mon = 1
    Tue = 2
    Wed = 3
    Thr = 4
    Fri = 5
    Sat = 6
    Sun = 7

# 按值访问
print(Weekend(5))        # Weekend.Fri

# 按枚举名访问
print(Weekend['Sun'])    # Weekend.Sun

# 访问成员的名称
print(Weekend.Tue.name)  # Tue

# 访问成员的值
print(Weekend.Thr.value) # 4
```

## 枚举遍历

如果把枚举当作 Dict 来看，`枚举类.枚举成员名`是key，`赋给枚举成员的值`是value


```python
print("name:    member   | value")
print("-" * 25)

for name, member in Weekend.__members__.items():
    print(name + " : " + str(member) + " | " + str(member.value))


# Output:

name:    member   | value
-------------------------
Mon : Weekend.Mon | 0
Tue : Weekend.Tue | 1
Wed : Weekend.Wed | 2
Thr : Weekend.Thr | 3
Fri : Weekend.Fri | 4
Sat : Weekend.Sat | 5
Sun : Weekend.Sun | 6



>>> print(Weekend.__members__)
{'Mon': <Weekend.Mon: 0>,
 'Tue': <Weekend.Tue: 1>,
 'Wed': <Weekend.Wed: 2>,
 'Thr': <Weekend.Thr: 3>,
 'Fri': <Weekend.Fri: 4>,
 'Sat': <Weekend.Sat: 5>,
 'Sun': <Weekend.Sun: 6>}


>>> type(Weekend.__members__)
<class 'mappingproxy'>
```


## 枚举比较

枚举成员并非整型，而是一种映射类型，是不能做大小比较的

当时可以做相等比较

```python
>>> Weekend.Mon == Weekend.Mon
True
>>> Weekend.Mon == Weekend.Tue
False
>>> Weekend.Mon != Weekend.Tue
True
>>> Weekend.Mon == 0
False
```


## 限定枚举唯一性
限定枚举唯一性是指 限制枚举类中的枚举成员 的 名称和值都不重复

限定枚举唯一性非常简单

导入 unique类，然后在自定义的枚举类前加上装饰器 `@unique`

```python
from enum import Enum, unique

@unique
class Weekend:
    Mon = 0
    Tue = 1
    Wed = 2
    Thr = 3
    Fri = 4
    Sat = 5
    Sun = 6
```