# Script

有时候简单的查询或者更新不能满足我们的需求，所以 ES 提供了 Script 让我们可以在操作的时候使用简单的脚本。

Script 模板如下：

```json
{
    "script": {
        "lang": "<painless | expression>",   // (1)
        "source | id": "<SCRIPT | SCRIPT_STORE_NAME>", // (2)
        "params": { // (3)
            "<PARAM>": <VALUE>,
            ...
        }
    }
}
```

1. lang: 指定脚本语言，这个字段不是必须的，默认值为 ES 官方的脚本语言 painless
2. source: 这里是脚本的内容，也可以是已经存储起来的脚本 stored script 的名称
3. params: 这里是参数对象，参数的值会赋值到脚本中，这样就可以在脚本中使用参数。

!!! example
    ```http
    GET /product/_search HTTP/1.1
    Content-Type: application/json

    {
        "query": {
            "bool": {
                "filter": [
                    {
                        "script": {
                            "script": {
                                "lang": "painless",
                                "source": "doc['create_time'].value.dayOfMonth == params.day",
                                "params": {"day": 1 }
                            }
                        }
                    }
                ]
            }
        }
    }
    ```

---

脚本有两种：一种是直接写的 `inline script`；一种是脚本模板 `stored script`。

## inline script

```http
PUT /product/_update/1 HTTP/1.1
Content-Type: application/json

{
    "script": {
        "source": "ctx._source.price += 10"
    }
}
```
上面的栗子用于更新 product 这个索引中 _id 为 1 的文档，其使用了脚本的方式将 price 字段自增了 10。

也可以使用参数方式指定自增多少，这样我们可以在编程语言中动态的指定自增 10 还是 20 还是多少都可以。

```http
PUT /product/_update/1 HTTP/1.1
Content-Type: application/json

{
    "script": {
        "source": "ctx._source.price += params.step",
        "params": {"step": 20}
    }
}
```


## stored script

除了把模板写在 source 中，我们还可以预先把模板存储起来。

在需要的地方把 `"source": "SCRIPT"` 换成 `"id": "STORED SCRIPT NAME"`

### 定义 stored 脚本
```json
PUT /_scripts/<STORED SCRIPT NAME>
{
    "script": {
        "lang": "<painless | expression>", // (1)
        "source": "<SCRIPT>"
    }
}
```

1. 这里 lang 并不是必须的

!!! example
    ```http
    PUT {{host}}/_scripts/add_price HTTP/1.1
    Content-Type: application/json

    {
        "script": {
            "lang": "painless",
            "source": "ctx._source.price += params.step"
        }
    }
    ```

### 使用 stored 脚本

```json
PUT /product/_update/1
{
    "script": {
        "id": "add_price",  // (1)
        "params": {"step": 10}  // (2)
    }
}
```

1. 这里使用的是模板脚本存储时的名称,注意键用的是 id 不是 source
2. 模板定义时带有参数，所以这里需要传递参数

### 查看模板
```http
GET /_scripts/<STORED SCRIPT NAME>
```

### 删除模板
```http
DELETE /_scripts/<STORED SCRIPT NAME>
```


## 常用模板

### 1. 给已有的文档添加新字段
```http
PUT /people/_update_by_query HTTP/1.1
Content-Type: application/json

{
    "script: {
        "source": "if (ctx._source.hobby == nill) {ctx._source.hobby = ['watch TV', 'coding']}"
    }
}
```
给 people 索引所有文档添加一个 hobby 字段并赋值。

### 2. 移除数组字段某一项

```http
PUT /people/_update/1 HTTP/1.1
Content-Type: application/json

{
    "script": {
        "source": "if (ctx._source.hobby.contains(params.hobbyName)) {ctx._source.hobby.remove(ctx._source.hobby.indexOf(params.hobbyName))}",
        "params": {"hobbyName": "watch TV"}
    }
}
```
先判断数组是否包含 hobbyName 这个值，包含则通过数组索引删除。也可以换成以下写法：

```http
PUT /people/_update/1 HTTP/1.1
Content-Type: application/json

{
    "script": {
        "source": "ctx._source.hobby.removeIf(it -> it == params.hobbyName);",
        "params": {"hobbyName": "watch TV"}
    }
}
```
除了 removeIf(), 还有以下这些：
```java
list.removeIf(item -> item == 2);
list.removeIf((int item) -> item == 2);
list.removeIf((int item) -> { item == 2 });
list.sort((x, y) -> x - y);
list.sort(Integer::compare);
list.sort(this::mycompare);
```

