# SQL

## 分类

所有 SQL 语句分为 4 类：

1.   数据定义语言（Data Definition Language，DDL）：用于定义数据库对象，如 数据库、表、列等；关键字：`create、drop、alter`。
2.   数据操作语言（Data Manupulation Language，DML）：用于对数据库中表的记录进行更新；关键字：`insert、update、delete`。
3.   数据控制语言（Data Control Language，DCL）：用于定义数据库访问权限和安全级别、创建用户等；关键字：`grant`。
4.   数据查询语言（Data Query Language，DQL）：用于查询数据库中表的记录；关键字：`select`。

## DDL：create、drop、alter

### 创建数据库

#### 创建一个确定不存在的数据库

```sql
CREATE DATABASE db_name;

-- eg

mysql> CREATE DATABASE school;
Query OK, 1 row affected (0.01 sec)
```

#### 查看数据库定义

```sql
mysql> show CREATE DATABASE db_name\g;

-- eg

mysql> show create database school\g;
+----------+-----------------------------------------------------------------------------------------------------------------------------------+
| Database | Create Database                                                                                                                   |
+----------+-----------------------------------------------------------------------------------------------------------------------------------+
|  school  | CREATE DATABASE `school` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */ /*!80016 DEFAULT ENCRYPTION='N' */  |
+----------+-----------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

ERROR:
No query specified
```

#### 创建一个不确定是否存在的数据库

```sql
CREATE DATABASE db_name IF NOT EXISTS;

-- eg

mysql> CREATE DATABASE IF NOT EXISTS school;
Query OK, 1 row affected, 1 warning (0.01 sec)
```

### 删除数据库

#### 删除一个确定存在的数据库

```sql
DROP DATABASE db_name;

-- eg

mysql> DROP DATABASE school;
Query OK, 0 rows affected (0.02 sec)
```

#### 删除一个不确定是否存在的数据库

```sql
DROP DATABASE IF EXISTS db_name;

-- eg

mysql> DROP DATABASE IF EXISTS test;
Query OK, 0 rows affected, 1 warning (0.01 sec)
```

### 创建表

```sql
CREATE TABLE [IF NOT EXISTS] tb_name (
	col_name1 data_type[size] [NOT NULL | NULL] [DEFAULT value] [AUTO_INCREMENT] [COMMENT comment],
    列名称 数据类型[大小] [是|否 允许非空] [默认值] [自增长] [zhu],
    ...
    
    [PRIMARY KEY (col_name)]
)[ENGINE=engine_name 
DEFAULT CHARSET=charset
COMMENT='table infomation'];

```

-   每当将新行插入到表中时，列的值会自动增加。每个表都有一个且只有一个`AUTO_INCREMENT`列

eg:

```sql
CREATE TABLE IF NOT EXISTS student (
	id    INT(11)     NOT NULL      AUTO_INCREMENT,
    name  VARCHAR(45) DEFAULT NULL,
    birth DATE        DEFAULT NULL,
    
    PRIMARY KEY (id)
)ENGINE=InnoDB;
```

### 删除表

```sql
DROP [TEMPORARY] TABLE [IF EXISTS] tb_name [, tb_name] ...
[RESTRICT | CASCADE];
```

-   `DROP TABLE` 删表关键字，多个表之间使用逗号分割
-   `TEMPORARY` 指定仅删除临时表。用于确保不会意外删除非临时表时非常方便。
-   `IF EXISTS` 可以防止删除不存在的表，若删除不存在的表时，`show warnings;` 命令可以打印警告信息。
-   `RESTRICT` 和 `CASCADE` 为未来版本的保留关键字

>   删表需为 root 管理员或建表者才有权限。

```sql
mysql> DROP TABLE IF EXISTS student, no_this_table;
Query OK, 0 rows affected, 1 warning (0.04 sec)

mysql> show warnings;
+-------+------+--------------------------------------+
| Level | Code | Message                              |
+-------+------+--------------------------------------+
| Note  | 1051 | Unknown table 'school.no_this_table' |
+-------+------+--------------------------------------+
1 row in set (0.00 sec)
```

### 修改表

