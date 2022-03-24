# Type: Set 

Set is an disorderly, unrepeatable, random type. 

The key words of type Set are **Random** and **Disorder**. So it is suitable for `raffle` scenarios.

It has some manipualtion:

| manipualtion | command               |
| :----------- | :---------------------|
| set          | SADD                  |
| get          | SPOP, SRANDMEMBER     |
| list         | SMEMBERS              |
| length       | SCARD                 |
| delete       | SREM                  |
| move         | SMOVE                 |
| check        | SISMEMBER             |
| operate      | SDIFF, SINTER, SUNION |

## SADD

- **Syntax**: `SADD key member [member ...]`
- **Description**: Adds a member into set which named `key`. The set will be created if it does not exist.
- **Return**: Number of successful added.

```sh
> SADD letters a b c d e f g a b c
(integer) 7

> SADD letters x y z
(integer) 3
```


## SMEMBERS

- **Syntax**: `SMEMBERS key`
- **Description**: Lists all members of set named `key`
- **Return**: All members of set, (empty array) if `key` does not exist.

```sh
> SMEMBERS letters
 1) "x"
 2) "d"
 3) "f"
 4) "a"
 5) "e"
 6) "c"
 7) "z"
 8) "b"
 9) "g"
10) "y"
```

## SCARD

- **Syntax**: `SCARD key`
- **Description**: Returns count of set named `key`.
- **Return**: Numbers of set's count, 0 if `key` does not exist.

```sh
> SCARD letters
(integer) 10

> SCARD abcset
(integer 0)
```

## SPOP

- **Syntax**: `SPOP key [count]`
- **Description**: Gets and **remove** a member **randomly** from set named `key`.
- **Return**: Members of set.
    - If count is specified, return count member.
    - If count greater than or equal to `(>=)` actual number of set's members, it will return all members and clear them.
    - Nil if `key` does not exist.

```sh
> SPOP letters
1) "d"

> SPOP letters 100
 1) "x"
 2) "f"
 3) "z"
 4) "a"
 5) "e"
 6) "c"
 7) "b"
 8) "g"
 9) "y"

> SPOP abcset
(nil)
```

## SRANDMEMBER

- **Syntax**: `SRANDMEMBER key [count]`
- **Description**: Gets a member **randomly** from set named `key`. **This will not remove any members.**
- **Return**: Members of set.
    - If count is specified, return count member.
    - If count greater than or equal to `(>=)` actual number of set's members, it will return all members.
    - Nil if key does not exist.

```sh
> SRANDMEMBER letters
1) "e"

> SRANDMEMBER letters 100
 1) "x"
 2) "f"
 3) "z"
 4) "a"
 5) "e"
 6) "c"
 7) "b"
 8) "g"
 9) "y"
10) "d"

> SRANDMEMBER abcset
(nil)
```


## SREM

- **Syntax**: `SREM key member [member ...]`
- **Description**: Removes the `member` from set.
- **Return**: 1 if remove successful, 0 if `member` or `key` does not exist.

```sh
> SMEMBERS age
1) "18"
2) "20"
3) "15"

> SREM age 15
(integer) 1

> SMEMBERS age
1) "18"
2) "20"

> SREM age 15
(integer) 0

> SREM agese 15
(integer) 0

```

## SMOVE

- **Syntax**: `SMOVE source destination member`
- **Description**: Moves the `member` from `source` to `destination`. The `source` and `destination` required same type.
- **Return**: 1 if move successful, 0 if `source` or `destination` or `member` does not exist.

```sh
> SMEMBERS name
1) "Boii"
2) "Alice"
3) "Eva"

> SMEMBERS age
1) "15"
2) "18"
3) "20"

> SMOVE name age Eva
(integer) 1

> SMEMBERS name
1) "Boii"
2) "Alice"

> SMEMBERS age
1) "18"
2) "20"
3) "15"
4) "Eva"

> smove name age abc
(integer) 0
> smove name ageset abc
(integer) 0
> smove nameset age abc
(integer) 0
```

## SISMEMBER

- **Syntax**: `SISMEMBER key member`
- **Description**: Returns whether `member` is a member of the set.
- **Return**:
    - 1 if the `member` is a member of set.
    - 0 if the `member` is not a number of set.
    - 0 if `key` does not exist.

```sh
> SISMEMBER letters a
(integer) 1

> SISMEMBER letters k
(integer) 0

> SISMEMBER abcset a
(integer) 0
```


## Set operations

### SINTER

- **Syntax**: `SINTER key1 [key2 ...]`
- **Description**: Intersects set `key1` and others set.
- **Return**: 
    - All members that exist in both sets.
    - (empty array) if any key does not exits.
    - All members of `key1` if only given `key1`

```sh
> SADD s1 a b c d e f g
(integer) 7

> SADD s2 g h i
(integer) 3

> SADD s3 j k l a b
(integer) 5

> SINTER s1 s3
1) "b"
2) "a"
```

### SUNION

- **Syntax**: `SUNION key1 [key2 ...]`
- **Description**: Combines set `key1` and others set.
- **Return**: 
    - All members of all sets but not duplicated.
    - (empty array) if any key does not exits.
    - All members of `key1` if only given `key1`

```sh
> SADD s1 a b c d e f g
(integer) 7

> SADD s2 g h i
(integer) 3

> SADD s3 j k l a b
(integer) 5

> SUNION s2 s3
1) "g"
2) "b"
3) "k"
4) "h"
5) "a"
6) "l"
7) "j"
8) "i"
```

### SDIFF

- **Syntax**: `SDIFF key1 [key2 ...]`
- **Description**: Subtracts others set from set `key1`.
- **Return**: 
    - All member that only exist in set `key1`.
    - (empty array) if any key does not exits.
    - All members of `key1` if only given `key1`

```sh
> SADD s1 a b c d e f g
(integer) 7

> SADD s2 g h i
(integer) 3

> SADD s3 j k l a b
(integer) 5

> SDIFF s1 s3
1) "c"
2) "g"
3) "f"
4) "d"
5) "e"

> SDIFF s1 s3 s2
1) "c"
2) "e"
3) "f"
4) "d"
```

### SINTERSTORE

- **Syntax**: `SINTERSTORE destination key1 [key2 ...]`
- **Description**: Intersect sset `key1` and others set, than store the result into a new set `destination`. You can modify the set by specifying `destination` as one of the keys.
- **Return**: 
    - All members that exist in both sets.
    - (empty array) if any key does not exits.
    - All members of `key1` if only given `key1`



### SUNIONSTORE

- **Syntax**: `SUNIONSTORE destination key1 [key2 ...]`
- **Description**: Combines set `key1` and others set, than store the result into a new set `destination`.  You can modify the set by specifying `destination` as one of the keys.
- **Return**: 
    - All members of all sets but not duplicated.
    - (empty array) if any key does not exits.
    - All members of `key1` if only given `key1`



### SDIFFSTORE

- **Syntax**: `SDIFFSTORE destination key1 [key2 ...]`
- **Description**: Subtracts others set from set `key1`, than store the result into a new set `destination`.  You can modify the set by specifying `destination` as one of the keys.
- **Return**: 
    - All member that only exist in set `key1`.
    - (empty array) if any key does not exits.
    - All members of `key1` if only given `key1`


