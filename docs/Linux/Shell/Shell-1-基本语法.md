# Shell
## 注释

### 单行注释
```bash
# 这是单行注释
# 使用井号开头
```

### 多行注释
第一种：
```bash
<<< COMMENT

    这是多行注释
    "COMMENT" 并不是固定的，可以换成其他内容

COMMENT
```

第二种：
```bash
:'

这是多行注释
以一个冒号和左单引号开头
以一个右单引号结尾

'
```

## 引号
使用简单的文本和字符串时，单引号或双引号都不会有任何区别.

使用变量时，单引号会原样输出，变量不会被转义；双引号会转义。
```bash
#!/bin/bash
name="Boii"

echo "My name is $name"
echo 'My name is $name'
```

输出：
```shell
My name is Boii
My name is $name
```

## 位置参数

`$0` 代表文件名、`$1` 代表第一个参数、`$2` 代表第二个参数、...、`$n` 代表第 n 个参数

- 使用 `shift` 可以滑动参数（`$0` 除外）

```shell
ec  A  B  C  D  E  F  G     # 命令行
$0 $1 $2 $3 $4 $5 $6 $7     # 原位置参数
$0    $1 $2 $3 $4 $5 $6     # 移位后位置参数
```

    



## 变量

- 变量名
    - [ _ | a-z | A-Z | 0-9]
    - **声明** 变量时 **省略** 符号`$`
    - 使用变量时 **加上** 符号`$`
    - 变量名 **区分** 大小写
    - 等号两侧不可以有空格

- **定义变量**
    ```bash
    var_name=value
    ```

- **使用变量**
    ```bash
    $var_name
    ```

    eg：
    ```bash
    #!/bin/bash
    name="Boii"	# 定义变量
    echo $name	# 使用变量
    ```

- **只读变量**：使用 `readonly`修饰
    ```bash
    name="Boii"     # 定义变量
    readonly name   # 设定只读
    ```

- **取消设置变量**：使用 `unset`修饰
  
    ```bash
    name="Boii"  # 定义变量
    unset name   # 取消设置
    ```
    注销或删除的变量告诉shell删除的变量的变量列表做了跟踪。一旦您取消设置变量，你不可以访问存储在变量值。


### 系统变量
这些变量是由LINUX操作系统本身创建和维护的预定义变量。它们的标准约定是通常以大写字母进行定义。因此，每当看到以大写字母定义的变量时，很可能它们就是系统定义的变量。


| 变量      | 备注                                               |
|-----------|--------------------------------------------------|
| $SHELL    | 默认shell                                          |
| $HOME     | 当前用户家目录                                     |
| $IFS      | 内部字段分隔符                                     |
| $LANG     | 默认语言                                           |
| $PATH     | 默认可执行程序路径                                 |
| $PWD      | 当前目录                                           |
| $UID      | 当前用户ID                                         |
| $USER     | 当前用户                                           |
| $HISTSIZE | 历史命令大小，可通过HISTTIMEFO 变量设置命令执行时间 |
| $RANDOM   | 随机生成一个 0 至 32767 的整数                     |
| $HOSTNAME | 主机名                                             |


### 位置变量
指脚本或函数后跟的第n个参数 `$1~$n`，注意 n>=10 时要加花括号：`${12}`

### 特殊变量

| 变量    | 备注                                                         |
|:--------|:-------------------------------------------------------------|
| $0      | 脚本自身名字                                                 |
| \$1~\$9 | 第一个到第九个参数                                           |
| $#      | 位置参数 **个数**                                            |
| $*      | 所有位置参数看作 **一整个字符串**                            |
| $@      | 将参数列表存储为数组，每个位置参数被看作各自 **独立的字符串** |
|         |                                                              |
| $$      | 当前进程PID                                                  |
| $!      | 上一条运行后台进程的PID                                      |
| $?      | 返回上一条命令是否执行成功.[0成功，非0失败]                   |


- $@ 的使用

```bash
#!/bin/bash
args=("$@")
echo ${args[*]}
```
```bash
$ ./b.sh a b c
a b c
```


## 流程控制

### if、elif、else