```sql
ALTER TABLE tb_name action1[, action2 ...];
```
```sql
action: {
    ADD [COLUMN] col_name col_definition [FIRST | AFTER col_name]
  | ADD [COLUMN] (col_name col_def, col_name col_def, ...)
  | ADD {INDEX | KEY} [index_name] [index_type]
    ...
}
```

-   `ALTER TABLE` 为修改表的关键字
-   `action` 为要应用于该表的操作，可以是添加新列、添加逐渐、重命名表等任何操作。

>   action 允许多个，使用逗号（,）分隔。

## DML：insert、update、delete

### insert

#### 插入单行记录

```sql
INSERT INTO tb_name(col_name1[, col_name2 ...]) 
VALUES (value1[, value2 ...]);
```

eg:

```sql
-- 表结构
mysql> describe student;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int         | NO   | PRI | NULL    | auto_increment |
| name  | varchar(45) | YES  |     | NULL    |                |
| birth | date        | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

-- 插入单行
mysql> INSERT INTO student(name, birth) VALUES ('boii', '1999-01-01');
Query OK, 1 row affected (0.02 sec)

-- 插入结果
mysql> SELECT * FROM student;
+----+------+------------+
| id | name | birth      |
+----+------+------------+
|  1 | boii | 1999-01-01 |
+----+------+------------+
1 row in set (0.00 sec)
```

#### 插入多行记录

```sql
INSERT INTO tb_name(col_name1[, col_name2 ...]) 
VALUES (value1[, value2 ...]),
	   (value1[, value2 ...]),
	   (value1[, value2 ...]),
	   ...;
```

eg:

```sql
-- 表结构
mysql> describe student;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int         | NO   | PRI | NULL    | auto_increment |
| name  | varchar(45) | YES  |     | NULL    |                |
| birth | date        | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
3 rows in set (0.00 sec)

-- 插入多行
mysql> INSERT INTO student(name, birth) VALUES
    -> ('Alice', '2000-01-02'),
    -> ('Candy', '1992-02-21'),
    -> ('Eva', '1998-08-24');
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0

-- 插入结果
mysql> SELECT * FROM student;
+----+-------+------------+
| id | name  | birth      |
+----+-------+------------+
|  1 | boii  | 1999-01-01 |
|  2 | Alice | 2000-01-02 |
|  3 | Candy | 1992-02-21 |
|  4 | Eva   | 1998-08-24 |
+----+-------+------------+
4 rows in set (0.00 sec)
```

#### 插入所有列

如果插入所有列，在表名后可省略列名。

```sql
INSERT INTO tb_name 
VALUES (value1[, value2 ...]);
```

eg:

```sql
mysql> INSERT INTO student VALUES (5, 'Danish', '1997-05-06');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Candy  | 1992-02-21 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
+----+--------+------------+
4 rows in set (0.00 sec)
```

#### 处理重复主键情况 on duplicate key update

如果插入的新行违反主键约束（唯一且非空）或 UNIQUE 约束，MySQL 会报错，可以使用 `ON DUPLICATE KEY UPDATE` 加以处理。

```sql
-- 插入前
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Candy  | 1992-02-21 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
+----+--------+------------+

-- 插入重复主键的记录
-- 处理策略为修改主键值
mysql> INSERT INTO student VALUES (3, 'Franch', '1994-03-05') ON DUPLICATE KEY UPDATE id = id + 3;
Query OK, 2 rows affected (0.01 sec)

-- 插入后
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
+----+--------+------------+
5 rows in set (0.00 sec)

-- 再次尝试插入
mysql> INSERT INTO student VALUES (3, 'Franch', '1994-03-05') ON DUPLICATE KEY UPDATE id = id + 3;
Query OK, 1 row affected (0.01 sec)

-- 这次主键就不重复了
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
+----+--------+------------+
6 rows in set (0.00 sec)
```

处理冲突的时候并不是将要插入的记录的主键进行修改，而是对表中被冲突的那条记录的主键进行修改，接着再执行一次插入就能插入成功了。

#### insert select

在 MySQL 中可以使用 SELECT 语句返回的值来填充 INSERT 语句的值。这个功能在赋值表时非常方便。

