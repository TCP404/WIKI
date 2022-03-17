# Shell-4-字符串处理

##  \${}

### 获取字符串长度

```shell
${#字符串}
```

eg：

```shell
echo ${#"Boii"}		# 错误写法

word="Boii"
echo ${#word}		# 4
```



### 字符串切片

```shell
${string:start:length}	# 对 string 从 start 开始截取 length 个字符
${string:start}			 # 对 string 从 start 开始截取到 字符串末尾
```

> 下标从 0 开始，负数表示倒数第 n 个。

eg:

```shell
string:| H | e | l | l | o |   | B | o | i | i |
正标：  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
负标：  |-10| -9| -8| -7| -6| -5| -4| -3| -2| -1| 

word="Hello Boii"

# 从左往右数，向右截取
echo ${word:0:5}	# Hello
echo ${word:6:2}	# Bo	从第7个开始，截取2个
echo ${word:6:-1}	# Boi  从第7个开始，截取至倒数第1个
echo ${word:6} 		# Boii 从第7个开始，截取至末尾
echo ${word:(-1)}	# i    截取最后一个字符
echo ${word:(-2)}	# ii   截取最后两个字符
echo ${word:(-3):2}	# oi   从倒数第3个开始，截取2个

# 从右往左数，向右截取
echo ${word:0-8:5}		# llo B
echo ${word:0-8}		# llo Boii
```

### 替换字符串

```shell
${string/OLD/NEW}	# 用 NEW 替换字符串中 第一个 OLD
${string//OLD/NEW}	# 用 NEW 替换字符串中 所有 OLD
```

eg：

```shell
str="abc,def,ghi,abcxyz"
echo ${str/abc/BOII}	# BOII,def,ghi,abcxyz
echo ${str//abc/BOII}	# BOII,def,ghi,BOIIxyz
```



### 字符串截取

- `#`：取右、删头
    - `#`：最短匹配，匹配第一个
    - `##`：最长匹配，匹配最后一个
- `%`：取左、删尾
    - `%`：最短匹配，匹配第一个
    - `%%`：最长匹配，匹配最后一个

```shell
str="opcabxyzc,def,ghi,abcxyz"

echo ${str#*ab}		# xyzc,,def,ghi,abcxyz
echo ${str##*ab}	# cxyz
```



```shell
str="opcabxyzc,def,ghi,abcxyz"

echo ${str%xyz*}	# abxyzc,def,ghi,abc
echo ${str%%xyz*}	# ab
```

![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Linux/%E5%AD%97%E7%AC%A6%E4%B8%B2%E6%88%AA%E5%8F%96-20210617203833.png)

### 变量状态赋值

- `${VAR:-str}` 如果 VAR 变量为空则返回 str

    ```go
    if VAR == nil {
        return str
    }else{
        return VAR
    }
    ```

    eg:

    ```shell
    word=
    echo ${word:-abc}	# abc
    
    word=123
    echo ${word:-abc}	# 123
    ```

    

- `${VAR:+str}` 如果 VAR 变量不为空则返回 str

    ```go
    if VAR != nil {
        return str
    }else{
        return nil
    }
    ```

    eg:

    ```shell
    word=
    echo ${word:+abc}	#
    
    word=123
    echo ${word:+abc}	# 123
    ```

    

- `${VAR:=str}` 如果 VAR 变量为空则重新赋值 VAR 变量值为 str

    ```go
    if VAR == nil {
        VAR = str
    }
    ```

    eg:

    ```shell
    word=
    echo ${word:=abc}	# abc
    
    word=123
    echo ${word:=abc}	# 123
    ```

    

- `${VAR:?str}` 如果 VAR 变量为空则将 str 输出到 stderr

    ```go
    if VAR == nil {
        panic(str)
    }
    ```

    eg:

    ```shell
    word=
    echo ${word:?word is empty}	# test.sh:行2: word: word is empty
    
    word2="aaa"
    echo ${word2:?word2 is empty} # aaa
    ```

    

### 字符串颜色

