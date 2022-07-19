# Query DSL

Query DSL (Query Domain Specified Language 特殊领域查询语言)，利用 REST API 传递 JSON 格式的请求数据与 ES 进行交互查询。这种方式的丰富查询方法让 ES 检索更强大、简洁。

## Syntax

```http
GET /<INDEX_NAME>/_doc/_search
{
    <DSL>
}


GET /<INDEX_NAME>/_search
{
    <DSL>
}
```

!!! example

    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "match_all": {}
            }
        }
        ```

    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 313

        {
            "took": 1,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 3,
                    "relation": "eq"
                },
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "3",
                        "_score": 1.0,
                        "_source": {
                            "title": "IPhone 11",
                            "price": 7999.99,
                            "description": "balabala11",
                            "create_at": "2022-07-19"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "1",
                        "_score": 1.0,
                        "_source": {
                            "title": "Andriod1",
                            "price": 1999.00,
                            "description": "Androd yyds",
                            "create_at": "2022-06-06"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.0,
                        "_source": {
                            "title": "Iphone1",
                            "price": 4999.00,
                            "description": "Iphone yyds",
                            "create_at": "2022-06-06"
                        }
                    }
                ]
            }
        }
        ```

## 常见检索
### 基本查询
#### 查询所有 match_all
!!! note
    返回索引中全部文档

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "match_all:" {}
    }
}
```

#### 关键词查询 term

!!! note
    使用关键词查询

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "term:" {
            "<FIELD>": {
                "value": <VALUE>
            }
        }
    }
}
```

keyword、integer、double、date 这些都是不分词的。

text 类型默认使用 ES 标准分词器。标准分词器对中文是单字分词，对英文是单词分词。

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "term": {
                    "price": {
                        "value": 4999.00
                    }
                }
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 240

        {
            "took": 2,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                "value": 1,
                "relation": "eq"
                },
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.0,
                        "_source": {
                            "title": "Iphone1",
                            "price": 4999.00,
                            "description": "Iphone yyds",
                            "create_at": "2022-06-06"
                        }
                    }
                ]
            }
        }
        ```

#### 范围 range

!!! note
    用于指定查询范围内的文档

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "range:" {
            "<FIELD>": {
                "<OPERATOR>": <VALUE>
                "relation": "<INTERSECTS | CONTAINS | WITHIN>"
                "boost": 1.0
            }
        }
    }
}
```

Operator 操作符包括：

- gt: 大于
- gte: 大于等于
- lt: 小于
- lte: 小于等于
- format: VALUE为字符串，匹配 data 用
- time_zone: 指定时区的偏移量

relation 用于指定查询策略：

- INTERSECTS(默认)：将文档与查询的范围做交集运算然后匹配
- CONTAINS：完全包含查询范围才算命中
- WITHIN：在查询范围内才算命中

boost 权重：用于提升在多范围查询时，当前范围的权重，默认1.0

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "range": {
                    "price": {
                        "gt": 1000,
                        "lt": 5000,
                    }
                }
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 266

        {
            "took": 11,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 2,
                    "relation": "eq"
                },
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "1",
                        "_score": 1.0,
                        "_source": {
                            "title": "Andriod1",
                            "price": 1999.00,
                            "description": "Androd yyds",
                            "create_at": "2022-06-06"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.0,
                        "_source": {
                            "title": "Iphone1",
                            "price": 4999.00,
                            "description": "Iphone yyds",
                            "create_at": "2022-06-06"
                        }
                    }
                ]
            }
        }
        ```

#### 前缀 prefix

!!! note
    使用前缀进行匹配查询

```http
GET /<INDEX_NAME>/_search

{
    "query": {
        "prefix": {
            "<FIELD>: {
                "value": <VALUE>
            }
        }
    }
}
```

```http
GET {{host}}/product/_search HTTP/1.1
Content-Type: application/json

{
    "query": {
        "prefix": {
            "title": {
                "value": "Ip"
            }
        }
    }
}
```


#### 通配符 wildcard

!!! note
    通配符查询，`？` 匹配一个任意字符，`*` 匹配多个任意字符

```http
GET /<INDEX_NAME>/_search

{
    "query": {
        "wildcard": {
            "<FIELD>: {
                "value": <VALUE>
            }
        }
    }
}
```

```http
GET {{host}}/product/_search HTTP/1.1
Content-Type: application/json

{
    "query": {
        "wildcard": {
            "title": {
                "value": "Ip*"
            }
        }
    }
}
```

#### 多ID查询 ids

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "ids": {
            "values": ["<ID>", <ID>, ...]
        }
    }
}
```