```sql
INSERT INTO tb_name
SELECT col_name[, col_name ...] FROM tb_name;
```

eg:

```sql
-- 复制表结构
mysql> CREATE TABLE student2 LIKE student;
Query OK, 0 rows affected (0.04 sec)

-- 查看所有表，可以看到现在有两张表了。
mysql> show tables;
+------------------+
| Tables_in_school |
+------------------+
| student          |
| student2         |
+------------------+
2 rows in set (0.01 sec)


-- 将 student 的记录复制到新表 student2。
mysql> INSERT INTO student2
    -> SELECT * FROM student;
Query OK, 6 rows affected (0.01 sec)
Records: 6  Duplicates: 0  Warnings: 0

-- 复制结果
mysql> SELECT * FROM student2;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
+----+--------+------------+
6 rows in set (0.00 sec)
```

#### insert ignore

当使用 INSERT 语句向表中添加一些行数据并且在处理期间发生错误时，INSERT 语句将被终止，并返回错误消息，由此导致没有插入任何的行，即插入失败。

但是如果使用 `INSERT IGNORE` 语句，则会忽略错误的行，并继续插入其他的行到表中。

>   `IGNORE`子句是MySQL对SQL标准的扩展。

```sql
INSERT IGNORE INTO tb_name[col_list]
VALUES (value_list),
	   (value_list),
	   ...;
```

eg:

```sql
-- 插入之前
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
+----+--------+------------+
6 rows in set (0.00 sec)

-- 没使用 ignore 的情况
mysql> INSERT INTO student VALUES
    -> (7, 'Gilis', '2011-02-09'),
    -> (8, 'Halis', '1995-09-10'),
    -> (4, 'Ivy', '1998-04-05'),	-- 这条将导致所有插入失败
    -> (9, 'Jene', '1996-03-11');
ERROR 1062 (23000): Duplicate entry '4' for key 'student.PRIMARY'

-- 报错，一条都没插入成功
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
+----+--------+------------+
6 rows in set (0.00 sec)



-- 使用 ignore 的情况
mysql> INSERT IGNORE INTO student VALUES
	-> (7, 'Gilis', '2011-02-09'),
    -> (8, 'Halis', '1995-09-10'),
    -> (4, 'Ivy', '1998-04-05'),	-- 这条将插入失败，其他不影响
    -> (9, 'Jene', '1996-03-11');
Query OK, 3 rows affected, 1 warning (0.02 sec)
Records: 4  Duplicates: 1  Warnings: 1

-- 错误的行没插入，其他都插入成功了。
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
|  7 | Gilis  | 2011-02-09 |
|  8 | Halis  | 1995-09-10 |
|  9 | Jene   | 1996-03-11 |
+----+--------+------------+
9 rows in set (0.00 sec)
```

### update

#### 单表更新

```sql
UPDATE [LOW_PRIORITY] [IGNORE] tb_name
SET
	col_name1 = exp1,
	col_name2 = exp2,
	...
[WHERE
 	condition];
```

-   `LOW_PRIORITY` 指示 UPDATE 语句延迟更新，延迟到没有连接在读取该表才更新；仅对使用表级锁定的引擎生效，如 MyISAM，MERGE，MEMORY。
-   `IGNORE` 忽略更新过程中发生错误的行，然后继续更新其他行。
-   `WHERE` 子句可选，如果忽略不写则对表中所有行进行修改。

eg:

