---
title: Python【Buildins】json
tags:
  - Python
  - json
categories:
  - Python
  - 内置库
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 41814
date: 2020-08-22 12:17:48
---

Python的内置库——json处理

<!--more-->

> `{"firstName":"John", "lastName":"Doe"}`

> 对于dictionaries，keys需要是字符串类型(字典中任何非字符串类型的key在编码时会先转换为字符串)。而且，在web应用程序中，顶层对象被编码为一个字典是一个标准做法。

这句话大概意思是：

1. 用`{}` 开头和结尾
2. 用`str`表示 key
3. 用`str, int, list, dict` 表示 value
4. list用`[index]`访问
5. dict用`.key`访问

json 字符串生成 python 对象：`json.load()`
python 对象格式化为 json 字符串：`json.dump()`


## 数据类型
转换对应表（python -> json）

| Python      | JSON   |
| :---------- | :----- |
| dict        | object |
| list, tuple | array  |
| str         | string |
| int, float  | number |
| True        | true   |
| False       | false  |
| None        | null   |

转换对应表（json -> python）

| JSON         | Python |
| :----------- | :----- |
| object       | dict   |
| array        | list   |
| string       | str    |
| number(int)  | int    |
| number(real) | float  |
| true         | True   |
| false        | False  |
| null         | None   |

## 常用方法

| 方法              | 功能                                           | 总结             |
| :---------------- | :--------------------------------------------- | :--------------- |
| json.dump(obj,fp) | 将python数据类型转换并保存到json格式文件内     | p -> j,写入文件  |
| json.dumps(obj)   | 将python数据类型转换为json格式的字符串         | p -> j           |
| json.load(fp)     | 从json格式的文件中读取数据并转换为python的类型 | 从文件读, j -> p |
| json.loads(str)   | 将json格式的字符串转换为python的类型           | j -> p           |

带s的是处理字符串
不带s的是从文件里处理

![json](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Py/Buildins-json.png)

### Python -> JSON

> `json.dump(obj, fp, *, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, default=None, sort_keys=False, **kw)`

> `json.dumps(obj, *, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, default=None, sort_keys=False, **kw)`

- `obj`：需要被转换的Python对象
- `fp`：文件对象，要写入的文件
- `skipkeys`：
    - Fasle时：不是基本对象(int、str、bool、float)的键不会被跳过
    - True时：不是基本对象(int、str、bool、float)的键会被跳过
- `ensure_ascii`：
    - True：将非ASCII字符转义，例如中文
    - False：将非ASCII字符原样输出
- `allow_nan`：
    - True：对json规格范围外的float类型(nan、inf、-inf)序列化时会引发`ValueError`
    - False：对json规格范围外的float类型(nan、inf、-inf)序列化时使用等价形式(NaN、Infinity、-Infinity)
- `indent`：指定缩进等级（非负整数/字符串）
    - 0、负数、""：只添加换行符
    - None：不缩进
    - 正整数：每层缩进同样数量的空格
    - 字符串：如`\t`，该字符串将被用于缩进每一层
- `sort_keys`：按字典的键排序
- `default`：其应该是一个函数，每当某个对象无法被序列化时它会被调用。它应该返回该对象的一个可以被 JSON 编码的版本或者引发一个 `TypeError`。

1. 使用`dumps()`

```python
import json

# 这是一个 python 字典对象
py_dic = {
    'firstName': 'Boii',
    'j': {
        'x': 1,
        'y': "2",
        'z': 3
    },
    "lastName": "Pro",
    'li': ["A", 'b', 3]
}

print('Python对象：', end="")
print(py_dic, end="\t")
print(type(py_dic))

# 将一个 Python 对象转换成 json 字符串
json_str = json.dumps(py_dic, sort_keys=True)

print('JSON字符串：', end="")
print(json_str, end="\t")
print(type(json_str))

--------------------------------------------------

# Output:
Python对象：{'firstName': 'Boii', 'j': {'x': 1, 'y': '2', 'z': 3}, 'lastName': 'Pro', 'li': ['A', 'b', 3]}    <class 'dict'>
JSON字符串：{"firstName": "Boii", "j": {"x": 1, "y": "2", "z": 3}, "lastName": "Pro", "li": ["A", "b", 3]}    <class 'str'>
```
可以发现，Python中都是使用单引号`'`，JSON中都是使用双引号`"`

