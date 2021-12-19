

## 文件管理

touch

mkdir

rmdir

rm

mv

cp



## 文件压缩

### 打包

**tar**

```shell
tar [选项] [参数]
```

- `-c, --create`: 打包，建立新打包文件
- `-x, --extract, --get`: 解包，从打包文件中还原文件
- `-f <文件名>, --file=<文件名>`: 指定打包文件
- `-v, --verbose`: 显示指令执行过程
- `-z, --gzip`: 通过 gzip 命令处理打包文件（压缩/解压）
- `-j, --bzip2`: 通过 bzip2 命令处理打包文件（压缩/解压）

```shell
$ tar -zcvf test.tar.gz	test/	# 打包 test 目录下所有文件并压缩，压缩包文件名为 test.tar.gz
$ tar -zxvf test.tar.gz 		 # 解压并解包

压　缩：tar -jcv -f filename.tar.bz2 要被压缩的文件或目录名称
查　询：tar -jtv -f filename.tar.bz2
解压缩：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录
```



### 压缩

zip



unzip

gzip

gunzip

zcat

bzip

## 文件查找

find

## 文本搜索

grep

awk

## 文本统计

**wc**：（word count）字数统计

- `-l, --lines`: 显示行数
- `-w, --words`: 显示字数
- `-c, --bytes, --chars`: 显示 Byte 数

## 用户管理

useradd

newusers

userdel



groupadd

groupdel

groupmod

批量修改密码

pwunconv

chpasswd

pwconv



chmod

chown

chgrp

chattr



`/etc/passwd`

Linux 系统中的每一个合法用户账户对应于该文件中的一行记录，记录了每个账户的属性。

如：

```bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/bin/sbin/nologin
```

一条用户记录由多个属性构成，每个属性由冒号分隔，各属性含义如下：

```
注册名 : 密码 : UID : GID : 用户名 : 用户主目录 : 用户默认命令解释程序
```

1. 注册名：用于区分不同用户，不可与其他用户的注册名重复，大小写敏感
2. 密码：一般记录在 `/etc/shadow` 中，在此文件中用 x 代替
3. UID：用户唯一标识码，除系统预置账户，新账户 UID 从 1000 开始递增
4. GID：用户组唯一标识码，除系统预置组，新组 GID 从 1000 开始递增
5. 用户名：类似用户昵称，可以重复
6. 用户主目录：用户的 home 目录
7. 默认命令解释器：用户登录后默认使用的解释器





`/etc/shadow`

Linux 系统中会使用专门的文件来记录用户的密码，当用户登录时，会将密码与 `/etc/shadow` 文件中记录的密码（加密后）进行比较。每一个用户占用一行记录。

如：

```bash
root:!:14871::::::
nobody:!*:18630::::::
dbus:!*:18630::::::
bin:!*:18630::::::
daemon:!*:18630::::::
```

一条记录共有9个属性，分别是：

1. 注册名
2. 密文密码
3. 上次更改密码距离纪元时间的天数
4. 密码更改后，不可再次更改的天数
5. 密码有效期
6. 密码失效前 n 天预警
7. 密码失效后 n 天被查封
8. 密码被查封后据纪元时间的天数
9. 保留字段

> 纪元时间：1970年1月1日



## 任务管理

bg fg jobs 

## 定时任务

**at**:

- `-f`: 指定包含具体指令的任务文件
- `-q`: 指定新任务的队列名称
- `-l`: 显示待执行任务的列表 
- `-d`: 删除指定的待执行任务
- `-m`: 任务执行完成后像用户发送 e-mail 

三天后的下午 5 点锺执行`/bin/ls`：

```bash
$ at 5pm+3 days
at> /bin/ls
at> <EOT>
job 7 at 2013-01-08 17:00
```

明天17点钟，输出时间到指定文件内：

```bash
$ at 17:20 tomorrow
at> date >/root/2013.log
at> <EOT>
job 8 at 2013-01-06 17:20
```