```sql
-- 更新前
mysql> select * from student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
|  7 | Gilis  | 2011-02-09 |	-- 修改这条记录
|  8 | Halis  | 1995-09-10 |
|  9 | Jene   | 1996-03-11 |
+----+--------+------------+
9 rows in set (0.00 sec)

-- 更新
mysql> UPDATE student
    -> SET
    ->     name = 'Galy'
    -> WHERE
    ->     id = 7;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0


-- 更新后
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1998-08-24 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
|  7 | Galy   | 2011-02-09 |	-- name 列已经被修改
|  8 | Halis  | 1995-09-10 |
|  9 | Jene   | 1996-03-11 |
+----+--------+------------+
9 rows in set (0.00 sec)


-- 使用 IGNORE 子句更新
mysql> UPDATE IGNORE student
    -> SET
    ->     birth = '1997-07-01'
    -> WHERE
    ->     id = 4 || id = 8 || id = 10;
Query OK, 2 rows affected, 2 warnings (0.01 sec)
Rows matched: 2  Changed: 2  Warnings: 2

-- 没有 id 为 10 的行，但因为 IGNORE 所以 id 4 和 8 依然可以更新 
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1997-07-01 |	-- id 4 的行已修改成功
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
|  7 | Galy   | 2011-02-09 |
|  8 | Halis  | 1997-07-01 |	-- id 8 的行已修改成功
|  9 | Jene   | 1996-03-11 |
+----+--------+------------+
9 rows in set (0.00 sec)
```

#### 联表更新 update join

```sql
UPDATE T1 [INNER JOIN | LEFT JOIN] T2
ON T1.C1 = T2.C1
SET 
	T1.C2 = T2.C2, 
    T2.C3 = expr
[WHERE
	condition];
```

-   T1 为要更新的主表、T2 为主表连接的从表。
-   `INNER JOIN` 为内连接，即 T1 与 T2 的交集，`LEFT JOIN` 为内连接的基础上并上 T1 与 T2 的差集，即 T1 中有而 T2 没有的部分 + T1 和 T2 都有的部分。
-   `ON` 指定连接条件
-   `SET` 指定更新规则
-   `WHERE` 为可选，筛选符合条件的行，忽略则对连接结果的所有行生效。

eg:

先创建 gross_score 表和 math_score 表

```sql
mysql> CREATE TABLE math_score (
    -> id INT(11) NOT NULL AUTO_INCREMENT,
    -> score INT(3) NULL,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected, 2 warnings (0.04 sec)    

mysql> CREATE TABLE gross_score (
    -> id INT(11) NOT NULL AUTO_INCREMENT,
    -> score INT(3) NULL,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected, 2 warnings (0.04 sec)
```

然后向两张表中插入一些数据：

```sql
mysql> INSERT INTO math_score VALUES
    -> (1, 99),
    -> (2, 87),
    -> (3, 64),
    -> (4, 96),
    -> (5, 79),
    -> (6, 40);
Query OK, 6 rows affected (0.02 sec)
Records: 6  Duplicates: 0  Warnings: 0

mysql> INSERT INTO gross_score VALUES
    -> (1, 192),
    -> (2, 287),
    -> (3, 164),
    -> (4, 396),
    -> (5, 279),
    -> (6, 140);
Query OK, 6 rows affected (0.02 sec)
Records: 6  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM math_score;
+----+-------+
| id | score |
+----+-------+
|  1 |    99 |
|  2 |    87 |
|  3 |    64 |
|  4 |    96 |
|  5 |    79 |
|  6 |    40 |
+----+-------+
6 rows in set (0.00 sec)

mysql> SELECT * FROM gross_score;
+----+-------+
| id | score |
+----+-------+
|  1 |   192 |
|  2 |   287 |
|  3 |   164 |
|  4 |   396 |
|  5 |   279 |
|  6 |   140 |
+----+-------+
6 rows in set (0.00 sec)
```

接着我们使用**联表更新**：

```sql
mysql> UPDATE gross_score INNER JOIN math_score
    -> ON gross_score.id = math_score.id
    -> SET
    -> 	gross_score.score = gross_score.score + math_score.score
Query OK, 6 rows affected (0.02 sec)
Rows matched: 6  Changed: 6  Warnings: 0


mysql> SELECT * FROM gross_score;
+----+-------+
| id | score |
+----+-------+
|  1 |   291 |
|  2 |   374 |
|  3 |   228 |
|  4 |   492 |
|  5 |   358 |
|  6 |   180 |
+----+-------+
6 rows in set (0.00 sec)
```

可以看到 gross_score 表的每一项都更新成功了。

### delete

```sql
DELETE FROM tb_name
[WHERE condition];
```