- if

    ```bash
    # 多行写法
    if condition
    then
        ...
    fi

    # 或
    # 单行写法1
    
    if condition; then
        ...
    fi

    # 或
    # 单行写法2
    
    if condition; then ...; fi
    ```

- elif、else

    ```bash
    if condition; then
        ...
    elif condition; then
        ...
    fi
    
    或
    
    if condition; then
        ...
    elif condition; then
        ...
    else
        ...
    fi
    
    或
    
    if condition; then
        ...
    else
        ...
    fi
    ```

对于 **condition** 部分，有以下三点

1. 由表达式构成，表达式执行成功时返回值为0，进入 `then` 部分
    ```bash
    if echo 'hi'; then
        echo 'Hello'
    fi
    ```
2. 可以有多条表达式，但只有最后一条决定是否运行 `then` 部分
    ```bash
    if false; true; then
        echo 'Hello'
    fi
    # 结果为 打印 Hello
    ```
3. condition 有三种形式
    1. `test expression`：使用 test 命令 获取返回值
    2. `[ expression ]`：使用 test 命令的简写 `[` 并以 `]` 结束 获取返回值。
    3. `[[ expression ]]`：使用 `[[]]` 获取返回值，这种方式支持正则判断。
    4. **注意`[]`、`[[]]`与`expression`之间必须有空格。**

eg：
```bash
# 写法一
if test -e /etc/profile; then
    echo "Found profile."
fi

# 写法二
if [ -e /etc/profile ]; then
    echo "Found profile."
fi

# 写法三
if [[ -e /etc/profile ]]; then
    echo "Found profile."
fi
```

### test 命令

test 命令用于检查某个条件是否成立，可以进行 **数值、字符、文件** 三个方面的测试，测试结束后会返回一个返回值，0为真，1为假。

### 判断

#### 数值判断

a=1
b=2

| 参数  | 说明         | 举栗        | 栗子返回值 |
|:------|:-----------|:------------|------------|
| `-eq` | 相等为真     | \$a -eq \$b | 1          |
| `-ne` | 不等为真     | \$a -ne \$b | 0          |
| `-gt` | 大于为真     | \$a -gt \$b | 1          |
| `-ge` | 大于等于为真 | \$a -ge \$b | 1          |
| `-lt` | 小于为真     | \$a -lt \$b | 0          |
| `-le` | 小于等于为真 | \$a -le \$b | 0          |


!!! note
    在命令中执行基本的算术运算，需要用 `[]` 包裹
    
    eg:

    ```bash
    #!/bin/bash
    a=1
    b=2
    
    # 错误写法
    # if test $a+1 -eq $b; ...
    # if test $[a]+1 -eq $b; ...
    
    # 正确写法
    if test $[a+1] -eq $b; then
        echo "a = b"
    fi
    ```
    
    ```bash
    $ ./b.sh
    a = b
    ```

#### 字符串判断

s1="abc"

s2="xyz"

s3=""

| 参数               | 说明                   | 栗子返回值 |
|:-------------------|:---------------------|:-----------|
| [ \$s1 `=` \$s2 ]  | 相等为真               | 1          |
| [ \$s1 `==` \$s2 ] | 与 [ $s1 = $s2 ]  等价 | 1          |
| [ \$s1 `!=` \$s2 ] | 不等为真               | 0          |
| [ `-z` \$s3 ]      | 空串为真               | 0          |
| [ `-n` \$s3 ]      | 非空串为真             | 1          |
> 注意这里等号 `=` 两边需要空格，否则就成了赋值操作了

```bash
#!/bin/bash
if [ -z $s3 ]; then
    echo "字符串长度为0"
elif
    echo "字符串长度不为0"
fi
```
```bash
$ ./b.sh
字符串长度为0
```

#### 文件判断


