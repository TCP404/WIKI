# IO
Python的自带模块 `os` 可以进行许多与操作系统相关的操作。

例如 **查看当前系统类型**

```python
>>> import os
>>> os.name
'nt'
```

`nt`表示`windows`，`posix`表示`Linux/Unix/Mac OS X`

在`Linux/Unix/Max OS X`上想获得更详细的信息，可以使用`uname()`，不过在`windows`上不提供

```python
>>> import os
>>> os.uname()
posix.uname_result(sysname='Darwin', nodename='MichaelMacPro.local', release='14.3.0', version='Darwin Kernel Version 14.3.0: Mon Mar 23 11:59:05 PDT 2015; root:xnu-2782.20.48~5/RELEASE_X86_64', machine='x86_64')
```

再例如 **查看环境变量**，`os.environ`

```python
>>> import os
>>> os.environ
environ({'ALLUSERSPROFILE': 'C:\\ProgramData', 'APPDATA': 'C:\\Users\\pr919\\AppData\\Roaming', 'ASL.LOG': 'Destination=file', 'CATALINA_HOME': 'D:\\---Programming---\\IntelliJ IDEA\\apache-tomcat-8.5.42', ..., 'WT_SESSION': '245de0da-1ef4-4cda-93ca-c132289e2c9c'})
```

这样就列出了所有环境变量了

如果要获得某一条，可以用`os.environ.get('key')`

```python
>>> import os
>>> os.environ.get('JAVA_HOME')
'C:\\Program Files\\Java\\jdk1.8.0_201'
```


## 获得当前目录 getcwd

!!! note ""
    `os.getcwd()`

```python
>>> import os
>>> os.getcwd()
'C:\\Users\\pr919'
```

## 列出目录 listdir、walk

!!! note ""
    `os.listdir(path)`，单层列出；path可以是绝对路径或相对路径，不填默认当前目录
    
    `os.walk(path)`，遍历列出；

listdir() 列出path下一层的目录和文件，不包括子目录

```python
>>> import os
>>> os.listdir()
>>> os.listdir(r'D:\---Programming---\Python\Python')
['DLLs', 'Doc', 'include', 'Lib', 'libs', 'LICENSE.txt', 'NEWS.txt', 'python-3.8.3-amd64.exe', 'python.exe', 'python3.dll', 'python38.dll', 'pythonw.exe', 'Scripts', 'tcl', 'Tools', 'vcruntime140.dll', 'vcruntime140_1.dll']

>>> for i in os.listdir(r'D:\---Programming---\Python\Python'):
...     print(i)
...
DLLs
Doc
include
Lib
libs
LICENSE.txt
NEWS.txt
python-3.8.3-amd64.exe
python.exe
python3.dll
python38.dll
pythonw.exe
Scripts
tcl
Tools
vcruntime140.dll
vcruntime140_1.dll
```

walk() 列出path下的目录和文件，包括子目录
```python
import os

cwd = os.getcwd()
for i in os.walk(cwd):
    print(i, end='\n\n')

--------------------------------------------------

# Output:
('D:\\Project', 
 ['weather', '__pycache__'], 
 ['app.config', 'io.txt', 'mo.py', 'mo2.py', 'my.log', 'test.py'])

('D:\\Project\\weather',
 ['.idea', '__pycache__'],
 ['city_code.csv', 'city_code.txt', 'query.py', 'Ui_weather.py', 'weather.py', 'weather.ui'])

('D:\\Project\\weather\\.idea',
 ['inspectionProfiles'],
 ['misc.xml', 'modules.xml', 'weather.iml', 'workspace.xml'])

('D:\\Project\\weather\\.idea\\inspectionProfiles',
 [],
 ['Project_Default.xml'])

('D:\\Project\\weather\\__pycache__',
 [],
 ['query.cpython-37.pyc', 'Ui_weather.cpython-37.pyc'])

('D:\\Project\\__pycache__',
 [],
 ['mo.cpython-38.pyc', 'Person.cpython-38.pyc'])
```

`os.walk(path)`返回的是一个`生成器 generator`

每次返回一个元组，元组中共有三个元素：(root, dirs, files)

1. 第一个`root`是一个string，表示当前层的 **路径**
2. 第二个`dirs`是一个list，表示当前层 **拥有的目录**
3. 第三个`files`是一个list，表示当前层 **拥有的文件**


## 创建目录 mkdir、makedirs
!!! note ""
    `os.mkdir(path)`，**创建单层目录**；path可以是相对路径或绝对路径

    如果目录已存在，会抛出`FileExistsError`错误。