```python
import json

# 这是一个python 列表对象
py_list = [1, 2, 3]

print('Python对象：', end="")
print(py_list, end="\t")
print(type(py_list))

# 将一个 Python 对象转换成 json 字符串
json_str = json.dumps(py_list, sort_keys=True)

print('JSON字符串：', end="")
print(json_str, end="\t")
print(type(json_str))

--------------------------------------------------

# Output:
Python对象：[1, 2, 3]   <class 'list'>
JSON字符串：[1, 2, 3]   <class 'str'>
```

2. 使用`dump()`

```python
import json

py_dic = {
    'firstName': 'Boii',
    'j': {
        'x': 1,
        'y': "2",
        'z': 3
    },
    "lastName": "Pro",
    'li': ["A", 'b', 3]
}

# 将一个 Python 对象转换成 json 字符串，然后写入文件
with open('./py_dic.json', 'w', encoding='utf-8') as f:
    json.dump(py_dic, f, sort_keys=True)

--------------------------------------------------

# Output:
没有输出，因为输出到文件中了

# py_dic.json
{"firstName": "Boii", "j": {"x": 1, "y": "2", "z": 3}, "lastName": "Pro", "li": ["A", "b", 3]}
```
有些教程中直接使用`open()`打开一个文件对象，然后将文件对象作为参数传入`dump()`，却没有使用`close()`关闭文件对象。这是错误的，会造成文件对象一直没有关闭而持续占用计算机资源。

**解决方法**：要么手动调用`close()`方法，要么用`with语句`

```python
f = open('./py_dic.json', 'w', encoding='utf-8')
json.dump(py_dic, f, sort_keys=True)
f.close()

或

with open('./py_dic.json', 'w', encoding='utf-8') as f:
    json.dump(py_dic, f, sort_keys=True)
```

### JSON -> Python
> `json.load(fp, *, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)`

> `json.loads(s, *, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)`

- `fp`：文件对象，要写入的文件
- `object_hook`：是一个可选的函数，它会被调用于每一个解码出的对象字面量（即一个 dict）。object_hook 的返回值会取代原本的 dict。这一特性能够被用于实现自定义解码器（如 JSON-RPC 的类型提示)。
- `parse_float`、`parse_int`、`parse_constant`、 `object_pairs_hook`：都是编码解码器需要的参数


1. 使用`loads()`

```python
import json

Str = '{"name": "Boii", "age": 30, "tel":["13800000000", "13100880088"], "isonly":true}'

print('JSON字符串：', end="")
print(Str, end='\t')
print(type(Str))

# 将一个 json 字符串 转换成 Python 对象
pythonObj = json.loads(Str)

print('Python对象：', end="")
print(pythonObj, end='\t')
print(type(pythonObj))
--------------------------------------------------

# Output:

JSON字符串：{"name": "Boii", "age": 30, "tel":["13800000000", "13100880088"], "isonly":true}    <class 'str'>
Python对象：{'name': 'Boii', 'age': 30, 'tel': ['13800000000', '13100880088'], 'isonly': True}    <class 'dict'>
```
可以发现，Python中都是使用单引号`'`，JSON中都是使用双引号`"`

2. 使用`load()`从json文件中读取转换成Python对象

```python
# py.json
{"firstName": "Boii", "j": {"x": 2, "y": "2", "z": 3}, "lastName": "Pro", "li": ["A", "b", 3]}

# j2p.py
import json

with open('./j2p.py', 'r', encoding='utf-8') as f:
    pythonObj = json.load(f)
    print('Python对象：', end="")
    print(pythonObj)
    print(type(pythonObj))

--------------------------------------------------

# Output:
Python对象：{'firstName': 'Boii', 'j': {'x': 1, 'y': '2', 'z': 3}, 'lastName': 'Pro', 'li': ['A', 'b', 3]}
<class 'dict'>
```

## 总结
```python
# Writing JSON data
with open('data.json', 'w') as f:
    json.dump(data, f)

# Reading data back
with open('data.json', 'r') as f:
    data = json.load(f)
```