### 3. 自定义返回脚本

=== "Req"
    ```http
    POST /people/_search HTTP/1.1
    Content-Type: application/json
    
    {
        "script_fields": {
            "year-asset": {
                "script": {
                    "lang": "painless",
                    "source": "doc['asset'].value * params.value",
                    "params": {"value": 12}
                }
            },
            "birthday-year":{
                "script":{
                    "lang":"painless",
                    "source":"doc.birthday.value.year"
                }
            }
        }
    }
    ```

    "Resp"
    ```http
    {
        "took": 8,
        "timed_out": false,
        "_shards": {
            "total": 3,
            "successful": 3,
            "skipped": 0,
            "failed": 0
        },
        "hits": {
            "total": {
                "value": 1,
                "relation": "eq"
            },
            "max_score": 1.2039728,
            "hits": [
                {
                    "_index": "people",
                    "_type": "_doc",
                    "_id": "4",
                    "_score": 1.2039728,
                    "fields": {
                        "year-asset": [
                            226656.0
                        ],
                        "birthday-year": [
                            1993
                        ]
                    }
                }
            ]
        }
    }
    ```

### 4. 自定义脚本排序
```http
POST /people/_search HTTP/1.1
Content-Type: application/json

{
    "query": {
        "match_all": {}
    },
    "sort": {
        "_script": {
            "type": "number",
            "order": "desc",
            "script":{
                "lang": "painless",
                "source": "doc['asset'].value * doc['age'].value"
            }          
        }
    }
}
```
将 asset 和 age 相乘后再排序。

### 5. 查询脚本过滤

```http
POST  http://172.16.1.236:9201/people/_search
{
    "query": {
        "bool": {
            "filter": {
                    "script": {
                        "script": {
                            "lang": "painless",
                             "source": "doc['age'].value > 25"
                        }
                    }
            }
        }
    }
}
```
只查询年龄大于 25 的

## Lucene expression 语言的一些表达式
### 1. 数值字段表达式

```java
doc['field_name'].value     //字段的值，作为 double
doc['field_name'].empty     //一个布尔值，指示该字段在doc中是否没有值。
doc['field_name'].length    //本文档中值的数量。
doc['field_name'].min()     //本文档中字段的最小值。
doc['field_name'].max()	    //本文档中字段的最大值。
doc['field_name'].median()  //本文档中字段的中间值。
doc['field_name'].avg()     //本文档中值的平均值。
doc['field_name'].sum()     //本文档中值的总和。
```

当文档完全缺少该字段时，默认情况下该值将被视为0。您可以将其视为另一个值，例如 `doc['myfield'].empty ? 100 : doc['myfield'].value`。

### 2. 日期字段表达式
日期字段被视为自1970年1月1日以来的毫秒数

```java
doc['field_name'].date.centuryOfEra         // 世纪（1-2920000）
doc['field_name'].date.dayOfMonth           // 第一天（1-31），例如1每月的第一天。
doc['field_name'].date.dayOfWeek            // 星期几（1-7），例如1星期一。
doc['field_name'].date.dayOfYear            // 一年中的一天，例如11月1日。
doc['field_name'].date.era                  // 时代：0公元前，1公元前。
doc['field_name'].date.hourOfDay            // 小时（0-23）。
doc['field_name'].date.millisOfDay          // 一天中的毫秒数（0-86399999）。
doc['field_name'].date.millisOfSecond       // 秒内的毫秒数（0-999）。
doc['field_name'].date.minuteOfDay          // 在一天中的分钟（0-1439）。
doc['field_name'].date.minuteOfHour         // 一小时内的分钟（0-59）。
doc['field_name'].date.monthOfYear          // 一年中的月份（1-12），例如1一月。
doc['field_name'].date.secondOfDay          // 一天中的第二个（0-86399）。
doc['field_name'].date.secondOfMinute       // 分钟内（0-59）秒。
doc['field_name'].date.year	                // 年（-292000000-292000000）。
doc['field_name'].date.yearOfCentury        // 本世纪内的年份（1-100）。
doc['field_name'].date.yearOfEra            // 时代以内（1-292000000）。
```

### geo_point 类型表达式
```java
doc['field_name'].empty     //一个布尔值，指示该字段在doc中是否没有值。
doc['field_name'].lat       //地理位置的纬度。
doc['field_name'].lon       //地理位置的经度。
```