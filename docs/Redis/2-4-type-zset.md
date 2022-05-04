# Type: ZSet

ZSet is an disorderly, unrepeatable type, but it can be ordered by score.

- When you add a member of Set, you need execute command like:

    ```sh
    > SADD setName member [member ...]
    ```

- When you add a member of ZSet, you need bring the `score` field. Execute command like:

    ```sh
    > ZADD zsetName score member [score member ...]
    ```

The key words of type ZSet are **Can be ordered**. So it is suitable for `Rank List` scenarios or other about **rank**.

The unit of ZSet is score-member pair. Score means weights and used for sorting.

The members are unique. It is the main field. Members can not be duplicated.

And score is not unique. It is more like an auxiliary field. 

By default, ZSet sort by score. Elements with the same score are ordered by lexicographically.



## ZADD

- **Syntax**: `ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]`
- **Description**: Adds a score-member pair into zset which named `key`. The ZSet will be created if it does not exist.
    - **NX | XX**:
        - **NX**: Only ==add== new elements. Don't update already existing elements.
        - **XX**: Only ==update== elements that already exist. Don't add new elements.
    - **GT | LT**:
        - **GT**: Only ==update== existing elements if the new score is ==^^greater than^^== the current score. This flag doesn't prevent adding new elements.
        - **LT**: Only ==update== existing elements if the new score is ==^^less than^^== the current score. This flag doesn't prevent adding new elements.
    - **CH**: When use this option, return value means ==total number of elements CHanged (^^new elements added + the score was updated^^)==. Return value means ==number of elements added== by default.
    - **INCR**: When this option is specified ZADD acts like ZINCRBY. Only one score-element pair can be specified in this mode.
- **Return**: Number of successful added.

!!! note
    - Add action: {++NX++}
    - Update action: {++XX, GT, LT++}
        - prevent adding: **XX**
        - doesn't prevent adding: **GT, LT**

??? example 
    ```sh title="Adding"
    > ZADD letters 1 a 2 b 3 c 4 d 26 z
    (integer) 5

    > ZRANGE letters 0 -1
    1) "a"
    2) "b"
    3) "c"
    4) "d"
    5) "z"

    > ZRANGE letters 0 -1 WITHSCORES
    1) "a"
    2) "1"
    3) "b"
    4) "2"
    5) "c"
    6) "3"
    7) "d"
    8) "4"
    9) "z"
    10) "26"
    ```


    ```sh title="Only adding, update action will not be executed"
    > ZADD letters NX 10 a 20 b 24 x 25 y
    (integer) 2
    > ZRANGE letters 0 -1 WITHSCORES
     1) "a"
     2) "1"
     3) "b"
     4) "2"
     5) "c"
     6) "3"
     7) "d"
     8) "4"
     9) "x"
    10) "24"
    11) "y"
    12) "25"
    13) "z"
    14) "26"
    ```

    ```sh title="Only Update, adding action will not be executed"
    > ZADD letters XX CH 10 a 20 b 24 x 25 y  # (1)
    (integer) 2

    > ZRANGE letters 0 -1 WITHSCORES
     1) "c"
     2) "3"
     3) "d"
     4) "4"
     5) "a"
     6) "10"
     7) "b"
     8) "20"
     9) "x"
    10) "24"
    11) "y"
    12) "25"
    13) "z"
    14) "26"
    ```

    1. CH flag means return CHanged numbers.

## Length

### ZCARD

- **Syntax**: `ZCARD key`
- **Description**: Returns the number of elements in the ZSet named `key`
- **Return**: Number of elements, 0 if key is not exist.

```sh
> ZRANGE letters 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"

> ZCARD letters
(integer) 5

> ZCARD not-such-key
(integer) 0
```

### ZCOUNT

