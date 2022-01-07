

## 模板语法

`{{}}` 为分界符

### 渲染对象

`{{.}}` 所有语法都包含在双大括号中，点 `.` 表示当前对象。

要使用对象中的字段，可以用 `{{.fieldName}}`

eg1:

```go
tmpl, _ := template.New("demo").Parse(`
hello {{.name}},
obj is {{.}}
`)

data := map[string]interface{}{
    "name": "Boii",
    "age":  18,
}

tmpl.Execute(os.Stdout, data)
```

输出：

```

hello Boii,
obj is map[name: Boii age:18]
```

eg2:

```go
tmpl, _ := template.New("demo").Parse(`
Hello {{.Name}},
obj is {{.}}
`)

data := struct {
	Name string
	Age  int
}{Name: "Boii", Age: 18}

tmpl.Execute(os.Stdout, data)
```

输出：

```

Hello Boii,
obj is {Boii 18}
```



### 注释

`{{/* 注释，可以多行，不能嵌套，注释符号贴紧分界符*/}}`

### 空格

`{{- xxx -}}`，移除空格，`-` 要贴紧分界符，与模板值 xxx 用空格隔开

-   `{{- xxx}}`：去掉左边所有空格
-   `{{xxx -}}`：去掉右边所有空格
-   `{{- xxx -}}`：去掉两边所有空格

### 使用变量

`{{$var := .}}` 使用美元符号加变量名可以定义一个模板中的变量，定义后可以在模板内其他任意地方使用。

```go
tmpl, _ := template.New("demo").Parse(`
{{$a := "Boii"}}
hello {{$a}}

{{$b := .}}
hello {{$b.Name}}
`)

data := struct{
    Name string
    Age  int
}{Name: "Eva", Age: 18}

tmpl.Execute(os.Stdout, data)
```

输出：

```

hello Boii

hello Eva
```





条件

```tpl
{{if cond}}
	...
{{end}}


{{if cond}}
	...
{{else}}
	...
{{end}}


{{if cond}}
	...
{{else if cond}}
	...
{{end}}
```

循环

Range 用于遍历数组，和 go 的 range 一样，可以直接得到每个变量，或者得到 index 和value。

```html
cond 的值必须为数组、切片、字典或通道

{{range cond}}
	...
{{end}}
如果 cond 长度为0，则不会有任何输出


{{range cond}}
	T1
{{else}}
	T2
{{end}}
如果 cond 长度为0，则输出 T2
```


```go
tmpl, _ := template.New("demo").Parse(`
{{range .val}}
	{{- . -}}
{{end}}

{{range $i, $v := .val}}
	id: {{$i}}, value: {{$v}}
{{end}}
`)

data := map[string]interface{}{
    "val": []string{"B", "o", "i", "i"}
}

tmpl.Execute(os.Stdout, data)
```


```
Boii

        id: 0, value: B

        id: 1, value: o

        id: 2, value: i

        id: 3, value: i
```

```html

{{with cond}}
	T1
{{end}}
如果 cond 不为 empty，将 dot 设为 cond 的值并执行 T1，不修改外面的 dot
如果 cond 为 empty 则不产生输出，


{{with cond}}
	T1
{{else}}
	T2
{{end}}
如果 cond 不为 empty，则 dot 设为 cond 的值并执行 T1
如果 cond 为 empty，则不改变 dot 并执行 T2
```



```html
{{block "name" pipeline}}
	T1
{{end}}
```

















