# Type: List

List is an orderly, repeatable, iterable type. It has LEFT and RIGHT directions and 8 manipulation, such as push, pop, range, set, delete, insert, specify index, trim.

!!! note "Specifically"

    The range consists of start and stop (or end), it means 'start with' and 'stop with', unlike more programing language.

    For example, in Python, `#!py list[3:6]` means `#!py list[start_with:stop]`, it will return `#!py list[3], list[4], list[5]` but NOT `#!py list[6]`.

    In Redis, `COMMAND list 3 6` means `#!py list[start_with:stop_with]`, it will return `#!py list[3], list[4], list[5]`, AND the `#!py list[6]`.

    The index is same with Python.

    |  element | a  | b  | c  | d  | e  | f  | g  |
    |----------|----|----|----|----|----|----|----|
    | positive | 0  | 1  | 2  | 3  | 4  | 5  | 6  |
    | negative | -7 | -6 | -5 | -4 | -3 | -2 | -1 |

## LPUSH 
**LPUSH**: means "push from left", in other words, add elements to list's head in order.

- **Syntax**: `LPUSH key element [element ...]`
- **Description**: add elements to list's head in order. The key will be created if it isn't exists

```sh
> LPUSH letters a b c d e

> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"
```

## RPUSH
**RPUSH**: in contrast, it means "push from right", in other words, add elements to list's tail in order.

- **Syntax**: `RPUSH key element [element ...]`
- **Description**: the key will be created if it isn't exists.

```sh
> RPUSH letters a b c d e

> LRANGE letters 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
```

## LPUSHX 
**LPUSHX**

- **Syntax**: `LPUSHX key element [element ...]`
- **Description**: add elements to list's head in order **if key is exists**.
- **Return**: length of list, 0 if key isn't exists.

## RPUSHX
**RPUSHX**

- **Syntax**: `RPUSHX key element [element ...]`
- **Description**: add elements to list's tail in order **if key is exists**.
- **Return**: length of list, 0 if key isn't exists.
 
## LPOP 
**LPOP**: means "pop from left", in other words, get elements from list's head in order.

- **Syntax**: `LPOP key [count]`
- **Description**: get count elements from list's head and **delete them**, count default 1.
- **Return**: count elements, nil if key isn't exists.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> LPOP letters 
"e"
> LPOP letters 3
"d"
"c"
"b"
```

## RPOP
**RPOP**: in contrast, it means "pop from right", in other words, get elements from list's tail in order.

- **Syntax**: `RPOP key [count]`
- **Description**: get count elements from list's tail and **delete them**, count default 1.
- **Return**: count elements, nil if key isn't exists.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> RPOP letters 
"a"
> RPOP letters 3
"b"
"c"
"d"
```

## LRANGE
**LRANGE**

- **Syntax**: `LRANGE key start end`
- **Description**: list the elements in the indicated range. -1 means until the end. More specifically, 'start' is 'start with', 'end' is 'end with'. The returns will include the 'end' elements.
- **Return**: elements in range, empty if key isn't exists or start index less than end index.

## LLEN
**LLEN**

- **Syntax**: `LLEN key`
- **Description**: return the length of key's list.
- **Return**: list's length, 0 if key isn't exists.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> LLEN letters
5
```

## LSET
**LSET**

- **Syntax**: `LSET key index element`
- **Description**: set the value to elements of key's list, index must be legal.
- **Return**: OK if successful, error if index is out of range or key isn't exists.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> LSET letters 2 abc
OK

> LRANGE letters 0 -1
1) "e"
2) "d"
3) "abc"
4) "b"
5) "a"
```

## LINDEX
**LINDEX**

- **Syntax**: `LINDEX key index`
- **Description**: get the element with index.
- **Return**: element, nil if index is out of range or key isn't exists.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> LINDEX letters 3
"b"
```

## LREM
**LREM**

- **Syntax**: `LREM key count element`
- **Description**: remove the duplicate elements of indicated element. 
    - if count is negative, it will remove indicated element from list's tail.
    - if count is positive, it will remove indicated element from list's head.
    - if count is 0 or greater than actual count, it will remove all of indicated element.
- Return: count of removed elements. 0 if key isn't exists or elements isn't exists in list.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"
6) "a"
7) "a"
# count is greater than actual count
> LREM letters 5 a
3
```

## LTRIM
**LTRIM**

- **Syntax**: `LTRIM key start stop`
- **Description**: LEAVE the elements in the range and REMOVE other elements. 
    - if start, except -1, is greater than stop, it will remove all elements.
    - it equal the end if stop is out of range.
- **Return**: OK if successful.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> LTRIM letters 1 3
OK

> LRANGE letters 0 -1
1) "d"
2) "c"
3) "b"
```

## LINSERT
**LINSERT**

- **Syntax**: `LINSERT key BEFORE|AFTER pivot element`
- **Description**: insert a element before or after the pivot element.
- **Return**: the length of list after insert. 0 if fail.

```sh
> LRANGE letters 0 -1
1) "e"
2) "d"
3) "c"
4) "b"
5) "a"

> LINSERT letters BEFORE c abc
6

> LRANGE letters 0 -1
1) "e"
2) "d"
3) "abc"
4) "c"
5) "b"
6) "a"
```