??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "ids": {
                    "values": ["1", "2", "3"]
                }
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 290

        {
            "took": 27,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 3,
                    "relation": "eq"
                },
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "1",
                        "_score": 1.0,
                        "_source": {
                            "title": "Andriod1",
                            "price": 1999.00,
                            "description": "Andriod1 yyds",
                            "create_at": "2022-06-06"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.0,
                        "_source": {
                            "title": "Iphone1",
                            "price": 4999.00,
                            "description": "Iphone1 yyds",
                            "create_at": "2022-06-07"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "3",
                        "_score": 1.0,
                        "_source": {
                            "title": "Andriod2",
                            "price": 2999.00,
                            "description": "Andriod2 yyds",
                            "create_at": "2022-06-08"
                        }
                    }
                ]
            }
        }
        ```

#### 模糊查询 fuzzy

!!! note
    用来模糊查询包含有关键词的文档

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "fuzzy": {
            "<FIELD>": "<VALUE>"
        }
    }
}
```

!!! warning
    fuzzy 模糊查询，最大模糊错误必须在 0-2 之间

    - 搜索关键词长度为 2 不允许存在模糊错误
    - 搜索关键词长度为 3-5 允许一次模糊错误
    - 搜索关键词长度大于 5 允许最大2次模糊错误

