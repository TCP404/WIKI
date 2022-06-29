# Transaction

Redis transaction begin with the `MULTI` command and end with `EXEC` or `DISCARD` command.

After the `MULTI` command, every non transactional command will be pushed on the queue.

But if any illegal command is pushed, an error will be reported. Then the transaction cannot be executed due to the error.


!!! warning
    Transactions in Redis do not follow the `all or nothing` principle.

    It can ensures all commands in the queue will be executed in sequence and that there will never be any command to jump in line.

    So it has no ROLLBACK operation.

## Commands

`MULTI`
:   Marks the start of a transaction block.

    After MULTI command, commands for each non transactional operation are pushed on the queue.

`EXEC/DISCARD`
:   Marks the end of a transaction block And **EXECUTE** or **DISCARD** the commands in the queue.

`WATCH`
:   `WATCH key [key ...]`

    Marks the given keys to be watched.

`UNWATCH`
:   Unmarks all the previously watched keys for a transaction.

    If you call EXEC or DISCARD, there’s no need to manually call UNWATCH.


## Simple example

```sh title="Execute transaction"
> MULTI
OK

(TX)> SET name Boii
QUEUED

(TX)> SET age 18
QUEUED

(TX)> SET bir 2022-01-01
QUEUED

(TX)> EXEC
1) OK
2) OK
3) OK
```

```sh title="Discard transaction"
> MULTI
OK

(TX)> SET name Eva
QUEUED

(TX)> SET age 18
QUEUED

(TX)> DISCARD
OK
```

```sh title="Do not atomic"
> MULTI
OK

(TX)> SET name Boii
QUEUED

(TX)> SET age 18
QUEUED

(TX)> INCR name
QUEUED

(TX)> EXEC
1) OK
2) OK
3) (error) ERR value is not an integer or out of range
```

```sh title="An illegal command"
> MULTI
OK

> SET name Boii
QUEUED

(TX)> SET age 18
QUEUED

(TX)> NO-SUCH-COMMAND abc
(error) ERR unknown command `NO-SUCH-COMMAND`, with args beginning with: `abc`,

(TX)> SET bir 2022-01-01
QUEUED

(TX)> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
```

## Watch and Unwatch

In the Redis transaction, the Lock operation is implemented through the `WATCH` command.

Before execute the `MULTI` command, you can use the `WATCH key [key ...]` to watch all keys that will be used in the following transaction.

And if other client modify any keys you watched before you execute `EXEC` command, your transaction will be not executed and return (nil).

`UNWATCH` command will be executed after `EXEC` or `DISCARD` command automatically. You can also execute it manually before `WATCH` command.


```sh title="Initial data"
> SET a 100
OK
> SET b 5
OK

> MGET a b
1) "100"
2) "5"
```

It was not watch the keys, so the result was unexpected.

| Client 1             |  Client 2       |
|----------------------|-----------------|
| > `MULTI`            |                 |
| OK                   |                 |
| (TX)> `MGET a b`     |                 |
| QUEUED               |                 |
| (TX)> `DECRBY a 100` |                 |
| QUEUED               |                 |
| (TX)> `INCRBY b 100` |                 |
| QUEUED               |                 |
| (TX)> `MGET a b`     |                 |
| QUEUED               |                 |
|                      | > `GET a`       |
|                      | "100"           |
|                      | > `DECRBY a 50` |
|                      | (integer) 50    |
| (TX)> `EXEC`         |                 |
| 1) 1) "50"           |                 |
| -- 2) "5"            |                 |
| 2) (integer) -50     |                 |
| 3) (integer) 105     |                 |
| 4) 1) "-50"          |                 |
| -- 2) "105"          |                 |


| Client 1             |  Client 2       |
|----------------------|-----------------|
| > `WATCH a b`        |                 |
| OK                   |                 |
| > `MULTI`            |                 |
| OK                   |                 |
| (TX)> `MGET a b`     |                 |
| QUEUED               |                 |
| (TX)> `DECRBY a 100` |                 |
| QUEUED               |                 |
| (TX)> `INCRBY b 100` |                 |
| QUEUED               |                 |
| (TX)> `MGET a b`     |                 |
| QUEUED               |                 |
|                      | > `GET a`       |
|                      | "100"           |
|                      | > `DECRBY a 50` |
|                      | (integer) 50    |
| (TX)> `EXEC`         |                 |
| (nil)                |                 |
| > `MGET a b`         |                 |
| 1) "50"              |                 |
| 2) "5"               |                 |


![](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/redis/tx-pipe.png)

