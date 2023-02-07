# Explain

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
| -- | ----------- | ----- | ---- | ------------- | --- | ------- | --- | ---- | ----- |
|    |             |       |      |               |     |         |     |      |       |


## id


## select_type

| 值                   | 说明                                                                                      |
| -------------------- | ---------------------------------------------------------------------------------------- |
| SIMPLE               | 简单 SELECT 查询，不使用 UNION 或子查询                                                     |
| PRIMARY              | 最外层的 SELECT，子查询中最外层查询。若查询中包含任何复杂子部分，最外层的 SELECT 都被标记为 PRIMARY |
| UNION                | UNION 中的第二个或之后的 SELECT 语句                                                        |
| DEPENDENT UNION      | UNION 中的第二个或之后的 SELECT 语句，取决于外层的查询                                        | 
| UNION RESULT         | UNION 的结果，UNION 中第二个 SELECT 或之后的 SELECT 语句                                     |
| SUBQUERY             | 子查询中的第一个 SELECT，结果不依赖于外层的查询                                                | 
| DEPENDENT SUBQUERY   | 子查询中的第一个 SELECT，结果依赖于外部查询                                                   |
| DERIVED              | 派生表，派生表的 SELECT，FROM 子句的子查询                                                   | 
| MATERIALIZE          | 物化子查询                                                                                 |
| UNCACHEABLE SUBQUERY | 不能缓存其结果的子查询，必须对外层查询的每一行重新求值                                           |
| UNCACHEABLE UNION    | UNION 中属于非缓存子查询的第二个或之后的选择（一个子查询的结果不能被缓存，必须重新评估外连接的第一行） |


## table


## type

表示 MySQL 在表中找到所需行的方式，又称“访问类型”。

常用类型有：All、Index、Range、Ref、eq_ref、Const、System、NULL

## possible_keys


## key


## key_len


## ref 


## rows


## Extra