计划任务设定后，在没有执行之前我们可以用[atq](http://man.linuxde.net/atq)命令来==查看==系统没有执行工作任务：

```bash
$ atq
8       2013-01-06 17:20 a root
7       2013-01-08 17:00 a root
```

计划任务设定后，在没有执行之前我们可以用[atrm](http://man.linuxde.net/atrm)命令来==删除==已经设置的任务：

```bash
$ atq
8       2013-01-06 17:20 a root
7       2013-01-08 17:00 a root

$ atrm 7
$ atq
8       2013-01-06 17:20 a root
```

显示指定的任务的内容：

```bash
$ at -c 8
#!/bin/sh
# atrun uid=0 gid=0
# mail     root 0
umask 22此处省略n个字符
date >/root/2013.log
```



**crontab**:

- `-e`: 编辑计时器设置
- `-l`: 列出计时器设置
- `-r`: 删除计时器设置
- `-u 用户名`: 指定计时器的用户名称 

Linux 下的任务调度分为：`系统任务调度`、`用户任务调度`。

**系统任务调度**：系统周期性要执行的工作，如日志清理、缓存写入硬盘等。配置文件为 `/etc/crontab`，内容如下：

```bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=""
HOME=/

# run-parts
51 * * * * root run-parts /etc/cron.hourly
24 7 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
```

前 4 行用于配置 crond 任务运行的环境变量，后 4 行用于配置定时任务。



**用户任务调度**：用户定期要指定的工作，如用户数据备份，定时邮件提醒等。配置文件为 `/var/spool/cron/用户名` ，使用者权限如下：

- `/etc/cron.deny`: 该文件中所列用户不允许使用 crontab 命令
- `/etc/cron.allow`: 该文件中所列用户允许使用 crontab 命令
- `/var/spool/cron/...`:  所有用户crontab文件存放的目录,以用户名命名

**crontab 文件含义**：

格式如下：分时日月周

```bash
min hour day month week command
```

| 字段    | 含义         | 取值范围                 |
| ------- | ------------ | ------------------------ |
| min     | 分钟         | 0-59                     |
| hour    | 小时         | 0-23                     |
| day     | 日期         | 1-31                     |
| month   | 月份         | 1-12                     |
| week    | 星期几       | 0-7，0或7表示周日        |
| command | 要执行的命令 | 系统命令或自行编写的脚本 |

以上各字段还可以使用以下特殊字符：

| 符号       | 含义             | 例子                                            |
| ---------- | ---------------- | ----------------------------------------------- |
| 星号 `*`   | 代表所有可能的值 | day字段设置为 *，代表每天都执行                 |
| 逗号 `,`   | 指定多个特定值   | day字段设置为 1,8,9，代表每月1、8、9号执行      |
| 中杆 `-`   | 指定一个范围     | day字段设置为 3-5，代表每月3、4、5号都执行      |
| 正斜线 `/` | 指定时间间隔频率 | day字段设置为 0-31/2，代表每个月中每2天执行一次 |
|            |                  | hour字段设置为 */3，代表每3个小时执行一次       |

**crond服务**

```bash
service crond status
service crond start
service crond stop
service crond restart
service crond reload
```

例子：

每1分钟执行一次command

```bash
* * * * * command
```

每小时的第3和第15分钟执行

```
3,15 * * * * command
```

在上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * * command
```

每隔两天的上午8点到11点的第3和第15分钟执行

```
3,15 8-11 */2 * * command
```

每个星期一的上午8点到11点的第3和第15分钟执行

```
3,15 8-11 * * 1 command
```

每晚的21:30重启smb 

```
30 21 * * * /etc/init.d/smb restart
```

每月1、10、22日的4 : 45重启smb 

```
45 4 1,10,22 * * /etc/init.d/smb restart
```

每周六、周日的1:10重启smb

```
10 1 * * 6,0 /etc/init.d/smb restart
```

每天18 : 00至23 : 00之间每隔30分钟重启smb 

```
0,30 18-23 * * * /etc/init.d/smb restart
```

每星期六的晚上11:00 pm重启smb 

```
0 23 * * 6 /etc/init.d/smb restart
```

每一小时重启smb 

```
* */1 * * * /etc/init.d/smb restart
```

晚上11点到早上7点之间，每隔一小时重启smb

```
* 23-7/1 * * * /etc/init.d/smb restart
```

每月的4号与每周一到周三的11点重启smb 

```
0 11 4 * mon-wed /etc/init.d/smb restart
```

一月一号的4点重启smb

```
0 4 1 jan * /etc/init.d/smb restart
```

每小时执行`/etc/cron.hourly`目录内的脚本

```
01 * * * * root run-parts /etc/cron.hourly
```







## 进程管理

ps

## 磁盘管理

df

du

