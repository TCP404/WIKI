# Hello Rust

## 特点

- 安全
- 无需 GC
- 易于维护、调试，代码安全高效

## 擅长领域
- 高性能 Web Service
- WebAssembly
- 命令行工具
- 网络编程
- 嵌入式编程
- 系统编程

## Hello World
程序文件后缀名: .rs

文件命名规范: 下划线 hello_world.rs
```Rust
fn main() {
    println!("Hello World");
}
```

- 定义函数: `fn main() {}`, 无参数，无返回值。
- main 函数是rust程序最先运行的代码
- 打印文本：`println!("Hello World");`
    - Rust 的缩进是 4 个空格而不是 tab
    - println! 是一个 Rust macro (宏)，如果是函数的话就没有 `!`

编译:
```bash
$ rustc main.rs
```

- Rust 程序必须先编后再执行。编译命令为 `rustc 文件名`
- 编译成功后会生成一个二进制文件。如果在 Windows 上还会生成一个 `.pdb` 文件，里面包含调试信息
- rustc 只适合简单的 Rust 程序，大型程序得用 cargo.

## Cargo

- 查看版本: `cargo --version`
- 新建项目: `cargo new 项目名称`
- 构建项目: `cargo build`
    - 创建可执行文件: `target/debug/main` 或 `target\debug\main.exe`
    - 第一次运行 cargo build 会在顶层目录生成 `cargo.lock` 文件，负责追踪项目依赖的精确版本，类似 js 的 `package.lock` 或 go 的 `go.sum`
- 运行项目: `cargo run`
    - 创建可执行文件: `target/debug/main` 或 `target\debug\main.exe`
    - 运行可执行文件: `target/debug/main` 或 `target\debug\main.exe`
- 检查项目: `cargo check`
    - 检查代码，确保能通过编译，不会产生任何可执行文件
    - 比 cargo build 快
- 构建发布项目: `cargo build --release`
    - 编译时会进行优化，代码会运行的更快，但编译时间更长
    - 会在 `target/release` 生成可执行文件

Rust 默认会把 prelude(预导入) 模块导入到每个程序的作用域中，如果使用了不再 prelude 的东西就得使用 `use` 关键字显式导入。
