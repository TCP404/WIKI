# Go Work

**版本要求：Go 1.18+**

## 起因（适用场景）

### 前置

项目（模块） -> 包（package）

-   项目（模块）指目录下有 go.mod 文件的那个目录，以 go.mod 文件所在的目录为项目根目录
-   包（package）指项目中更小一级的管理单位

## 另一个项目未发布

遇到多项目开发时，比如 A1 和 A2 是两个项目，也就是两个模块，彼此进行开发，彼此都未发布，而在 A1 中引用了 A2。因为 A2 没发布，所以会引用失败，以前的做法是在 A1 的 go.mod 中使用 replace 指令：`replace A2 => ../A2`，到时候等 A2 发布后，准备发布 A1 时把这句去掉。要是忘了就麻烦了。



## 使用方法

创建一个工作区目录 work_dir，在此目录下创建项目，例如 hello/、util，要求每个项目下都有 go.mod 文件。

在 work_dir 目录下执行：`go work init`，会生成一个 `work_dir/go.work` 文件。其语法与 go.mod 相同，多了一个 use 指令。其他的如 replace 指令，其优先级比 go.mod 中的 replace 高。

接下来在项目中正常引用就行了。

```shell
[work_dir]
 ├── go.work
 │   +──────────+ 
 │   │ go 1.18  │  
 │   │          │
 │   │ use (    │
 │   │	./hello │
 │   │	./util  │
 │   │ )        │
 │   +──────────+
 │ 
 ├── [hello]
 │    ├── go.mod
 │    │   +──────────────────────────────────+
 │    │   │ module abc.com/hello_demo        │
 │    │   │                                  │
 │    │   │ go 1.18                          │
 │    │   │                                  │
 │    │   │ require abc.com/util_demo v0.0.0 │
 │    │   +──────────────────────────────────+
 │    │  
 │    └── main.go
 │        +─────────────────────────────────────────+
 │        │ package main                            │
 │        │                                         │
 │        │ import (                                │
 │        │ 	"abc.com/util_demo"                 │
 │        │ )                                       │
 │        │                                         │
 │        │ func main() {                           │
 │        │     util.Assert(1==1, "panic message.") │
 │        │ }                                       │
 │        +─────────────────────────────────────────+
 │
 └── [util]
      ├─── go.mod 
      │   +──────────────────────────+
      │   │ module abc.com/util_demo │
	  │	  │                          │
	  │   │ go 1.18                  │
      │   +──────────────────────────+
      │
      └── main.go
          +──────────────────────────────────────+
      	  │ package util                         │       
      	  │                                      │
      	  │ func Assert(cond bool, msg string) { │
      	  │     if cond {                        │
      	  │         return                       │
      	  │     }                                │
      	  │     panic(msg)                       │
      	  │ }                                    │
		  +──────────────────────────────────────+
```

 