```sql
-- 删除之前
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  3 | Franch | 1994-03-05 |
|  4 | Eva    | 1997-07-01 |	-- id 4 的行已修改成功
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
|  7 | Galy   | 2011-02-09 |
|  8 | Halis  | 1997-07-01 |	-- id 8 的行已修改成功
|  9 | Jene   | 1996-03-11 |
+----+--------+------------+
9 rows in set (0.00 sec)

-- 删除名字为 Franch 的记录
mysql> DELETE FROM student
    -> WHERE name = 'Franch';
Query OK, 1 row affected (0.01 sec)

-- 删除后，表中已没有 Franch 的记录
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  4 | Eva    | 1997-07-01 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | 1992-02-21 |
|  7 | Galy   | 2011-02-09 |
|  8 | Halis  | 1997-07-01 |
|  9 | Jene   | 1996-03-11 |
+----+--------+------------+
8 rows in set (0.00 sec)
```

## DCL：grant

DCL 语句主要是管理数据库权限的时候使用，这类操作一般是 DBA 使用的，开发人员不会使用 DCL 语句。

## DQL：select

```sql
SELECT [DISTINCT] * | col_list | sql_function
FROM T1 [ [LEFT | INNER | RIGHT | CROSS] JOIN T2 ON conditions ]
[WHERE conditions]
[GROUP BY col]
[HAVING group_conditions]
[ORDER BY col1 [DESC | ASC] [, col2 [DESC | ASC]]]
[LIMIT offset, length];
```

-   `DISTINCT` 用于将结果去重。
-   `FROM` 后跟要查询的表，只有一个表名时为单表查询，使用 `JOIN` 时为联表查询。
-   `WHERE` 用于设置条件，满足此条件的行才会被加入结果集。
-   `GROUP BY` 通常和 SQL 函数配合使用，用于将结果集按一个或多个列进行分组，即将结果在行的方向上进行分组。
-   `HAVING` 用于对 `GROUP BY` 的结果进行筛选，一半跟随在 `GROUP BY` 后，也可以不跟随而单独使用。
-   `ORDER BY` 用于将结果集进行排序，`ASC` 是升序，`DESC` 是降序。
-   `LIMIT` 用于限制返回的行数，`offset` 表示从结果集第几行开始，默认为 0 即第一行，`length` 表示显示多少行。

### 最简单的方式

```sql
SELECT * FROM tb_name;
```

### 去重

#### 查询单行时使用去重

```sql
mysql> SELECT jobtitle FROM employees;
+----------------------+
| jobtitle             |
+----------------------+
| President            |
| VP Sales             |
| VP Marketing         |
| Sales Manager (APAC) |
| Sale Manager (EMEA)  |
| Sales Manager (NA)   |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
| Sales Rep            |
+----------------------+
23 rows in set (0.00 sec)

-- 使用去重关键字
mysql> SELECT DISTINCT jobtitle FROM employees;
+----------------------+
| jobtitle             |
+----------------------+
| President            |
| VP Sales             |
| VP Marketing         |
| Sales Manager (APAC) |
| Sale Manager (EMEA)  |
| Sales Manager (NA)   |
| Sales Rep            |
+----------------------+
7 rows in set (0.00 sec)
```

#### 查询多行时去重

当查询多行还需要去重时，去重的规则是每一列都重复才算重复。

```sql
mysql> SELECT DISTINCT jobtitle, id FROM employees;
+----------------------+----+
| jobtitle             | id |
+----------------------+----+
| President            |  1 |
| VP Sales             |  2 |
| VP Marketing         |  3 |
| Sales Manager (APAC) |  4 |
| Sale Manager (EMEA)  |  5 |
| Sales Manager (NA)   |  6 |
| Sales Rep            |  7 |
| Sales Rep            |  8 |
| Sales Rep            |  9 |
| Sales Rep            | 10 |
| Sales Rep            | 11 |
| Sales Rep            | 12 |
| Sales Rep            | 13 |
| Sales Rep            | 14 |
| Sales Rep            | 15 |
| Sales Rep            | 16 |
| Sales Rep            | 17 |
| Sales Rep            | 18 |
| Sales Rep            | 19 |
| Sales Rep            | 20 |
| Sales Rep            | 21 |
| Sales Rep            | 22 |
| Sales Rep            | 23 |
+----------------------+----+
23 rows in set (0.00 sec)
```