```python
# 标准创建目录流程
dirpath = 'iodir'
if not os.path.exists(dirpath):
    os.mkdir(dirpath)
```

!!! note ""
    `os.makedirs(dir/subdir)`，**创建多层目录**；

    如果目录已存在，会抛出`FileExistsError`错误。

```python
# 创建多层目录
dirpath = 'iodir/subiodir'
if not os.path.exists(dirpath):
    os.makedirs(dirpath)
```




## 删除目录 rmdir、shutil.rmtree

!!! note ""
    
    `os.rmdir(path)`，**删除空目录**；path可以说相对路径或绝对路径
    
    如果目录不存在，会抛出`FileNotFoundError`错误。
    
    如果目录不为空，会抛出`OSError`错误。

```python
# 标准删除目录流程
import os

dirpath = 'iodir'
if os.path.exists(dirpath):
    os.rmdir(dirpath)

--------------------------------------------------

# Output:
没任何输出，删除目录成功

目录下不为空，输出：
Traceback (most recent call last):
  ...
OSError: [WinError 145] 目录不是空的。: 'iodir'
```

!!! note ""
    `shutil,rmtree(path)`，**删除非空目录**；

    如果目录不存在，会抛出`FileNotFoundError`错误。

`iodir`下有一个`io.txt`文件，执行`shuitl.rmtree()`如下

```python
import os, shuitl

dirpath = 'iodir'
if os.path.exists(dirpath):
    shuitl.rmtree(dirpath)

--------------------------------------------------

# Output:
没任何输出，删除目录成功

目录不存在，输出：
Traceback (most recent call last):
  ...
FileNotFoundError: [WinError 3] 系统找不到指定的路径。: 'iodir'
```

## 删除文件

!!! note ""
    `os.remove(filename)`； **删除单个文件**

```python
import os

filename = 'io.txt'
if os.path.exists(filename):
    os.remove(filename)
```


## 路径拼接 join
!!! note ""
    `os.path.join()`

不用的操作系统路径分隔符是不同的

例如在`windows`下是`\`，在`Linux/Unix/Mac OS X`下是`/`

所以保险起见，在拼接路径的时候用os提供的`os.path.join()`函数。

这样在不同操作系统中运行代码的时候都可以得到正确的路径拼接。

```python
import os

dir = os.path.join('gpdir', 'padir', 'subdir')
print(dir)

--------------------------------------------------

# Output in windows:
gpdir\padir\subdir

# Output in Linux/Unix
gpdir/padir/subdir
```

## 路径拆分 split
!!! note ""
    `os.path.split()`

返回值是一个 **双元素元组**，第一个元素是路径的前部分，第二个元素是最后级别的目录或文件名

```python
>>> os.path.split('gpdir\padir\subdir')
('gpdir\\padir', 'subdir')

>>> os.path.split('gpdir/padir/subdir')
('gpdir/padir', 'subdir')
```

## 获取文件扩展名 splitext
!!! note ""
    `os.path.splitext()`

返回值是一个 **双元素元组**，第一个元素是路径前部分，第二个元素是最后级别文件的扩展名

```python
>>> os.path.splitext('gpdir/padir/subdir/io.txt')
('gpdir/padir/subdir/io', '.txt')

>>> os.path.splitext('gpdir/padir/subdir')
('gpdir/padir/subdir', '')

>>> os.path.splitext('gpdir\padir\subdir\io.txt')
('gpdir\\padir\\subdir\\io', '.txt')

>>> os.path.splitext('gpdir\padir\subdir')
('gpdir\\padir\\subdir', '')
```

## 文件、目录重命名
!!! note ""
    `os.rename(oldName, newName)`

可以对文件或目录进行重命名

```python
>>> .rename('testdir', 'newtestdir')

>>> os.rename('test.txt', 'iotest.py')
```

## 判断是目录还是文件

!!! note ""
    `os.path.isdir(path)`

    `os.path.isfile(path)`


## 示例
列出当前目录下所有目录

```python
>>> [x for x in os.listdir() if os.path.isdir(x)]
['.lein', '.local', '.m2', '.npm', '.ssh', '.vim', 'Applications', 'Desktop', ...]
```

列出当前目录下所有`.py`文件

```python
>>> [x for x in os.listdir() if os.path.isfile(x) and os.path.splitext(x)[1] == '.py']
['apis.py', 'config.py', 'models.py', 'test_db.py', 'urls.py', 'wsgiapp.py']
```