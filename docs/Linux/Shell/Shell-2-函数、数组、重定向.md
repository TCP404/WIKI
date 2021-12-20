# Shell-2

## 函数

1. 函数需先声明再使用
2. 在内部使用 `$1`、 `$2`、 `$n` 等访问参数
3. 函数内的声明变量为全局变量
4. 函数内使用 `local` 限定的变量为局部变量，范围仅限于函数内
5. Bash shell 中使用函数可以覆盖内置命令

### 声明

```shell
function_name() {
	commands
}

或

function_name() { commands; }

或

function function_name {
	commands
}
```

### 使用

使用函数时，直接写函数名即可。

```shell
# 声明函数
welcome() {
	echo "Hello, I'm Boii."
}

welcome	# 调用函数
```

### 函数传参

函数传参时，在函数名后按空格分隔逐一写上参数

```shell
fn() {
	echo $1
	echo $2
	echo $3
}

fn 1 2 3

# Output
1
2
3
```

在函数中：

- `$0` 是函数名称
- `$1 ~ $n` 是参数
- `$#` 是参数个数
- `$*` 和 `$@` 保存所有参数
    - `"$*"` 等于 `"$1 $2 $3 $n"`
    - `"$@"` 等于 `"$1" "$2" "$3" "$n" `
    - `$*` 等于 `$@`

### 返回值

shell 中的函数返回的是状态，类似于程序退出状态。

默认情况下，函数返回值是函数中最后执行的语句的状态，成功返回 `0`，失败返回非零。

**在函数外部使用 `$?` 获取返回值。** 

```shell
fn() {
	echo "Hello, $1"
	return 10
}

fn "Boii"

echo "return value is $?"

# Output
Hello, Boii
10
```

第二种方式：使用 `echo` 或 `printf` 命令将打印值发送到 `stdout`：

```shell
fn() {
	echo "Hello, $1"
	echo 10
}

echo "$(fn 'Boii')"

# Output
Hello, Boii
10
```



### 作用域

函数内声明的变量默认是全局变量，想要声明局部变量，必须使用 `local` 限定。

```shell
#!/bin/bash  

v1='A'  
v2='B'  

my_var () {  
local v1='C'  
v2='D'  
echo "Inside Function"  
echo "v1 is $v1."  
echo "v2 is $v2."  
}  

echo "Before Executing the Function"  
echo "v1 is $v1."  
echo "v2 is $v2."  

my_var  

echo "After Executing the Function"  
echo "v1 is $v1."  
echo "v2 is $v2."

# Output
v1 is A.
v1 is B.
Inside Function
v1 is C.
v2 is D.
After Executing the Function
v1 is A.
v2 is D.
```

在上面输出中，如果在函数体内设置与全局变量同名的局部变量，则它将优先于全局变量(即，局部变量覆盖全局变量)。可以在函数内修改全局变量。



## 数组

Bash 支持普通数组和关联数组。

普通数组使用整数作为索引，关联数组使用字符串作为索引（类似 kv 对）。

> Bash 4.0 起才支持关联数组。

### 普通数组

#### 创建数组

```shell
# 方式1
declare -a ARR_NAME

# 方式2
ARR_NAME=(val1 val2 val3 valN)

# 方式3
ARR_NAME[0]=val1
ARR_NAME[1]=val2
ARR_NAME[2]=val3
ARR_NAME[3]=val4
...

# 方式4 将命令输出作为数组元素
ARR_NAME=($(command))
```

eg:

```shell
# 方式1
declare -a HttpMethods

# 方式2
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

# 方式3
nums[0]=10
num2[1]=11
nums[2]=12
nums[3]=13
```

```shell
time=($(date))
declare -p time

# Output
declare -a a=([0]="2021年" [1]="06月" [2]="17日" [3]="星期四" [4]="14:52:16" [5]="CST")
```

#### 获取单个元素

```shell
${ARR_NAME[INDEX]}
```

eg：

```shell
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

echo ${books[2]}					# C Shell
idx=3 && echo ${books[$idx]}	 # Bash toturial
```

> 数组的下标从 0 开始。

#### 获取所有元素

```shell
${ARR_NAME[*]}

或

${ARR_NAME[@]}
```

eg：

```shell
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

echo ${books[*]}	# Go Web Effective Go C Shell Bash toturial
echo ${books[@]}	# Go Web Effective Go C Shell Bash toturial
```

#### 获取所有元素下标

```shell
${!ARR_NAME[*]}

或

${!ARR_NAME[@]}
```

eg：

```shell
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

echo ${!books[*]}	# 0 1 2 3
echo ${!books[@]}	# 0 1 2 3

unset books[2]	# 删除第3个元素 C Shell
echo ${!books[*]}	# 0 1 3
```

#### 获取数组长度

```shell
${#ARR_NAME[*]}

或

${#ARR_NAME[@]}
```

eg：

```shell
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

echo ${#books[*]}	# 4
echo ${#books[@]}	# 4
```

#### 添加元素

- 添加单个元素

    ```shell
    ARR_NAME[IDX]=val
    ```

    eg：

    ```shell
    books=("Go Web" "Effective Go" "C Shell" "Bash toturial")
    
    books[5]=11
    
    echo ${books[*]}	# Go Web Effective Go Bash toturial 11
    ```
    
- 添加多个元素

    ```shell
    ARR_NAME+=(NEW_val1 NEW_val2 NEW_val3)
    ```

    eg:

    ```shell
    books=("Go Web" "Effective Go" "C Shell" "Bash toturial")
    
    books+=(11 12 13)
    
    echo ${books[@]}	# Go Web Effective Go Bash toturial 11 12 13
    ```