- **Syntax**: `ZCOUNT key min max`
- **Description**: Return the number of elements in the ZSet at `key` with **score** between `min` and `max`.
    - The `min` and `max` have the same semantic as describe for [ZRANGE](#zrange)'s `Score range`.

```sh
> ZRANGE letters 0 -1 WITHSCORES
 1) "a"
 2) "11"
 3) "b"
 4) "22"
 5) "c"
 6) "33"
 7) "d"
 8) "44"
 9) "e"
10) "55"

> ZCOUNT letters -inf +inf
(integer) 5

> ZCOUNT letters 20 +inf
(integer) 4
```


### ZLEXCOUNT

- **Syntax**: `ZLEXCOUNT key min max`
- **Description**: Return the number of elements in the ZSet at `key` with **lexicographical** between `min` and `max`.
    - The `min` and `max` have the same semantic as describe for [ZRANGE](#zrange)'s `Lexicographical range`.

```sh
> ZRANGE letters 0 -1 WITHSCORES
 1) "a"
 2) "11"
 3) "b"
 4) "22"
 5) "c"
 6) "33"
 7) "d"
 8) "44"
 9) "e"
10) "55"

> ZLEXCOUNT letters - +
(integer) 5

> ZLEXCOUNT letters [b +
(integer) 4

> ZLEXCOUNT letters (b +
(integer) 3
```

---

## List
### ZRANGE

ZRANGE can perfrom different types of range queries: by index(rank), by score, by lexicographical order.

- **Syntax**: `ZRANGE key min max [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]`
- **Description**: Lists the elements within the specified MIN and MAX ranges.
    - The ranges ==includes== minimum element and maximum element, equal to the ==start-with and end-with==. 
    - **WITHSCORES**: Lists members with score. Only members are listed by default.
    - **BYSCORE**: Lists members from min to max by score
    - **BYLEX**: Lists members from min to max ly lexicographical order.
    - **REV**: Reverses the result. The elements are ordered from low to high score by default.
    - **LIMIT**: Same like SELECT LIMIT offset count. It needs specified BYSCORE or BYLEX. 
        A negative `count` will returns all members from `offset`. 

    - **Index range**:
        - **min | max**: By default, the command perfroms an index range query.
            - if min is greater than either the end index of the ZSet or max, an empty list is returns.
            - if max is greater than the end index of the ZSet, the index of last element will be used.
            
            - `ZRANGE key 0 -1` equal `ZRANGE key 0 index-of-last-element`
            - `ZRANGE key 0 -2` equal `ZRANGE key 0 index-of-penultimate-element`
            - `ZRANGE key 11 5` return `(empty list)`
            - `ZRANGE key 0 999` return `0 <= index <= last-element`
            - `ZRANGE key (1 5` return `1 < index <= 5`
            - `ZRANGE key 1 (5` return `1 <= index < 5`
            - `ZRANGE key (1 (5` return `1 < index < 5`

    - **Score range**:
        - **min | max**: When the BYSCORE option is used, `-inf` and `+inf` is vaild.
            - `-inf` and `+inf` is a built-in variable mean infinite.

            - if `min` is greater than either the highest score of the ZSet or `max`, an empty list is returns.
            - if `max` is greater than the highest score of the ZSet, the score of highest score will be used.

            - `ZRANGE key 0 -1 BYSCORE` equal `ZRANGE key 0 index-of-last-element BYSCORE`
            - `ZRANGE key 0 -2 BYSCORE` equal `ZRANGE key 0 index-of-penultimate-element BYSCORE`
            - `ZRANGE key -inf +inf BYSCORE` return all element
            - `ZRANGE key 3 +inf BYSCORE` return `3 <= score <= highest-score`
            - `ZRANGE key 5 -inf BYSCORE` return `(empty list)` because `min` is greater than `max`

    - **Lexicographical range**:
        - **start | stop**: 
            - When the BYLEX option is used, `start` and `stop` must start with `(` or `[` to specify whether the range interval is exclusive or inclusive.
                - `ZRANGE key (a [e BYLEX` return `b c d e`
                - `ZRANGE key a [e BYLEX` or `ZRANGE key [a e BYLEX` is invaild.
            - The special value of `+` or `-` for `start` or `stop` mean positive or negative infinite strings.
                - `ZRANGE key - + BYLEX` return `a b c d e`
                - `ZRANGE key + - BYLEX REV` return `e d c b a`

  

- Conclusion - Min and Max:

    - In a **index range** query, it is in the same way as Python index: 
        - 0 means First element, 1 means Second element, 
        - -1 means Last element, -2 means Penultimate element, and so on.
    - In a **score range** query, `-inf` and `+inf` is {++vaild++}, then `-` and `+` is {++invaild++}.
    - In a **lexicographical range** query, `-` and `+` is {++vaild++}, then `-inf` and `+inf` is {++invaild++}. And it ==must== start with `(` or `[`, except `-` and `+`.

??? example
    
    ```sh title="Normal"
    > ZRANGE letters 0 -1
    127.0.0.1:6379> ZRANGE letters 0 -1
     1) "a"
     2) "b"
     3) "c"
     4) "d"
     5) "e"
     6) "f"
     7) "g"
     8) "h"
     9) "i"
    10) "j"
    11) "x"
    12) "y"
    13) "z"
    ```

    ```sh title="WITHSCORES"
    > ZRANGE letters 0 -1 WITHSCORES
     1) "a"
     2) "1"
     3) "b"
     4) "2"
     5) "c"
     6) "4"
     7) "d"
     8) "4"
     9) "e"
    10) "5"
    11) "f"
    12) "6"
    13) "g"
    14) "7"
    15) "h"
    16) "8"
    17) "i"
    18) "9"
    19) "j"
    20) "10"
    21) "x"
    22) "24"
    23) "y"
    24) "25"
    25) "z"
    26) "26" 
    ```

    ```sh title="BYSCORE and LIMIT"
    > ZRANGE letters -inf +inf BYSCORE LIMIT 2 3
    1) "c"
    2) "d"
    3) "e"
    ```

    ```sh title="BYLEX"
    > ZRANGE letters - [g BYLEX
    1) "a"
    2) "b"
    3) "c"
    4) "d"
    5) "e"
    6) "f"
    7) "g"
    ```

    ```sh title="REV"
    > ZRANGE letters [x [g BYLEX REV
    1) "x"
    2) "j"
    3) "i"
    4) "h"
    5) "g"

    > ZRANGE letters - + BYLEX
     1) "a"
     2) "b"
     3) "c"
     4) "d"
     5) "e"
     6) "f"
     7) "g"
     8) "h"
     9) "i"
    10) "j"
    11) "x"
    12) "y"
    13) "z"
    ```

#### ZRANGEBYSCORE

- **Syntax**: `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
- **Description**: Equal to `ZRANGE key min max BYSCORE`

```sh 
> ZRANGEBYSCORE letters 0 -1 

```
#### ZRANGEBYLEX

- **Syntax**: `ZRANGEBYLEX key min max [LIMIT offset count]`
- **Description**: Equal to `ZRANGE key min max BYLEX`

#### ZREVRANGE

- **Syntax**: `ZREVRANGE key start stop [WITHSCORES]`
- **Description**: Equal to `ZRANGE key min max REV`

#### ZREVRANGEBYSCORE

- **Syntax**: `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`
- **Description**: Equal to `ZRANGE key min max BYSCORE REV`

#### ZREVRANGEBYLEX

- **Syntax**: `ZREVRANGEBYLEX key max min [LIMIT offset count]`
- **Description**: Equal to `ZRANGE key min max BYLEX REV`


#### ZRANGESTORE

- **Syntax**: `ZRANGESTORE dst src min max [BYSCORE|BYLEX] [REV] [LIMIT offset count]`
- **Description**: Equal to `ZRANGE key min max `

## Get
### ZRANK

- **Syntax**: `ZRANK key member`
- **Description**: Returns the index of the member in key. Nil when the member is not exist.

```sh 
> ZRANGE letters 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"

> ZRANK letters b
(integer) 1

> ZRANK letters e
(integet) 4

> ZRANK letters g
(nil)
```

### ZREVRANK

- **Syntax**: `ZREVRANK key member`
- **Description**: Returns the index of the member after `key` is reversed. Nil whene the member is not exist.

```sh
> ZRANGE letters 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"

> ZREVRANK letters a
(integer) 4

> ZREVRANK letters d
(integer) 1

> ZREVRANK letters x
(nil)
```

### ZSCORE
- **Syntax**: `ZSCORE key member`
- **Description**: Returns the score of the member in key. Nil when the member is not exist.

```sh 
> ZRANGE letters 0 -1 WITHSCORES
 1) "a"
 2) "11"
 3) "b"
 4) "22"
 5) "c"
 6) "33"
 7) "d"
 8) "44"
 9) "e"
