# 文件
同步和异步的区别就在于是否等待IO执行的结果。好比你去麦当劳点餐，
你说“来个汉堡”，服务员告诉你，对不起，汉堡要现做，需要等5分钟，于是你站在收银台前面等了5分钟，拿到汉堡再去逛商场，这是 **同步IO**。

你说“来个汉堡”，服务员告诉你，汉堡需要等5分钟，你可以先去逛商场，等做好了，我们再通知你，这样你可以立刻去干别的事情（逛商场），这是 **异步IO**。

很明显，使用异步IO来编写程序性能会远远高于同步IO，但是异步IO的缺点是编程模型复杂。想想看，你得知道什么时候通知你“汉堡做好了”，而通知你的方法也各不相同。如果是服务员跑过来找到你，这是 **回调模式**，如果服务员发短信通知你，你就得不停地检查手机，这是 **轮询模式**。总之，异步IO的复杂度远远高于同步IO。

文件读写其实都是由操作系统完成的，现代操作系统是不允许普通程序直接操作磁盘的，所以读写文件就是请求操作系统打开一个 `文件对象`（又称：文件描述符），通过操作系统提供的接口从这个 `文件对象` 中读取数据或写入数据

## 同步IO


```python
>>> with open(file_path, 'r/w/a') as alias:
···     do sth
```

- `r`  ：读
- `w`  ：覆盖写
- `a+`：追加写

### 读 read
读取的过程：1. 打开文件；2. 读取文件；3. 关闭文件

- 第一步：`f = open('/user/boii/io_note.txt', 'r')`
    用python内置的`open()`函数打开一个文件对象，
    第一个参数传入读的文件的路径，第二个参数传入`r`为读取的意思
    如果打开成功，`open()`会返回一个`文件对象`；
    如果打开失败，`open()`会抛出一个`IOError`错误。

- 第二步：`f.read()`
    通过`read()`方法，可以将全部内容读取出来。

- 第三步：`f.close()`
    文件打开成功后，必须使用`close()`方法关闭文件。
    因为文件对象会占用操作系统资源，且操作系统同一时间能打开的文件数量也有限。
    文件打开失败后，会产生`IOError`，则不会调用`f.close()`

综合起来可以这么写：
```python
try:
    f = open('/user/boii/io_note.txt', 'r')    # 打开文件
    print(f.read())    # 读取文件
except IOError as e:
    print(e)
else:
    f.close()    # 关闭文件

--------------------------------------------------

# Output:
# 如果文件打开成功，会输出（文件全部内容）：io
# 如果文件打开失败，会输出：[Errno 2] No such file or directory: '/user/boii/io_note.txt'
```

但是这样太繁琐，一点也不符合Python优雅的气质。
所以 Python 引入了`with`语句来自动帮我们调用`close()`方法

```python
with open('/user/boii/io_note.txt', 'r') as f:
    print(f.read())
```

#### 指定字节 read(size)

!!! note ""
    `file_obj.read(size)`

`read()`方法不填写参数，直接读取文件全部内容。如果一个文件太大，全部读取会爆内存的。

所以可以通过指定 size 的方式指定读取 size 个字节的内容。

```python
# io.txt
Hello Python!
I am Boii.
I am learning.

--------------------------------------------------

# io_read.py
with open('io.txt', 'r') as f:
    data = f.read(8)    # 读取前8个字节
    print(data)

--------------------------------------------------

# Output:
Hello Py
```

#### 跳过字节 seek(size)
    
!!! note ""
    `file_obj.seek(size)`

通过 `seek()`可以在读取之前跳过 size 个字节，再开始读取

```python
# io.txt
Hello Python!
I am Boii.
I am learning.

--------------------------------------------------

# io_read.py
with open('io.txt', 'r') as f:
    f.seek(3)
    data = f.read(8)    # 读取前8个字节
    print(data)

--------------------------------------------------

# Output:
lo Pytho
```

#### 读取一行 readline()

!!! note ""
    `file_obj.readline()`

`readline()`方法会读取文件的一行，遇到`\n`就认为是一行

```python
# io.txt
Hello Python!
I am Boii.
I am learning.

--------------------------------------------------

# io_read.py
with open('io.txt', 'r') as f:
    data = f.readline()    # 读取第一行
    print(data)

--------------------------------------------------

# Output:
Hello Python!

```
这样只能读取一行，可以使用 `while` 循环一行一行的读取

