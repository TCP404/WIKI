# Hello MySQL

## 概念

-   **数据库**（Database，**DB**）：按照一定格式存储数据的一些文件的组合。
-   **数据库管理系统**（Database Management System，**DBMS**）：用于管理数据库中数据的，数据库管理系统可以对数据库当中的数据进行增删改查。常见有 MySQL、Oracle、MS SQLServer、DB2、sybase 等。
-   **结构化查询语言**（Structured Query Language，**SQL**）：SQL 是一套标准，使用这套标准的语言可以与 DBMS 交互，使得 DBMS 执行增删改查等相应操作。

>【DBMS】 -- 执行 --> 【SQL】 -- 操作 --> 【DB】

## MySQL常用命令

-   退出：`exit` 或 `\q`
-   查看所有数据库：`show databases;`
-   查看所有表：`show tables;`
-   使用数据库：`use DB_NAME;`
-   查看表结构：`describe TB_NAME;`
-   清空表中记录：`delete from TB_NAME;`
-   查看使用引擎：`show variables like '%storage_engine%';`
-   查看所有引擎：`show engines;`

## Install

```bash
# 安装 MySQL
> yay -Sy mysql

# 初始化并获取初始登陆密码
> sudo mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql

# 启动 MySQL
> systemctl start mysqld.service

# 进入 MySQL 控制台
> mysql -u root -p
Enter password: 

# 修改默认密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
```

## 免密登录

```bash
> echo '[client]' > ~/.my.cnf
> echo 'user=帐号' >> ~/.my.cnf
> echo 'password=密码' >> ~/.my.cnf

> chmod 400 ~/.my.cnf
```