10) "55"


> ZSCORE letters b
(integer) 22

> ZSCORE letters e
(integet) 55

> ZSCORE letters g
(nil)
```

### ZMSCORE

- **Syntax**: `ZMSCORE key member [member ...]`
- **Description**: Returns the scores associated with the specified members in the ZSet named `key`.
- **Return**: Every member's score. Nil if any member is not exist.

??? example

    ```sh
    > ZRANGE letters 0 -1 WITHSCORES
    1) "a"
    2) "11"
    3) "b"
    4) "22"
    5) "c"
    6) "33"
    7) "d"
    8) "44"
    9) "e"
    10) "55"

    > ZMSCORE zsets a b c
    1) "11"
    2) "22"
    3) "33"

    > ZMSCORE zsets a b c no-such
    1) "11"
    2) "22"
    3) "33"
    4) (nil)
    ```


### ZPOPMIN

- **Syntax**: `ZPOPMIN key [count]`
- **Description**: Removes and returns up to `count` members with the lowest scores in the ZSet named `key`. `Count` is 1 by default.

??? example

    ```sh
    > ZRANGE letters 0 -1 WITHSCORES
    1) "a"
    2) "11"
    3) "b"
    4) "22"
    5) "c"
    6) "33"
    7) "d"
    8) "44"
    9) "e"
    10) "55"

    > ZPOPMIN letters 1
    1) "a"
    2) "11"

    > ZRANGE letters 0 -1 WITHSCORES
    1) "b"
    2) "22"
    3) "c"
    4) "33"
    5) "d"
    6) "44"
    7) "e"
    8) "55"

    > ZPOPMIN letters 3
    1) "b"
    2) "22"
    3) "c"
    4) "33"
    5) "d"
    6) "44"

    > ZRANGE letters 0 -1 WITHSCORES
    1) "e"
    2) "55"
    ```

### ZPOPMAX

- **Syntax**: `ZPOPMAX key [count]`
- **Description**: Removes and returns up to `count` members with the highest scores in the ZSet named `key`. `Count` is 1 by default.

??? example

    ```sh
    > ZRANGE zsets 0 -1 WITHSCORES
    1) "a"
    2) "11"
    3) "b"
    4) "22"
    5) "c"
    6) "33"
    7) "d"
    8) "44"
    9) "e"
    10) "55"

    > ZPOPMAX zsets
    1) "e"
    2) "55"

    > ZRANGE zsets 0 -1 WITHSCORES
    1) "a"
    2) "11"
    3) "b"
    4) "22"
    5) "c"
    6) "33"
    7) "d"
    8) "44"

    > ZPOPMAX zsets 2
    1) "d"
    2) "44"
    3) "c"
    4) "33"

    > ZRANGE zsets 0 -1 WITHSCORES
    1) "a"
    2) "11"
    3) "b"
    4) "22"
    ```


### ZRANDMEMBER

- **Syntax**: `ZRANDMEMBER key [count [WITHSCORES]]`
- **Description**: Returns a random element from the ZSet named `key`. `Count` is 1 by default.

```sh
> ZADD dadi 1 uno 2 due 3 tre 4 quattro 5 cinque 6 sei
(interger) 6