#### DISTINCT 与 NULL

在去重时，会将 NULL 视为普通值。如果列中有多个 NULL，在结果集中会显示一个 NULL。

```sql
mysql> SELECT * FROM student;
+----+--------+------------+
| id | name   | birth      |
+----+--------+------------+
|  1 | boii   | 1999-01-01 |
|  2 | Alice  | 2000-01-02 |
|  4 | Eva    | 1997-07-01 |
|  5 | Danish | 2000-01-03 |
|  6 | Candy  | NULL       |	-- null 值
|  7 | Galy   | 2011-02-09 |
|  8 | Halis  | 1997-07-01 |
|  9 | Jene   | NULL       |	-- null 值
+----+--------+------------+
8 rows in set (0.00 sec)

mysql> SELECT DISTINCT birth FROM student;
+------------+
| birth      |
+------------+
| 1999-01-01 |
| 2000-01-02 |
| 1997-07-01 |
| 2000-01-03 |
| NULL       |	-- null 值
| 2011-02-09 |
+------------+
6 rows in set (0.00 sec)
```

#### DISTINCT 与聚合函数

在使用 SQL 的聚合函数如 `SUM()、AVG()、COUNT()` 时，可以使用 DISTINCT 去重，从而得到想要的结果。

```sql
mysql> SELECT * FROM employees;
+----+-----------+-----------+----------------------+
| id | lastname  | firstname | jobtitle             |
+----+-----------+-----------+----------------------+
|  1 | Murphy    | Diane     | President            |
|  2 | Patterson | Mary      | VP Sales             |
|  3 | Firrelli  | Jeff      | VP Marketing         |
|  4 | Patterson | William   | Sales Manager (APAC) |
|  5 | Bondur    | Gerard    | Sale Manager (EMEA)  |
|  6 | Bow       | Anthony   | Sales Manager (NA)   |
|  7 | Jennings  | Leslie    | Sales Rep            |
|  8 | Thompson  | Leslie    | Sales Rep            |
|  9 | Firrelli  | Julie     | Sales Rep            |
| 10 | Patterson | Steve     | Sales Rep            |
| 11 | Tseng     | Foon Yue  | Sales Rep            |
| 12 | Vanauf    | George    | Sales Rep            |
| 13 | Bondur    | Loui      | Sales Rep            |
| 14 | Hernandez | Gerard    | Sales Rep            |
| 15 | Castillo  | Pamela    | Sales Rep            |
| 16 | Bott      | Larry     | Sales Rep            |
| 17 | Jones     | Barry     | Sales Rep            |
| 18 | Fixter    | Andy      | Sales Rep            |
| 19 | Marsh     | Peter     | Sales Rep            |
| 20 | King      | Tom       | Sales Rep            |
| 21 | Nishi     | Mami      | Sales Rep            |
| 22 | Kato      | Yoshimi   | Sales Rep            |
| 23 | Gerard    | Martin    | Sales Rep            |
+----+-----------+-----------+----------------------+
23 rows in set (0.00 sec)

-- 使用 COUNT 计算行数
mysql> SELECT COUNT(jobtitle) FROM employees;
+------------------+
| COUNT( jobtitle) |
+------------------+
|               23 |
+------------------+
1 row in set (0.00 sec)

-- 使用 DISTINCT 去重
mysql> SELECT DISTINCT jobtitle FROM employees;
+----------------------+
| jobtitle             |
+----------------------+
| President            |
| VP Sales             |
| VP Marketing         |
| Sales Manager (APAC) |
| Sale Manager (EMEA)  |
| Sales Manager (NA)   |
| Sales Rep            |
+----------------------+
7 rows in set (0.00 sec)

-- 在 COUNT 中使用 DISTINCT 去重后计算行数 
mysql> SELECT COUNT(DISTINCT jobtitle) FROM employees;
+--------------------------+
| COUNT(DISTINCT jobtitle) |
+--------------------------+
|                        7 |
+--------------------------+
1 row in set (0.00 sec)
```

### 