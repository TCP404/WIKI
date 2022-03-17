# Go Module

## 启用 GO111MODULE
**版本要求：go 1.11+，1.13起默认开启**

启用 go module 支持首先要设置环境变量，有三个可选值：

- auto
- on
- off

默认是 auto

1. `GO111MODULE=off` 禁用模块支持，编译时会从 `GOPATH` 和 `vendor` 文件夹中查找包。
2. `GO111MODULE=on` 开启模块支持，编译时会忽略 `GOPATH` 和 `vendor` 文件夹，只根据 `go.mod` 下载依赖。
3. `GO111MODULE=auto`，当项目在 `$GOPATH/src` 外且项目根目录有 `go.mod` 文件时，开启模块支持。

设置 `GO111MODULE=on` 之后就可以使用 `go module` 了，以后就没必要在 `GOPATH` 中创建项目了。

使用 go module 管理依赖后会在项目根目录下生成两个文件 `go.mod` 和 `go.sum`。

开启方式：

=== "Windows"
    在环境变量中添加一条值

    ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210217115631352_20287.png)

=== "Linux"
    直接执行命令到 `/etc/profile` 或 `$HOME/.profile`

    ```shell
    $ echo "export GO111MODULE=auto" >> /etc/profile
    ```
    或者
    ```go
    $ go env -w GO111MODULE=auto
    ```



## go mod 命令

常用 `go mod` 命令如下：

```
go mod init     初始化当前文件夹，创建 go.mod 文件
go mod download 下载依赖的 module 到本地cache（默认为 $GOPATH/pkg/mod，使用命令 go env GOMODCACHE 可查看）
go mod edit     编辑 go.mod 文件
go mod graph    打印模块依赖图
go mod tidy     增加缺少的 module，删除无用的 module
go mod vendor   将依赖复制到 vendor 下，vendor 目录不存在时会自动创建
go mod verify   检验依赖
go mod why      解释为什么要依赖
```

## go mod init 初始化

初始化当前文件夹，创建 go.mod 文件。

第一步，切换到项目目录中
第二步，执行命令 `go mod init module名`

eg：

![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210217123923793_585.png)


## go mod tidy 整理依赖
我们在代码中删除依赖代码后，相关的依赖库并不会在 `go.mod` 文件中自动移除。这种情况下我们可以使用 `go mod tidy` 命令更新 `go.mod` 中的依赖关系。

```shell
$ go mod tidy
```


## go mod edit 编辑mod文件
### 格式化
因为我们可以手动修改go.mod文件，所以有些时候需要格式化该文件。Go提供了一下命令：

```shell
$ go mod edit -fmt
```

### 添加依赖项

```shell
$ go mod edit -require=要添加的模块
```

eg：
```shell
$ go mod edit -require=rsc.io/quote
```

### 移除依赖项
```shell
$ go mod edit -droprequire=要移除的模块
```

eg：
```shell
$ go mod edit -require=github.com/sirupsen/logrus@v1.7.1
```

## go get

在项目中执行 `go get` 命令可以下载依赖包，并且还可以指定下载的版本。

1. 运行 `go get -u` 将会升级到最新的次要版本或者修订版本(x.y.z, z是修订版本号， y是次要版本号)
2. 运行 `go get -u=patch` 将会升级到最新的修订版本
3. 运行 `go get package@version` 将会升级到指定的版本号 version

```shell
$ go get 
```

如果下载所有依赖可以使用 `go mod download` 命令。


## go.mod 文件

**`go.mod`** 有 5 种指令：`module`、`go`、`require`、`exclude`、`replace`

- **module**：定义模块路径
- **go**：设置预期的语言版本
- **require**：依赖包列表及版本
- **exclude**：禁止依赖包列表（仅在当前模块为主模块时生效）
- **replace**：替换依赖包列表（仅在当前模块为主模块时生效）


```js
module my/thing

go 1.16

require other/thing v1.0.2 // 这是注释
require new/thing/v2 v2.3.4 // indirect
require（
  new/thing v2.3.4
  old/thing v0.0.0-20190603091049-60506f45cf65
）

exclude old/thing v1.2.3

replace bad/thing v1.4.5 => good/thing v1.4.5
```

## 语义化版本
语义化版本号格式为：`X.Y.Z`（主版本号.次版本号.修订号），使用方法如下：

- 进行不向下兼容的修改时，递增主版本号。
- API 保持向下兼容的新增及修改时，递增次版本号。
- 修复问题但不影响 API 时，递增修订号。