> ZRANDMEMBER dadi
"cinque"

> ZRANDMEMBER dadi
"due"

> ZRANDMEMBER dadi -5 WITHSCORES
1) "due"
2) "2"
3) "tre"
4) "3"
5) "quattro"
6) "4"
7) "quattro"
8) "4"
9) "quattro"
10) "4"
```



## Delete

### ZREM

- **Syntax**: `ZREM key member [member ...]`
- **Description**: Removes the special members from ZSet named `key`.
- **Return**: 1 if removed successful, 0 either removed failed or members or key are not exist. error when `key` is not a ZSet.

```sh
> ZRANGE letters 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"

> ZREM letters b
(integer) 1

> ZRANGE letters 0 -1
1) "a"
2) "c"
3) "d"
4) "e"

> ZREM list-type a 
(error) WRONGTYPE Operation against a key holding the wrong kind of value
```


#### ZREMRANGEBYLEX

- **Syntax**: `ZREMRANGEBYLEX key min max`
- **Description**: Ranges the elements from `min` to `max` then removes them. The `min` and `max` have the same semantic as describe for [ZRANGE](#zrange)'s `Lexicographical range`.

```sh
> ZRANGE letters 0 -1
1) "ALPHA"
2) "aaaa"
3) "alpha"
4) "b"
5) "c"
6) "d"
7) "e"
8) "foo"
9) "zap"
10) "zip"

