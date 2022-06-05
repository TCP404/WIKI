# Hello Mongo

| MongoDB          | 关系性数据库 |
| -----------------| -----------|
| 库(Database)     | 库          |
| 集合(Collection) | 表          |
| 文档(Document)   | 行          |
| 字段(field)      | 列          |

## 库

- 查看所有库
    ```bash
    > show databases; | show dbs;
    ```
    ```bash
    > show dbs
    admin   0.000GB
    config  0.000GB
    local   0.000GB
    ```

- 创建、使用库
    ```bash
    > use <DATABASE>;
    ```
    但库不存在时会创建，但库存在时会切换。
    ```bash
    > use exam
    switched to db exam
    ```

- 查看当前使用库
    ```bash
    > db
    exam
    ```

- 删库
    ```bash
    > db.dropDatabase();
    ```
    ```bash
    > db.dropDatabase()
    { "ok" : 1 }
    ```

- **admin**: 从权限的角度看，这是 root 数据库。如果将一个用户添加到这个库中，该用户会拥有所有数据库的权限。一些特定的服务端命令（例如，列出所有库、关闭服务器）也只能从这个数据库运行。
- **local**: 这个数据库永远不会被复制，可以用来存储限于本地单台服务器的任意集合。
- **config**: 当 Mongo 用于分片设置时，config 数据库在内部使用，用于保存分片相关信息。


## 集合

- 创建集合
    ```bash
    > db.createCollection('集合名称', [options])
    ```
    options 参数：

    | 字段   | 类型 | 描述 |
    | ------ | ------- | --- |
    | capped | bool    | 默认 false。true：创建有固定大小的集合，当达到最大值时，自动覆盖最早的文档。为 true 时须指定 size |
    | size   | integer | 单位 byte，为固定集合指定一个最大值 |
    | max    | integet | 指定固定集合中包含文档的最大数量 |

    ```bash
    > db.createCollection('users')
    { "ok" : 1 }
    > db.createCollection('teacher', {capped: true, size: 100})
    { "ok" : 1 }
    ```
    当集合不存在时，向集合中插入文档也会自动创建该集合。

- 查看所有集合
    ```bash
    > show collections; | show tables;
    ```

    ```bash
    > show tables
    teacher
    users
    ```

- 删除集合
    ```bash
    > db.集合名称.drop()
    ```
    ```bash
    > db.users.drop()
    true
    ```