??? example "关键词长度为6(> 5)允许最大2次模糊错误"
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "fuzzy": {
                    "description": "Iphonew"    // 错误了一个字符 w
                }
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 302

        {
            "took": 14,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 3,
                    "relation": "eq"
                },
                "max_score": 0.9241961,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 0.9241961,
                        "_source": {
                            "id": 2,
                            "title": "Iphone1",
                            "price": 4999.00,
                            "description": "Iphone1 yyds",
                            "create_at": "2022-06-07"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "4",
                        "_score": 0.9241961,
                        "_source": {
                            "id": 4,
                            "title": "Iphone2",
                            "price": 3999.00,
                            "description": "Iphone2 yyds",
                            "create_at": "2022-06-09"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "5",
                        "_score": 0.9241961,
                        "_source": {
                            "id": 5,
                            "title": "Iphone3",
                            "price": 6978.00,
                            "description": "Iphone3 yyds",
                            "create_at": "2022-06-10"
                        }
                    }
                ]
            }
        }
        ```

### 复杂查询

#### 布尔查询 bool

!!! note
    `bool` 关键字用来组合多个条件实现复杂查询

    - `must`: 相当于 `&&`
    - `should`: 相当于 `||`
    - `must_not`: 相当于 `!`

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "bool": {
            "< must | should | must_not >": [
                { 
                    "<term | range | fuzzy | ...>": {
                        "<FIELD>": <VALUE>
                    }
                },
                ...other condition
            ]
        }
    }
}
```

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "bool": {
                    "must": [
                        {
                            "range": {
                                "price": {
                                    "gt": 4999,
                                    "lt": 10000
                                }
                            }
                        }, {
                            "prefix": {
                                "title": {
                                    "value": "Iph"
                                }
                            }
                        }
                    ]
                }
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 246

        {
            "took": 15,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 1,
                    "relation": "eq"
                },
                "max_score": 2.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "5",
                        "_score": 2.0,
                        "_source": {
                            "id": 5,
                            "title": "Iphone3",
                            "price": 6978.00,
                            "description": "Iphone3 yyds",
                            "create_at": "2022-06-10"
                        }
                    }
                ]
            }
        }
        ```

#### 多字段查询 multi_match

!!! note
    multi_match 在多个字段中检索能匹配查询关键词的文档

```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "multi_match": {
            "query": "<QUERY_STRING>"
            "fields": ["<FIELD>", "<FIELD>", ...]
        }
    }
}
```
!!! warning
    字段类型如果分词，则将查询关键词分词之后去指定的那些字段检索

    字段类型如果不分词，则将查询关键词分词作为整体去指定的那些字段检索

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "multi_match": {
                    "query": "五分钟",
                    "fields": [ "title", "description" ]
                }
            }
        }
        ```
        虽然 title 是 keyword 类型，不分词，但是 description 是 text 类型，分词的。

        ES 会将 “五分钟” 作为整体去 title 中检索，检索不到会分词成为 “五”、“分”、“钟”，然后去 description 中检索。最后检索到 id 为 4 的文档。

    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 296

        {
            "took": 2,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 1,
                    "relation": "eq"
                },
                "max_score": 3.0841155,
                "hit
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "4",
                        "_score": 3.0841155,
                        "_source": {
                            "id": 4,
                            "title": "Oppo",
                            "price": 3999.00,
                            "description": "充电两小时, 通话五分钟",
                            "create_at": "2022-06-09"
                        }
                    }
                ]
            }
        }
        ```

#### 默认字段分词查询 query_string


```http
GET /<INDEX_NAME>/_search
{
    "query": {
        "query_string": {
            "default_field": "<FIELD>"
            "query": ""
        }
    }
}
```

### 其他特性

#### 高亮查询 highlight

!!! warning
    ES7 中只有能分词的才能高亮

```http
GET /<INDEX_NAME>/_search
{
    "query:" { ... },
    "highlight": {
        "fields": {
            "<FIELD | *>": {},
            "pre_tags": ["<PRE_TAGS>"],
            "post_tags": ["<POST_TAGS>"],
            "requiree_field_match": "<true | false>"    // 其他字段也一起高亮，false开启，true 只高亮对应字段
        }
    }
}
```

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "fuzzy": {
                    "description": "Iphonew"
                }
            },
             "highlight": {
                "fields": {
                    "*": {}
                }
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 278

        {
            "took": 18,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 1,
                    "relation": "eq"
                },
                "max_score": 1.129573,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.129573,
                        "_source": {
                            "id": 2,
                            "title": "Iphone",
                            "price": 4999.00,
                            "description": "Iphone just yyds",
                            "create_at": "2022-06-07"
                        },
                        "highlight": {  // 多出这一段即为高亮检索词的结果
                            "description": [
                                "<em>Iphone</em> just yyds"
                            ]
                        }
                    }
                ]
            }
        }
        ```

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "fuzzy": {
                    "description": "Iphone"
                }
            },
            "highlight": {
                "fields": { "*": {} },
                "pre_tags": ["<span style='color:red;'>"],
                "post_tags": ["</span>"]
            }
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 278

        {
            "took": 18,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 1,
                    "relation": "eq"
                },
                "max_score": 1.129573,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.129573,
                        "_source": {
                            "id": 2,
                            "title": "Iphone",
                            "price": 4999.00,
                            "description": "Iphone just yyds",
                            "create_at": "2022-06-07"
                        },
                        "highlight": {  // 多出这一段即为高亮检索词的结果
                            "description": [
                                "<span style='color:red;'>Iphone</span> just yyds"
                            ]
                        }
                    }
                ]
            }
        }
        ```

#### 分页 from, size

!!! note
    `from` 用来指定起始返回位置，默认0；`size` 用来指定返回条数，默认10。

    `size` 可以单独使用，两者结合可以实现分页效果。

```http
GET /<INDEX_NAME>/_search
{
    "query": { ... },
    "size": <SIZE>,
    "from": <FROM>
}
```

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "match_all": {}
            },
            "from": 2,
            "size": 10
        }
        ```

    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 381

        {
            "took": 1,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 5,
                    "relation": "eq"
                },
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "3",
                        "_score": 1.0,
                        "_source": {
                            "id": 3,
                            "title": "Xiaomi",
                            "price": 2999.00,
                            "description": "Xiaomi 为发烧而生",
                            "create_at": "2022-06-08"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "4",
                        "_score": 1.0,
                        "_source": {
                            "id": 4,
                            "title": "Oppo",
                            "price": 3999.00,
                            "description": "充电两小时, 通话五分钟",
                            "create_at": "2022-06-09"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "5",
                        "_score": 1.0,
                        "_source": {
                            "id": 5,
                            "title": "Vivo",
                            "price": 6978.00,
                            "description": "拍出你的美",
                            "create_at": "2022-06-10"
                        }
                    }
                ]
            }
        }
        ```

#### 排序 sort

默认降序 desc

```http
GET /<INDEX_NAME>/_search
{
    "query": { ... },
    "sort": [
        { "<FIELD>": "<desc | asc>" },
        { 
            "<FIELD>": {
                "order": "<desc | asc>",
                "format": "strict_date_optional_time_nanos"
            }
        },
        { 
            "<FIELD>": {
                "order": "<desc | asc>",
                "mode": "<min | max | sum | avg | median>"
            }
        }
    ]
}
```

#### 返回指定字段 _source
```http
GET /<INDEX_NAME>/_search
{
    "query": { ... },
    "_source": ["<FIELD>", "<FIELD>", ...]
}
```

??? example
    === "Req"
        ```http
        GET {{host}}/product/_search HTTP/1.1
        Content-Type: application/json

        {
            "query": {
                "match_all": {}
            },
            "size": 2,
            "_source": ["id", "title"]
        }
        ```
    === "Resp"
        ```http
        HTTP/1.1 200 OK
        content-type: application/json; charset=UTF-8
        content-encoding: gzip
        content-length: 213

        {
            "took": 4,
            "timed_out": false,
            "_shards": {
                "total": 1,
                "successful": 1,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": {
                    "value": 5,
                    "relation": "eq"
                },
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "1",
                        "_score": 1.0,
                        "_source": {    // 只返回指定字段
                            "id": 1,
                            "title": "Andriod"
                        }
                    },
                    {
                        "_index": "product",
                        "_type": "_doc",
                        "_id": "2",
                        "_score": 1.0,
                        "_source": {    // 只返回指定字段
                            "id": 2,
                            "title": "Iphone"
                        }
                    }
                ]
            }
        }
        ```