| 参数                  | 说明                                                                                                   |
|:----------------------|:-----------------------------------------------------------------------------------------------------|
| **类型判断**          | **类型判断**                                                                                           |
| [ `-a` file ]         | 如果 file **存在**，则为true。                                                                           |
| [ `-e` file ]         | 如果 file **存在**，则为true。                                                                           |
| [ `-b` file ]         | 如果 file 存在并且是一个 **块（设备）文件**，则为true。                                                    |
| [ `-c` file ]         | 如果 file 存在并且是一个 **字符（设备）文件**，则为true。                                                  |
| [ `-d` file ]         | 如果 file 存在并且是一个 **目录**，则为true。                                                            |
| [ `-f` file ]         | 如果 file 存在并且是一个 **普通文件**，则为true。                                                        |
| [ `-S` file ]         | 如果 file 存在并且是一个 **网络 socket**，则为true。                                                     |
| [ `-h` file ]         | 如果 file 存在并且是一个 **符号链接**，则为true。                                                        |
| [ `-L` file ]         | 如果 file 存在并且是一个 **符号链接**，则为true。                                                        |
| [ `-p` file ]         | 如果 file 存在并且是一个 **命名管道**，则为true。                                                        |
| [ `-t` fd ]           | 如果 fd 是一个 **文件描述符**，并且重定向到终端，则为true。 这可以用来判断是否重定向了标准输入／输出／错误。 |
| **权限判断**          | **类型判断**                                                                                           |
| [ `-r` file ]         | 如果 file 存在并且 **可读**（当前用户有可读权限），则为true。                                              |
| [ `-w` file ]         | 如果 file 存在并且 **可写**（当前用户拥有可写权限），则为true。                                            |
| [ `-x` file ]         | 如果 file 存在并且 **可执行**（有效用户有执行／搜索权限），则为true。                                       |
| **属性判断**          | **属性判断**                                                                                           |
| [ `-g` file ]         | 如果 file 存在并且 **设置了组 ID**，则为true。                                                           |
| [ `-k` file ]         | 如果 file 存在并且 **设置了它的“sticky bit”**，则为true。                                                |
| [ `-u` file ]         | 如果 file 存在并且 **设置了 setuid 位**，则为true。                                                      |
| [ `-G` file ]         | 如果 file 存在并且 **属于有效的组 ID**，则为true。                                                       |
| [ `-O` file ]         | 如果 file 存在并且 **属于有效的用户 ID**，则为true。                                                     |
| [ `-s` file ]         | 如果 file 存在且其 **长度大于零**，则为true。                                                            |
| [ `-N` file ]         | 如果 file 存在并且自上次读取后 **已被修改**，则为true。                                                  |
| [ file1 `-nt` file2 ] | 如果 FILE1 比 FILE2 的更新时间 **更近**，或者 FILE1 存在而 FILE2 不存在，则为true。                       |
| [ file1 `-ot` file2 ] | 如果 FILE1 比 FILE2 的更新时间 **更旧**，或者 FILE2 存在而 FILE1 不存在，则为true。                       |
| [ FILE1 `-ef` FILE2 ] | 如果 FILE1 和 FILE2 **引用相同的设备和 inode 编号**，则为true。                                          |





#### 正则判断



#### 逻辑判断

### 循环

#### for 循环

第一种：

```shell
for var
do
    commands
done
```

第二种

```shell
for var in list
do
    commands
done
```

!!! example

    ```shell
    learn="Hello Shell, I am Boii"
    for word in $learn
    do
        echo $word
    done
    
    # Output
    Hello
    Shell,
    I
    am
    Boii
    ```

第三种

```shell
for num in {1..n}
do
    commands
done
```

!!! example

    ```shell
    for num in {1..5}
    do
        echo $num
    done
    
    # Output
    1
    2
    3
    4
    5
    ```

第四种

```shell
for var in {START..END..STEP}
do
    commands
done
```

!!! example

    ```shell
    for num in {1..10..2}
    do
        echo $num
    done
    
    # Output
    1
    3
    5
    7
    9
    ```

第五种

```shell
for (( expression1; expression2; expression3 ))
do
    commands
done
```

!!! example

    ```shell
    for (( i=1; i<6; i++ ))
    do
        echo $i
    done
    
    # Output
    1
    2
    3
    4
    5
    6
    ```

第六种，无限循环

```shell
for ((;;))
do
    commands
done
```

!!! example

    ```shell
    for ((;;))
    do
        echo "infinite loop"
    done
    ```

    

第七种，按空格分割读取字符串

```shell
str="Let's start
talk about sth."

for i in $str
do
    echo $i
done

#Output
Let's
start
talk
about
sth.
```

