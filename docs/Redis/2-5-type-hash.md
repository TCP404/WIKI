# Type: Hash

## Set
### HSET 

- **Syntax**: `HSET key field value [field value ...]`
- **Description**: Set one or more pair of field-value.

```sh
> HSET m1 name Boii
(integer) 1

> HGET m1 name
"Boii"
```

### HSETNX

- **Syntax**: `HSETNX key field value`
- **Description**: Set a pair of field-value **if key is not exist**.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HGET m1 name
"Boii"

> HSETNX m1 name Eva
(integer) 0

> HSETNX m1 addr SWT
(integer) 1

> HKEYS m1
1) "name"
2) "age"
3) "bir"
4) "addr"
```

### HMSET

- **Syntax**: `HMSET key field value [field value ...]`
- **Description**: Set more pair of field-value.

```sh
> HMSET m1 name Boii age 18
OK

> HGET m1 name
"Boii"

> HGET m1 age
"18"
```

## Get
### HGET

- **Syntax**: `HGET key field`
- **Description**: Gets the value of field at Hash named `key`.

```sh
> HMSET m1 name Boii age 18
OK

> HGET m1 name
"Boii"

> HGET m1 age
"18"
```

### HMGET

- **Syntax**: `HMGET key field [field ...]`
- **Description**: Gets more values of fields at Hash named `key`.

```sh
> HMSET m1 name Boii age 18
OK

> HMGET m1 name age
"Boii"
"18"
```

### HGETALL

- **Syntax**: `HGETALL key`
- **Description**: Gets all pair of field-value at Hash named `key`.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HGETALL m1
1) "name"
2) "Boii"
3) "age"
4) "18"
5) "bir"
6) "2022-01-01"
```

### HRANDFIELD

- **Syntax**: `HRANDFIELD key [count [WITHVALUES]]`
- **Description**: Returns a random key from the Hash named `key`. `Count` is 1 by default.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HRANDFIELD m1 5
1) "name"
2) "age"
3) "bir"

> HRANDFIELD m1 5 WITHVALUES
1) "name"
2) "Boii"
3) "age"
4) "18"
5) "bir"
6) "2022-01-01"
```

### HKEYS

- **Syntax**: `HKEYS key`
- **Description**: Returns all fields of the Hash named `key`.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HKEYS m1
1) "name"
2) "age"
3) "bir"
```

### HVALS

- **Syntax**: `HVALS key`
- **Description**: Returns all values of the Hash named `key`.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HVALS m1
1) "Boii"
2) "18"
3) "2022-01-01"
```

### HEXISTS

- **Syntax**: `HEXISTS key field`
- **Description**: Returns whether the field exists in Hash named `key`.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HEXISTS m1 name
(integer) 1

> HEXISTS m1 Boii
(integer) 0
```

## Del 
### HDEL

- **Syntax**: `HDEL key field [field ...]`
- **Description**: Removes one or more pair field-value from the Hash named `key`.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HDEL m1 name
(integer) 1

> HKEYS m1 
1) "age"
2) "bir"
```

## Length
### HLEN

- **Syntax**: `HLEN key`
- **Description**: Returns the number of field-value in Hash named `key`.

```sh
> HMSET m1 name Boii age 18 bir 2022-01-02
OK

> HLEN m1
(integer) 3
```
### HSTRLEN

- **Syntax**: `HSTRLEN key field`
- **Description**: Returns the length of field's value in Hash named `key`.

```sh
> HMSET m1 name Boii age 18
OK

> HSTRLEN m1 name
(integer) 4

> HSTRLEN m1 age
(integer) 2
```

## Calculation

### HINCRBY

- **Syntax**: `HINCRBY key field increment`
- **Description**: Increases increment from a numeric value, similar `key.field++`.

```sh
> HMSET m1 name Boii age 18
OK

> HINCRBY m1 age 2
(integer) 20

> HGET m1 age
"20"
```

### HINCRBYFLOAT

- **Syntax**: `HINCRBYFLOAT key field increment`
- **Description**: Increases a float value to a numeric value.

```sh
> HSET m1 age 10
OK

> HINCRBYFLOAT m1 age 2.2
"12.2"
```

