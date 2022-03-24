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

The score is not unique. It is more like an auxiliary field. 

And members are unique. It is the main field. Members can not be duplicated.

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


ZCOUNT
ZCARD

---

## ZRANGE

- **Syntax**: `ZRANGE key min max [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]`
- **Description**: Lists the elements within the specified MIN and MAX ranges.
    - The ranges includes min element and max element, equal to the start-with and end-with. If -1 as MAX argument, it will take the actual max value as the MAX argument.
    - **min | max**: 
        - `-inf` and `+inf` is a built-in variable mean infinite.
        - `ZRANGE key 0 -1` equal `ZRANGE key 0 +inf`
        - `ZRANGE key (1 5` return `1 < score <= 5`
        - `ZRANGE key 1 (5` return `1 <= score < 5`
        - `ZRANGE key (1 (5` return `1 < score < 5`
    - **WITHSCORE**: Lists member and score. Only members are listed by default.
    - **BYSCORE**: 
    - **REV**: Reverses the result. The elements are ordered from low to high score by default.



ZRANGEBYSCORE
ZRANGEBYLEX
ZRANGESTORE

ZREVRANGE
ZREVRANGEBTSTORE
ZREVRANGEBYLEX

---
ZRANK
ZREVRANK

ZSCORE
---

ZREM
ZREMRANGEBYLEX
ZREMRANGEBYSCORE
ZREMRANGEBYRANK

ZINCRBY

ZUNION
ZUNIONSTORE
ZINTER
ZINTERSTORE
ZDIFF
ZDIFFSTORE