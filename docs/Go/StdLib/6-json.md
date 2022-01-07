**序列化（struct -> JSON）**：结构体对象 转成 JSON格式字符串

**反序列化（JSON -> struct）**：JSON格式字符串 转成 结构体对象


使用 `encoding/json` 完成。

```go
import "encoding/json"
```

```go
// 序列化函数，struct -> JSON
func Marshal(v interface{}) ([]byte, error) {}
// 反序列化函数，JSON -> struct
func Unmarshal(data []byte, v interface{}) error {}
```

!!!example

    ```go
    package main

    import (
        "encoding/json"
        "fmt"
    )

    // 1. 序列化 encode：把结构体变量 -> json格式的字符串
    // 2. 反序列化 decode：json格式的字符串 -> 结构体变量

    type person struct {
        Name string `json:"name" db:"name" ini:"name"`
        Age  int    `json:"age"`
        Sex  string
        Addr string `json:"-"`    // value 为 "-" 时不序列化
    }

    func main() {
        p1 := person{"Boii", 18, "male"}

        /* 序列化 */
        b, err := json.Marshal(p1)
        if err != nil {
            fmt.Println(err)
        }
        fmt.Println(string(b)) // {"name":"Boii","age":18,"Sex":"male"}
        fmt.Printf("%T \n", b) // []uint8

        /* 反序列化 */
        str := `{"name":"Eva","age":18}`
        var p2 person
        json.Unmarshal([]byte(str), &p2)
        fmt.Printf("%#v \n", p2) // main.person{Name:"Eva", Age:18, Sex:""}
        fmt.Printf("%T \n", p2)  // main.person
    }
    ```

序列化时，key 会优先选择字段的 Tag 中指定的 key

!!! warning

    **结构体中非 public 的字段不会被序列化**