```python
# io.txt
Hello Python!
I am Boii.
I am learning.

--------------------------------------------------

# io_read.py
with open('io.txt', 'r') as f:
    data = f.readline()    # 读取第一行
    while data:
        print(data, end='')
        data = f.readline()    # 接着读下一行

--------------------------------------------------

# Output:
Hello Python!
I am Boii.
I am learning.
```

#### 读取多行 readlines()

!!! note ""
    `file_obj.readlines()`

`readlines()`方法会一次性读取文件中多行内容，最后返回一个 list，一个元素一行内容。

```python
# io.txt
Hello Python!
I am Boii.
I am learning.

--------------------------------------------------

# io_read.py
with open('io.txt', 'r') as f:
    datas = f.readlines()    # 读取多行
    for line in datas:
        print(line, end='')

--------------------------------------------------

# Output:
Hello Python!
I am Boii.
I am learning.
```
效果是一样的。


### 写 write

写入的过程：1. 打开文件；2. 写入文件；3. 关闭文件

写入同样可以使用`with`语句，`with`语句自动会关闭文件

读文件的时候如果文件不存在，会抛出 `IOError` 错误

写文件的时候如果文件不存在，会创建文件

写入有两种，一种是覆盖原有内容写入新内容；一种是在原有内容基础上追加新内容。

覆盖写在`open()`函数中要传入 `w`，追加写在`open()`函数中要传入`a`

#### 写入单行 wirte()

```python
# io_write.py
contend = '''I am the contend which be writed down into file.
Hello Python!'''

with open('io.txt', 'w') as f:
    f.write(contend)    # 写入文件

--------------------------------------------------

# io.txt
I am the contend which be writed down into file.
Hello Python!

```

#### 写入多行 writelines()

!!! note ""
    `file_obj.writelines(str)`

当有多行要写入的时候，除了上面用 `'''`的多行字符串，可以用 `writelines(list)`来将内容一次写入。第一个参数接受的是一个 list 列表 或 string 字符串

```python
# io_write.py
datas = ['Hello Python!', 'I am Boii', 'I like coding']
with open('io.txt', 'w') as f:
    f.writelines(datas)    # 写入文件

--------------------------------------------------

# io.txt
Hello Python!I am BoiiI like coding

```

跟预设结果不一样，并没有换行符。可以直接给`datas`的每个元素加上`\n`

但是这种办法很费力，且很多时候内容不是你决定的，这时候 列表生成式 就排上用场了。

```python
# io_write.py
datas = ['Hello Python!', 'I am Boii', 'I like coding']
with open('io.txt', 'w') as f:
    new_datas = [line + '\n' for line in datas]
    f.writelines(new_datas)    # 写入文件

--------------------------------------------------

# io.txt
Hello Python!
I am Boii
I like coding

```

#### 追加写

```python
# io_write.py
contend = '''I am the contend which be writed down into file.
Hello Python!'''

with open('io.txt', 'a') as f:
    f.write(contend)    # 写入文件

--------------------------------------------------

# io.txt 第一次执行
I am the contend which be writed down into file.
Hello Python!

# io.txt 第二次执行
I am the contend which be writed down into file.
Hello Python!I am the contend which be writed down into file.
Hello Python!
```

### 打开模式
打开模式有很多，大致分为：

| 字符 | 意义                           |
| :-- | :----------------------------- |
| r   | 读取（默认）                    |
| w   | 覆盖写                          |
| a   | 追加写                          |
| x   | 排他性创建，如果文件已存在则失败 |
| b   | 二进制模式                      |
| t   | 文本模式（默认）                |
| +   | 可读可写                       |

表中的字符组合起来就有丰富的打开方式

- `r`只读，`r+`可读可写，`rb`只读一个二进制文件（图片音视频等）
- `w`只覆盖写，`w+`可读可覆盖写，`wb`只覆盖写一个二进制文件
- `a`只追加写，`a+`可读可追加写，以此类推

### 字符编码

要读取非UTF-8编码的文件，需要给`open()`函数传入`encoding`参数

例如打开一个GBK编码的文件
```python
>>> with open('io.txt', 'r', encoding='gbk') as f:
···    print(f.read())
Hello Python!
```

但有时候打开的文件编码不规范，混杂了多种编码的字符，
而打开时只能按一种编码打开，所以势必会造成乱码。
这种乱码的处理可以用 `errors`参数。

!!! note ""
    `open(filepath, mode, encoding='value', errors='value')`

`errors`参数的值有

- ignore：直接忽略
- strict：引发`ValueError`异常。与默认值`None`效果相同
- replace_sign：会将错误的地方替换为指定的`replace_sign`符号
- 其他