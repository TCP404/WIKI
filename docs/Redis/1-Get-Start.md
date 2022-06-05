# Hello Redis

目前正在练习英文写作，待连载完成后再重写中文版

## Install

- Mac：
    ```sh
    brew install redis
    ```

- Linux(ArchLinux):
    ```sh
    sudo pacman -Sy redis
    
    # or

    yay -Sy redis
    ```

- Source:

    ```sh
    # download source && decompress && enter into source code directory
    $ wget https://download.redis.io/releases/redis-6.2.6.tar.gz
    $ tar -zxf redis-6.2.6.tar.gz
    $ cd redis-6.2.6

    # make && install
    $ make MALLOC=libc
    $ make install PREFIX=/usr/redis    # maybe need root permission

    # create link to global command directory
    $ ln -s /usr/redis/bin/redis-server /usr/local/bin/redis-server
    $ ln -s /usr/redis/bin/redis-cli /usr/local/bin/redis-cli
    ```

## Usage
Redis need launch server end to open the redis server then manipulate it by the connection from redis client. Default port is 6379.

Redis 需要启动服务端开启 Redis 服务，然后通过客户端连接进行操作。默认端口为 「6379」。

- launch server end 启动服务端：
    ```sh
    $ redis-server
    ```

- launch server with config 指定配置文件启动服务端：
    ```sh
    $ redis-server ./custom-redis.conf
    ```

- launch client end 启动客户端：
    ```sh
    $ redis-cli -h localhost -p 6379
    
    localhost:6379> _
    ```
    It doesn't need to write host and port when the server and client be deployed on the same host.

    如果都在本机，则不需要填写主机和端口，默认就是 localhost:6379

- launch client end with raw data:
    ```sh
    $ redis-cli --raw
    ```

- 退出服务端：


- 退出客户端：
    ```sh
    localhost:6379> exit

    $ _
    ```

## Redis

Redis is a KEY-VALUE cache middleware. 

The KEY can be any binary data. Reids is binary secure. That is to say, you can use a string as a key, or use a JSON object, or even a picture as a key.

It has 5 basic types of VALUE: ==String==, ==List==, ==Set==, Sorted Set(==ZSet==) and ==Hash==.


## Conception

!!! note 
    **Database**: The basic unit used to store data. Every database has a unique ID which start from 0 to 15, because the maximum number of database in default configuration file is 16.  DB 0 is used by default.

Databases are isolated from each other. Where you execute command `set name Boii` in DB 0, you can't get it from other databases.  

change database command: `select <dbID>`

clear all data from database: `FLUSHDB`, it will clear all data from current database.

clear all data from all database: `FLUSHALL`, it will clear all data from all database.


## Manipulation with key

- SET
    - Syntax: `SET key value`
    - Description: create a pair of key-value.
    - Return: OK if setting is successful.
    - Example: `SET age 18`, tips: the default type of value is String.

- GET
    - Syntax: `GET key`
    - Description: get the value of the given key.
    - Return: value of the given key if success, otherwise nil.
    - Example: `GET age`

- DEL
    - Syntax: `DEL key [key ...]`
    - Description: delete the pair of key-value from given keys. Ignore if key is not exists.
    - Return: the number of keys deleted.
    - Example: `DEL name age bir`

- EXISTS
    - Syntax: `EXISTS key [key ...]`
    - Description: check whether the given keys is exists.
    - Return: 1 if the key or one of the given keys is exists, otherwise 0.
    - Example: `EXISTS name`

- EXPIRE
    - Syntax: `EXPIRE key seconds`
    - Description: set the expire(unit: second) for the given key, the key will be delete when expired.
    - Return: 1 if setting is successful.
    - Example: `EXPIRE age 5` means that the key `age` expiration time is set to 5 seconds later.

- PEXPIRE
    - Syntax: `PEXPIRE key microsecond`
    - Description: same with the command EXPIRE but unit is microsecond.
    - Return: 1 if setting is successful.
    - Example: `EXPIRE age 10000` means that the key `age` expiration time is set to 10000 microseconds (equal 10 second) later.

- TTL
    - Syntax: `TTL key`
    - Description: count the remainder second of the given key's expiration.
    - Return: the remainder second of given key's expiration. -1 if the key is permanent, -2 if the key is not exists.
    - Example: `TTL name`

- PTTL
    - Syntax: `PTTL key`
    - Description: same with the command TTL but unit is microsecond.
    - Return: the remainder microsecond of given key's expiration. -1 if the key is permanent, -2 if the key is not exists.
    - Example: `PTTL age`

- KEYS
    - Syntax: `KEYS pattern`
    - Description: list all keys that matched the pattern.
    - Return: list of keys that matched the given pattern.
    - Example: 
        - `KEYS *`: return all keys
        - `KEYS h?llo`: '?' match one character, this will return hello, hallo, hullo etc.
        - `KEYS h*llo`: '*' match 0 or any number of character, this will return hllo, hello, hallo, heeello etc.
        - `KEYS h[ae]llo`: '[]' restrict a range, this will return hello, hallo.

- MOVE
    - Syntax: `MOVE key dbID`
    - Description: move the given key into indicated database.
    - Return: 1 if move successful.
    - Example: `MOVE hello 2` means move the pair of key-value that key is hello into database 2.

- RANDOMKEY
    - Syntax: `RANDOMKEY`
    - Description: return a key from current database randomly without delete.
    - Return: a key, nil if database if empty.
    - Example: `RANDOMKEY`

- RENAME
    - Syntax: `RENAME key newKey`
    - Description: rename the given key to newKey.
    - Return: a error will be returned when the key is same the newKey or key is not exists, otherwise OK.
    - Example: `RENAME hillo hello`

- TYPE
    - Syntax: `TYPE key`
    - Description: return the type of the given key.
    - Return: 
        - none (key is not exists)
        - string
        - list
        - hash
        - set
        - zset

## Manipulation with Redis

- SELECT
- FLUSHALL, FLUSHDB