#### 删除元素、删除数组

```shell
unset ARR_NAME[IDX]

unset ARR_NAME
```

eg:

```shell
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

unset books[1]
echo ${books[*]}	# Go Web C Shell Bash toturial

unset books
echo ${books[*]}	# 
```

#### 遍历数组

```shell
books=("Go Web" "Effective Go" "C Shell" "Bash toturial")

for i in ${!books[*]}
do
	echo ${books[$i]}
done

# Output
Go Web
Effective Go
C Shell
Bash toturial
```

### 关联数组

> 关联数组必须先声明 `declare -A MAP_NAME`

#### 创建

```shell
declare -A MAP_NAME
```

eg：

```shell
declare -A color
```

#### 初始化

```shell
MAP_NAME=([K1]=V1 [K2]=V2 [K3]=V3 [KN]=VN)

或

MAP_NAME[K1]=V1
MAP_NAME[K2]=V2
MAP_NAME[K3]=V3
MAP_NAME[KN]=VN
```

eg：

```shell
declare -A f_color
f_color=([black]=30 [red]=31 [green]=32 [yellow]=33 [blue]=34 [purple]=35 [cyan]=36 [white]=37)

declare -A b_color
b_color[black]=40
b_color[red]=41
b_color[green]=42
b_color[yellow]=43
b_color[blue]=44
b_color[purple]=45
b_color[cyan]=46
b_color[white]=47
```

#### 获取元素

1. 获取单个元素

    ```shell
    ${MAP_NAME[KEY]}
    ```
    
    eg：
    
    ```shell
    declare -A f_color
    f_color=([black]=30 [red]=31 [green]=32 [yellow]=33 [blue]=34 [purple]=35 [cyan]=36 [white]=37)
    
    echo ${f_color[red]}	# 31
    ```
    
2. 获取所以元素

    ```shell
    ${MAP_NAME[*]}
    
    或
    
    ${MAP_NAME[@]}
    ```

    eg：

    ```shell
    declare -A f_color
    f_color=([black]=30 [red]=31 [green]=32 [yellow]=33 [blue]=34 [purple]=35 [cyan]=36 [white]=37)
    
    echo ${f_color[*]}	# 30 31 32 33 34 35 36 37
    ```



#### 获取所有元素下标、获取数组长度

- 获取所有元素下标

    ```shell
    ${!MAP_NAME[*]}
    
    或
    
    ${!MAP_NAME[@]}
    ```
    
- 获取数组长度

    ```shell
    ${#MAP_NAME[*]}
    
    或
    
    ${#MAP_NAME[@]}
    ```



## 重定向

- `/dev/null`：代表空设备文件，是个黑洞文件
- `0`：代表 `stdin` 标准输入
- `1`：代表 `stdout` 标准输出
- `2`：代表 `stderr` 标准错误
- `>`：代表重定向输出，`> f` 表示将输出重定向到 f 文件，会覆盖 f 原本的内容
- `>>`：代表重定向输出，`>> f` 表示将输出重定向到 f 文件，追加在 f 原本的内容后面。
- `<`：代表重定向输入，`< f` 表示输入 f 的内容
- `&`：表示等同于的意思。



- `cmd > /dev/null`：所有的正确输出将会被丢弃，因为 `/dev/null` 是个黑洞文件。
- `cmd 1> /dev/null`。与上面一条等价，错误信息还是会照样输出的。
- `cmd 2> /dev/null`：所有的报错信息将会被丢弃，而正确输出会原样打印到屏幕上。
- `cmd 2>&1`：所有报错信息将被重定向到 `stdout`，错误信息也被当成正确信息打印。
- `cmd 2>1`：所有报错信息将被重定向到文件 `1` 中。
- `cmd 1>&2`：所有正确信息将被重定向到 `stderr` ，正确信息也被当成报错信息打印。
- `cmd >&2`：和 `cmd 1>&2` 、`cmd 1>& 2`等价。



- `cmd 1>y 2>n`：正确信息将输出到文件 `y`，错误信息将输出到文件 `n`。
- `cmd >y 2>n`：与上面一条等价。
- `cmd 1>y 2>&1`：正确信息将输出到文件 `y`，错误信息将输出到 `stdout`，跟着一起输出到文件 `y`。



## 输入

使用 `read` 命令可以读取用户的输入

```shell
read [-u fd] [-n nchars] [-a arr] [-p prompt] [val1 val2 ...]
```

- `-u`：接受一个文件句柄 fd 作为参数，表示从 fd 中读取数据。
- `-n`：只读取 nchars 个字符后就返回，而不等整行输入完。
- `-p`：打印提示信息 prompt。
- `-a`：读取输入到数组中
- `val1 val2 ...`：用户的输入会存储在这些变量中，如果没有给出，用户的输入默认在 `$REPLY` 中。

```shell
#!/bin/bash

read -p "Please enter name: " name
echo "Hello $name"

# Output
> ./test.sh
Please enter name: Boii
Hello Boii
```

```shell
#!/bin/bash

read -n 3 "Please enter name:" name
echo "Hello $name"

# Output
> ./test.sh
Please enter name: Boi
Hello Boi
```

```shell
#!/bin/bash

read -a inputs -p "Enter sth: "
echo ${inputs[*]}

# Output
> ./text.sh
Enter sth: abc xyz abo
abc xyz abo
```