第八种，按换行分割读取字符串

```shell
str="Let's start
talk about sth."

for i in "$str"
do
    echo $i
done

#Output
Let's start
talk about sth.
```

##### 彩色打印

```shell
#!/bin/bash
for b in {40..47}
do
    for f in {30..37}
    do
        for s in 0 1 4 5 7 8
        do
            echo -n -e "\033[${s};${f};${b}mHello World!\033[0m "
        done
    done
done
```



#### while 循环

当 condition 为真时，执行 commands

1. 基本样式
    ```shell
    while condition
    do
        commands
    done
    ```

    !!! example

        ```shell
        # 单条件 while 循环
        i1=10
        i2=20
        while [[ $i1 -lt $i2 ]]
        do
            echo $i1
            ((i1++))
        done
        
        # 多条件 while 循环
        a1=10
        a2=20
        a3=30
        while [[ $a1 -gt 0 && $a2 -lt $3 ]]
        do
            echo $a2
            ((a2++))
        done
        ```

2. 无限循环
    ```shell
    while :
    do
        commands
    done
    
    或
    
    while true
    do
        commands
    done
    
    或
    
    while :; do commands; done
    ```

    !!! example

        ```shell
        while :
        do
            echo "Hello, Boii"
            sleep 1s
        done
        
        或
        
        while true
        do
            echo "Hello, Boii"
            sleep 1s
        done
        
        或
        
        while :; do echo "Hello, Boii"; sleep 1s; done
        ```

3. 带条件
    ```shell
    while ((condition))
    do
        commands
    done
    ```

    !!! example

        ```shell
        i=1
        while ((i <= 10))
        do
            echo $i
            ((i++))
        done
        ```

#### until 循环

当 condition 为假时，执行 commands

1. 基本样式
    ```shell
    until condition
    do
        commands
    done
    
    或
    
    until false
    do
        commands
    done
    ```

    !!! example

        ```shell
        i=1
        until false
        do
            if [[ $i == 10 ]]
            then
                break
            fi
            echo $i
        done
        ```

    

2. 带条件
    ```shell
    until (( condition ))
    do
        commands
    done
    ```

#### select 循环

```shell
select var in word1 word2 ... wordN
do
    commands
done
```

select 循环会列出后面所有的 word，并接收数字作为选择。常用来创建一个带编号的菜单。

```shell
#!/bin/ksh

select DRINK in tea cofee water juice appe all none
do
   case $DRINK in
      tea|cofee|water|all) 
         echo "Go to canteen"
         ;;
      juice|appe)
         echo "Available at home"
      ;;
      none) 
         break 
      ;;
      *) echo "ERROR: Invalid selection" 
      ;;
   esac
done
```

```bash
# Output

$ ./test.sh
1) tea
2) cofee
3) water
4) juice
5) appe
6) all
7) none
#? 4
Available at home
#? 7
$
```

可以修改 `PS3` 变量来更改选择提示

```bash
$ PS3="Please make a selection => " ; export PS3
$ ./test.sh
1) tea
2) cofee
3) water
4) juice
5) appe
6) all
7) none
Please make a selection => 4
Available at home
Please make a selection => 7
$
```

### case 分支

```shell
case expression in
    pattern_1)
        statements
        ;;
    pattern_2)
        statements
        ;;
    pattern_3|pattern_4|pattern_5)
        statements
        ;;
    *)
        statements
        ;;
esac
```

- `case` 开头，接 `in` 关键字，`esac` 结尾
- 分支中可以用 `|` 分隔多个模式，分支以 `)` 结尾
- 每一个分支的语句以 `;;` 终止
- 使用 `*` 表示默认分支

在第一个匹配项之后，`case`以最后执行的语句的退出状态终止。
如果没有匹配的模式，则`case`的退出状态为零。否则，返回状态是已执行语句的退出状态。
如果使用默认的星号(`*`)模式，则在没有匹配模式的情况下将执行它。

```shell
read -p "Do you know Golang?[Yes/no] " answer

case $answer in
    Yes|yes|y|Y)
        echo "That is amazing."
        echo
        ;;
    No|no|n|N)
        echo "emmmmmm."
        echo
        ;;
easc
```