> ZREMRANGEBYLEX letters [alpha [omega
(interger) 6

> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "zap"
4) "zip"
```


#### ZREMRANGEBYSCORE

- **Syntax**: `ZREMRANGEBYSCORE key min max`
- **Description**: Ranges the elements from `min` to `max` then removes them. The `min` and `max` have the same semantic as describe for [ZRANGE](#zrange)'s `Score range`.

```sh
> ZRANGE letters 0 -1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"

> ZREMRANGEBYSCORE letters -inf (2
(interger) 1

> ZRANGE letters 0 -1 WITHSCORES
1) "two"
2) "2"
3) "three"
4) "3"
```

#### ZREMRANGEBYRANK

- **Syntax**: `ZREMRANGEBYRANK key start stop`
- **Description**: Ranges the elements from `min` to `max` then removes them. The `min` and `max` have the same semantic as describe for [ZRANGE](#zrange)'s `Index range`.

```sh
> ZRANGE letters 0 -1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"

> ZREMRANGEBYRANK letters 1 2
(interger) 1

> ZRANGE letters 0 -1 WITHSCORES
1) "one"
2) "1"
```





## Calculation
### ZINCRBY

- **Syntax**: `ZINCRBY key increment member`
- **Description**: Increments the score of `member` in the ZSet named `key`.

??? example

    ```sh
    > ZRANGE letters 0 -1 WITHSCORES
    1) "a"
    2) "1"
    3) "b"
    4) "2"
    5) "c"
    6) "3"
    7) "d"
    8) "4"
    9) "e"
    10) "5"

    > ZINCRBY letters 1 a
    "2"

    > ZRANGE letters 0 -1 WITHSCORES
    1) "a"
    2) "2"
    3) "b"
    4) "2"
    5) "c"
    6) "3"
    7) "d"
    8) "4"
    9) "e"
    10) "5"
    ```

    ```sh title="Increment a not exist member"
    > ZINCRBY letters 15 z
    "15"
    
    > ZRANGE letters 0 -1 WITHSCORES
     1) "a"
     2) "2"
     3) "b"
     4) "2"
     5) "c"
     6) "3"
     7) "d"
     8) "4"
     9) "e"
    10) "5"
    11) "z"
    12) "15"
    ```
### ZUNION


### ZUNIONSTORE

### ZINTER

### ZINTERSTORE

### ZINTERCARD

### ZDIFF

### ZDIFFSTORE