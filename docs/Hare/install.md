
# 下载
先安装 qbe
```shell
sudo pacman -Sy qbe scdoc
```

下载后端编译器
```shell
git clone https://git.sr.ht/\~sircmpwn/harec
```

下载构建器
```shell
git clone https://git.sr.ht/\~sircmpwn/hare
```

先进入编译器目录
```bash
cd harec
./configure       # 生成配置文件
make check        # 检查一遍
sudo make install # 用超管执行安装
```

然后进入构建器目录
```bash
cd ../hare
cp config.example.mk config.mk # 复制编译配置文件
make check                     # 检查一遍
sudo make install              # 用超管执行安装标准库
make                           # 执行构建器安装
```

## Hello World
```hare
// main.ha

use fmt;

export fn main() void = {
    fmt::println("Hello World")!;
};
```

执行命令 `hare run main.ha` 将会编译一个临时的可执行文件并执行。
```shell
$ hare run main.ha
Hello World
```