例如：
有一个语义化版本号为：`v0.1.2`，则其主版本号为 0，次版本为 1，修订号为 2。

而前面的 v 是 version（版本）的首字母，是 Go 语言惯例使用的，标准的语义化版本没有这个约定。


所以在使用 Go 命令行工具或 go.mod 文件时，就可以使用语义化版本号来进行模块查询，具体规则如下：

- 默认值（`@latest`）：将匹配**最新可用**的标签版本或源码库的最新未标签版本。
- 指定某个 commit（`@c854792`）：将匹配该 **commit** 时的版本。
- 指定某个分支（`@master`）：将匹配该**分支**版本。
- 完全指定版本（`@v1.2.3`）：将匹配该**指定**版本。
- 版本前缀（`@v1 或 @v1.2`）：将匹配具有该**前缀**的**最新可用**标签版本。
- 版本比较（`@<v1.2.3 或 @>=v1.5.6`）：将匹配**最接近比较目标**的可用标签版本。< 则为小于该版本的最新版本，> 则为大于版本的最旧版本。当使用类 Unix 系统时，需用引号将字符串包裹起来以防止大于小于号被解释为重定向。
    如：`go get 'github.com/gin-gonic/gin@<v1.2.3'`。


![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210217145451209_10624.png)

如上图所示，为了能让 Go Modules 的使用者能够从旧版本更方便地升级至新版本，Go 语言官方提出了两个重要的规则：

1. 导入兼容性规则（import compatibility rule）：如果旧包和新包具有相同的导入路径，则新包必须向后兼容旧包。
2. 语义化导入版本规则（semantic import versioning rule）：每个不同主版本（即不兼容的包 v1 或 v2）使用不同的导入路径，以主版本结尾，且每个主版本中最多一个。如：一个 rsc.io/quote、一个 rsc.io/quote/v2、一个 rsc.io/quote/v3。

而与 Git 分支的集成如下：

![Go Modules 分支](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210217145532595_1214.png)

!!! note ""
    参考 [Go Modules 详解](https://juejin.cn/post/6844903872054427656)


## 在项目中使用 go module



=== "既有项目"
    如果需要对一个已经存在的项目启用 `go module`，可以按照以下步骤操作：

    1. 在项目目录下执行 `go mod init`，生成一个go.mod文件。
    2. 执行 `go get`，查找并记录当前项目的依赖，同时生成一个go.sum记录每个依赖库的版本和哈希值。

=== "新项目"
    对于一个新创建的项目，我们可以在项目文件夹下按照以下步骤操作：

    1. 执行 `go mod init 项目名` 命令，在当前项目文件夹下创建一个 `go.mod` 文件。
    2. 手动编辑 `go.mod` 中的 require 依赖项或执行 `go get` 自动发现、维护依赖。
    3. 如果没反应，可以试试 `go mod tidy`


## VSCode-go 的坑

=== "开启 GOMODULE 就报错"

    !!! note ""
        报错信息：`found module "golang.org/x/tools" twice in the workspace`

    一开始的时候我是在 VSC 里建了一个工作区，把 GOPATH 所在目录添加在工作区里，然后开启 GO111MODULE=auto，结果就一直报错。

    后来 **新建一个工作区**，把自己放项目的文件夹添加到工作区，就没事了。

    我的 GOPATH：`GOPATH=E:\---CODE\GO\root`

    GOPATH 放进去的工作区：
    ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210216232600109_13119.png)


    没有 GOPATH 的工作区

    ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210216232836076_13857.png)

    相安无事


=== "go tools 下载失败"

    解决办法：把 GOPROXY 改到 [https://goproxy.cn](https://goproxy.cn)。阿里云那个有时候不行，官网被墙。
    我试了只有这个地址一次过。

    - 第一步：修改环境变量 GOPROXY
        ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210216233219520_22889.png)

    - 第二步：确保 GOPATH 下有`bin`、`pkg`、`src` 三个目录

    - 第三步：在 VSC 中按下 `Ctrl+Shift+P` 唤醒快捷命令行，输入：`Go:install/Update Tools`，回车。
        ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210216232958058_14544.png)


    - 第四步：全选，然后点击 OK 开始下载
        ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210216233135419_18930.png)

    - 第四步：丝滑顺畅一次过
        ![](https://xcdn.loli.top/gh/TCP404/Picgo/blog/illustration-pic/Go/IMG/20210216233350018_30494.png)
