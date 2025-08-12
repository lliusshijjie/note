# Rust

## 一、Rust简介

Rust 是由 Mozilla 主导开发，并于 2015 年首次稳定发布（1.0 版本）的开源语言。它的设计目标是解决系统编程领域长期存在的痛点（尤其是内存安全和并发安全问题），同时保持与 C/C++ 相媲美的高性能和底层控制能力。

### 1. 核心目标：安全、并发、性能

#### 1.1 **内存安全 (Memory Safety):** 

这是 Rust 最著名的特性。它通过一套**独特的编译时所有权系统**，在**不依赖垃圾回收 (GC)** 的情况下，**几乎完全消除了悬垂指针、缓冲区溢出、数据竞争、空指针解引用等困扰 C/C++ 的经典内存错误**。开发者无需手动管理内存（像 C/C++ 那样易出错），也无需忍受 GC 带来的停顿（像 Java, Go, Python 那样）。

#### 1.2 **并发安全 (Concurrency Safety):** 

Rust 的所有权系统和类型系统天然地帮助开发者编写安全的并发代码。编译器能够在编译期就阻止**数据竞争**的发生。这使得编写高效、可靠的多线程程序变得相对容易和自信。

#### **1.3 高性能 (Performance):**

 Rust 编译成本地机器码，没有运行时开销（无 GC，极小的运行时），可以精细控制内存布局，优化程度高。其性能通常与 C/C++ 处于同一水平，非常适合对性能要求苛刻的场景（操作系统、游戏引擎、浏览器组件、数据库、实时系统等）。

### 2. 核心机制：所有权、借用、生命周期

#### 2.1 **所有权 (Ownership):** 

这是 Rust 的基石规则。

*   每个值在 Rust 中都有一个被称为其 **所有者** 的变量。
*   值在任一时刻有且只有一个所有者。
*   当所有者离开作用域时，这个值将被自动 **丢弃** (`drop`)，其占用的内存被释放。

#### 2.2 借用 (Borrowing):

 为了避免频繁的所有权转移（移动），Rust 允许你“借用”值。

*   **引用 (`&T`)**: 允许你读取数据但不修改它（不可变引用）。一个作用域内可以有多个不可变引用。
*   **可变引用 (`&mut T`)**: 允许你读取并修改数据（可变引用）。在特定作用域内，对同一块数据，**要么只能有一个可变引用，要么只能有多个不可变引用，二者不能同时存在**。这从根本上防止了数据竞争。

*   **生命周期 (Lifetimes):** 是 Rust 用来确保引用始终有效的机制。编译器会分析引用的作用域，确保它们不会比它们所引用的数据存活得更久（防止悬垂引用）。开发者通常不需要显式标注，编译器可以自动推断（生命周期省略），但在复杂场景下需要手动标注生命周期参数（如 `'a`）来帮助编译器理解。

这套机制在**编译时**强制执行，确保了内存安全和线程安全，将很多运行时错误提前到编译期发现，极大地提高了程序的健壮性。

### 3. 零成本抽象 (Zero-Cost Abstractions)

Rust 提供了高级的语言特性（如泛型、trait、闭包、迭代器、模式匹配等），但这些特性在设计上力求“零成本”。这意味着：

*   **使用高级抽象不会引入运行时开销。** 生成的机器码理论上应该和你手写底层代码一样高效。
*   你只需要为你实际使用的功能付出代价。未使用的抽象不会影响性能。

这使得 Rust 既能让你用优雅、表达力强的方式编写代码，又能获得底层语言般的性能。

### 4. 强大的类型系统和模式匹配

*   **强类型 & 静态类型:** 类型在编译期确定，有助于捕获大量错误。
*   **类型推断:** 编译器能根据上下文推断出很多类型，减少冗余的类型注解。
*   **代数数据类型 (ADT):** 通过 `enum` 和 `struct` 提供了强大的数据建模能力。`enum` 可以携带不同类型和数量的数据（比传统枚举强大得多）。
*   **模式匹配 (`match`):** 一种极其强大、表达力强的控制流结构，可以优雅地解构 `enum`、`struct`、元组、引用等，并根据不同的模式执行不同的代码分支。它是处理复杂逻辑和状态的安全方式（强制覆盖所有可能情况）。
*   **Trait:** Rust 实现多态和代码复用的核心机制，类似于其他语言中的接口 (`interface`) 或类型类 (`typeclass`)。它定义了类型必须实现的一组方法。Trait 可以用于：
    *   **泛型约束:** 限制泛型类型参数必须实现某些 Trait。
    *   **Trait 对象 (`dyn Trait`):** 实现动态分发（运行时多态）。
    *   **运算符重载。**
    *   **定义默认方法实现。**
    *   **扩展已有类型的功能 (通过 `impl Trait for Type`)。**

### 5. 卓越的工具链 (`Cargo` & `rustup`)

#### 5.1 **Cargo:**

   Rust 的**构建系统和包管理器**，是开箱即用的体验核心。

*   轻松创建、构建、运行、测试项目 (`cargo new`, `cargo build`, `cargo run`, `cargo test`)。
*   强大的依赖管理：从 [crates.io](https://crates.io/) (Rust 的官方包仓库) 添加、更新和管理依赖项极其简单（在 `Cargo.toml` 中声明即可）。
*   统一的项目结构和构建流程。
*   丰富的插件支持 (`cargo clippy` - linting, `cargo fmt` - 代码格式化)。

#### 5.2 **rustup:** 

   Rust 的**工具链安装器和版本管理器**。

*   轻松安装、更新和切换不同版本的 Rust 编译器 (`rustc`) 和工具链（稳定版、测试版、夜间版）。
*   管理不同编译目标（交叉编译）。
*   安装常用组件。

### 6. 活跃友好的社区

*   Rust 拥有一个**规模庞大、极其活跃且以友好、包容著称的社区**。
*   官方文档 ([The Rust Programming Language](https://doc.rust-lang.org/book/) 俗称 "The Book") 是**公认的极其优秀的入门教程**，清晰、深入、循序渐进。
*   丰富的学习资源、第三方库 (`crates`)、论坛 ([Rust Users Forum](https://users.rust-lang.org/), [Reddit r/rust](https://www.reddit.com/r/rust/)) 和聊天平台 (Discord, Zulip)。
*   社区文化强调互助和学习，对新手非常友好。

### 7. 应用领域

得益于其安全、性能和并发特性，Rust 被广泛应用于：

*   **系统编程:** 操作系统 (如 Redox OS, Linux kernel modules)、文件系统、驱动程序、嵌入式系统 (no_std)、浏览器引擎 (Firefox 的 Servo, Chromium 的部分组件)、虚拟化。
*   **网络服务:** WebAssembly (WASM) 后端、高性能网络服务器/代理 (如 nginx 模块)、API 后端 (得益于框架如 Actix-web, Rocket, Axum)。
*   **命令行工具 (CLI):** 构建高效、易于分发的命令行应用 (如 ripgrep, fd, exa, bat)。
*   **区块链/加密货币:** 众多区块链项目（如 Solana, Polkadot, Near）的核心组件使用 Rust 编写。
*   **游戏开发:** 游戏引擎 (Bevy, Amethyst)、游戏逻辑、高性能后端服务。
*   **基础设施软件:** 数据库 (如 TiKV, SurrealDB, Materialize)、消息队列、搜索引擎组件。
*   **跨平台开发:** 编译成本地代码或 WASM，可在多种平台运行。
*   **需要高可靠性和安全性的关键系统。**

### 8. Rust 的优势

1.  **无与伦比的内存安全和线程安全保证 (编译时检查)。**
2.  **媲美 C/C++ 的极致性能。**
3.  **卓越的开发者体验：** 强大的工具链 (Cargo, rustup)，优秀的文档，清晰的错误信息。
4.  **现代化且表达力强的语法：** 模式匹配、代数数据类型、Traits、零成本抽象、闭包、迭代器等。
5.  **蓬勃发展的生态系统和友好活跃的社区。**
6.  **适用于从底层系统到高性能网络服务的广泛领域。**

### 9. 结论

Rust 是一门雄心勃勃的语言，它成功地将**系统级的性能和控制力**与**高级语言的安全性和开发效率**结合在了一起。它通过创新的机制在编译期解决了许多传统上在运行时才能发现或难以解决的顽疾（尤其是内存和并发错误）。虽然学习曲线存在，但其带来的安全性、性能和现代化的开发体验，使其成为系统编程、高性能服务和关键基础设施领域极具吸引力的选择。其活跃的社区和强大的工具链也极大地提升了开发者的生产力。

如果你想追求性能、安全性和现代开发体验，并且愿意投入时间去掌握其独特的概念，Rust 绝对是一门值得学习和使用的强大语言。



## 二、cargo

---

以下是 Rust 构建工具和包管理器 `cargo` 的**常用命令清单**，覆盖了开发全流程的核心操作：

### 1. 🏗 **项目创建与初始化**

| 命令                             | 说明                                               |
| -------------------------------- | -------------------------------------------------- |
| `cargo new <project_name>`       | 创建**二进制 (可执行)** 项目（默认生成 `main.rs`） |
| `cargo new --lib <library_name>` | 创建**库**项目（生成 `lib.rs`）                    |
| `cargo init`                     | 在当前目录初始化新项目（适合已有目录）             |
| `cargo init --lib`               | 在当前目录初始化库项目                             |

---

### 2. 🛠 **构建与编译**
| 命令                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `cargo build`           | 编译项目（生成 debug 版，输出到 `target/debug/`）            |
| `cargo build --release` | 编译**优化后的发布版**（输出到 `target/release/`，性能更高） |
| `cargo check`           | **快速检查代码**（确保能编译通过，不生成可执行文件，速度极快） |
| `cargo clean`           | **清理**构建产物（删除 `target` 目录）                       |

---

### 3. ▶ **运行与测试**
| 命令                      | 说明                                          |
| ------------------------- | --------------------------------------------- |
| `cargo run`               | 编译并**运行**项目（自动 `build` 未编译时）   |
| `cargo run --release`     | 编译发布版并运行                              |
| `cargo run -- <args>`     | 带参数运行程序（如 `cargo run -- arg1 arg2`） |
| `cargo test`              | **运行所有测试**（检测 `#[test]` 标记的函数） |
| `cargo test <test_name>`  | 运行**指定名称**的测试（支持模糊匹配）        |
| `cargo test -- --ignored` | 运行被标记为 `#[ignore]` 的测试               |
| `cargo bench`             | 运行基准测试（需要 `#[bench]`，Nightly Rust） |

---

### 4. 📦 **依赖管理**
| 命令                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| `cargo add <crate_name>`      | **添加依赖**到 `Cargo.toml`（需安装 `cargo-edit`）<br>例：`cargo add serde` |
| `cargo add <crate>@<version>` | 添加指定版本依赖<br>例：`cargo add serde@1.0`                |
| `cargo update`                | 更新依赖（根据 `Cargo.lock` 升级到兼容版本）                 |
| `cargo update -p <crate>`     | 更新**指定依赖**                                             |
| `cargo tree`                  | **可视化依赖树**（显示所有传递依赖）                         |
| `cargo tree -p <crate>`       | 查看某个依赖被谁引入                                         |

---

### 5. 📝 **代码质量与文档**
| 命令                                 | 说明                                                    |
| ------------------------------------ | ------------------------------------------------------- |
| `cargo fmt`                          | **格式化代码**（使用 rustfmt，保持代码风格统一）        |
| `cargo clippy`                       | **静态代码检查**（需安装 clippy，捕捉常见错误和优化点） |
| `cargo doc`                          | **生成项目文档**（输出到 `target/doc/`）                |
| `cargo doc --open`                   | 生成文档并在浏览器中打开                                |
| `cargo doc --document-private-items` | 生成包含私有项的文档                                    |

---

### 6. 📤 **发布与安装**
| 命令                           | 说明                                                   |
| ------------------------------ | ------------------------------------------------------ |
| `cargo publish`                | **发布 crate 到 crates.io**（需先登录 `cargo login`）  |
| `cargo install <crate_name>`   | **全局安装二进制 crate**（如 `cargo install ripgrep`） |
| `cargo install --path .`       | 将当前项目安装为全局命令                               |
| `cargo uninstall <crate_name>` | 卸载全局安装的 crate                                   |

---

### 7. 🔍 **高级调试与分析**
| 命令                     | 说明                                                        |
| ------------------------ | ----------------------------------------------------------- |
| `cargo build -vv`        | **超详细构建输出**（调试构建问题时使用，`-v` 是普通详细）   |
| `cargo rustc -- <flags>` | 向 rustc 传递额外参数（如 `cargo rustc -- -C opt-level=3`） |
| `cargo expand`           | 查看宏展开后的代码（需安装 `cargo-expand`）                 |

---

### 8. ⚙ **配置与信息**
| 命令                      | 说明                                       |
| ------------------------- | ------------------------------------------ |
| `cargo --version`         | 查看 cargo 版本                            |
| `cargo version --verbose` | 查看 cargo 和 rustc 的详细版本信息         |
| `cargo metadata`          | 以 JSON 格式输出项目元数据（IDE/工具常用） |
| `cargo search <keyword>`  | 在 crates.io 搜索 crate                    |

---

### 9. 💡 高效工作流建议：
1. **日常开发循环**：  
   ```bash
   cargo check  # 快速检查语法
   cargo clippy # 静态检查
   cargo test   # 跑测试
   cargo run    # 运行
   ```
2. **提交代码前**：  
   ```bash
   cargo fmt    # 统一格式
   cargo clippy # 确保代码质量
   cargo test   # 验证功能
   ```
3. **性能敏感场景**：  
   ```bash
   cargo build --release # 编译发布版
   cargo run --release   # 运行发布版
   ```

> 小贴士：安装常用插件提升体验：
> ```bash
> cargo install cargo-edit  # 支持 cargo add/rm/upgrade
> cargo install cargo-watch # 文件变化时自动执行命令（如 `cargo watch -x check`）
> ```

掌握这些命令，你就能高效驾驭 Rust 的开发全流程！🚀



# Rust 基础

## Rust：输入和输出

Rust 的输入输出（I/O）主要通过标准库 `std::io` 模块实现，设计上注重**安全性和明确性**。以下是核心概念和常用操作的清晰介绍：

---

### 1. 📥 **基础输出：`println!` 和 `print!` 宏**
这是最常用的输出方式，用于向控制台打印文本。

1. **`println!`**：打印内容并**换行**
   ```rust
   println!("Hello, world!"); // 输出：Hello, world!（带换行）
   ```

2. **`print!`**：打印内容**不换行**
   ```rust
   print!("Hello, "); // 不换行
   println!("Rust!"); // 接上一行输出：Hello, Rust!
   ```

3. **格式化输出**（类似 Python 的 f-string 或 C 的 `printf`）
   ```rust
   let name = "Alice";
   let age = 30;
   println!("Name: {}, Age: {}", name, age); // 位置参数
   println!("Age: {1}, Name: {0}", name, age); // 索引参数
   println!("Hex: {:x}, Binary: {:b}", 255, 15); // 格式化数字：十六进制、二进制
   ```

---

### 2. 📤 **基础输入：从标准输入读取**
需要导入 `std::io` 模块。核心是 `io::stdin()` 和 `read_line` 方法。

#### 基本流程（读取字符串）：
```rust
use std::io;

fn main() {
    // 创建可变 String 存储输入
    let mut input = String::new();

    // 提示用户
    println!("请输入文本:");

    // 读取一行（包括换行符）
    io::stdin()
        .read_line(&mut input) // 传入可变引用
        .expect("读取失败");   // 处理可能的错误

    // 打印输入（注意包含换行符）
    println!("你输入的是: {}", input);
}
```

#### 关键点解析：
1. **`let mut input = String::new()`**  
   - 创建可变的空 `String` 来存储输入。

2. **`io::stdin().read_line(&mut input)`**  
   - `io::stdin()`：获取标准输入句柄。
   - `.read_line(&mut input)`：读取一行数据，追加到 `input` 字符串中（**保留换行符 `\n`**）。

3. **`.expect("错误信息")`**  
   - `read_line` 返回 `Result<usize, io::Error>` 类型（成功返回读取字节数，失败返回错误）。
   - `.expect()`：如果发生错误，**使程序崩溃**并显示指定信息（适合简单程序）。
   - 生产代码建议用 `match` 或 `?` 运算符处理错误。

---

### 3. ✂️ **处理输入数据**
#### 1. 去除换行符/空白
`read_line` 读取的内容包含换行符（`\n` 或 `\r\n`），通常需要清理：
```rust
let cleaned_input = input.trim(); // 移除首尾空白字符（包括换行）
println!("清理后: {}", cleaned_input);
```

#### 2. 转换数据类型（字符串 → 数字）
使用 `parse()` 方法配合类型推断：
```rust
let num: i32 = cleaned_input.parse().expect("请输入数字！");
println!("数字 + 1 = {}", num + 1);
```
- `parse()` 返回 `Result<T, ParseIntError>`，需处理错误。
- 更健壮的写法：
  ```rust
  match cleaned_input.parse::<i32>() {
      Ok(n) => println!("数字: {}", n),
      Err(_) => println!("无效数字"),
  }
  ```

---

### 4. 📝 **进阶：文件 I/O（读写文件）**
需导入 `std::fs` 和 `std::io::prelude::*`。

#### 1. 写入文件
```rust
use std::fs::File;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = File::create("output.txt")?; // 创建文件（覆盖已有）
    file.write_all(b"Hello, File!")?;          // 写入字节数据
    Ok(())
}
```

#### 2. 读取文件
```rust
use std::fs;
use std::io;

fn main() -> io::Result<()> {
    let content = fs::read_to_string("input.txt")?; // 读取整个文件为 String
    println!("文件内容:\n{}", content);
    Ok(())
}
```
- 逐行读取（适合大文件）：
  ```rust
  use std::io::{BufRead, BufReader};
  
  let file = File::open("data.txt")?;
  let reader = BufReader::new(file);
  for line in reader.lines() { // lines() 返回迭代器
      println!("{}", line?);   // 每行也是 Result
  }
  ```

---

### 5. 🚨 **错误处理核心：`Result` 类型**
Rust 的 I/O 操作几乎都返回 `Result<T, io::Error>`：
- **`Ok(T)`**：操作成功，包含结果值（如读取的字节数）。
- **`Err(io::Error)`**：操作失败，包含错误信息。

#### 推荐处理方式：
1. **`?` 运算符**（在返回 `Result` 的函数中使用）
   ```rust
   fn read_file() -> io::Result<String> {
       let mut s = String::new();
       io::stdin().read_line(&mut s)?; // 出错时自动返回错误
       Ok(s)
   }
   ```

2. **`match` 表达式**（显式处理）
   ```rust
   let result = io::stdin().read_line(&mut input);
   match result {
       Ok(bytes) => println!("读取了 {} 字节", bytes),
       Err(e) => eprintln!("错误: {}", e), // eprintln! 输出到标准错误
   }
   ```

---

### 6. 💡 **最佳实践总结**
1. **输出**：优先用 `println!`/`print!` + 格式化。
2. **输入**：
   - 用 `read_line` 读取用户输入。
   - 立即用 `.trim()` 清理数据。
   - 用 `parse()` + 模式匹配转换类型。
3. **错误处理**：
   - 简单场景用 `.expect("提示")` 快速失败。
   - 正式代码用 `?` 或 `match` 传播/处理错误。
4. **文件操作**：
   - 小文件用 `fs::read_to_string`/`fs::write`。
   - 大文件用 `BufReader`/`BufWriter` 提升性能。

---

### 7. 📚 示例：完整的用户输入程序
```rust
use std::io;

fn main() {
    println!("请输入两个整数：");

    let mut input1 = String::new();
    io::stdin().read_line(&mut input1).expect("读取失败");
    let num1: i32 = input1.trim().parse().expect("请输入整数！");

    let mut input2 = String::new();
    io::stdin().read_line(&mut input2).expect("读取失败");
    let num2: i32 = input2.trim().parse().expect("请输入整数！");

    println!("{} + {} = {}", num1, num2, num1 + num2);
}
```

掌握这些核心操作，你就能处理 Rust 中 90% 的基础 I/O 需求！🚀 后续可深入学习 `std::io` 模块的异步 I/O、缓冲区定制等高级特性。



## Rust：变量

Rust 的变量设计有独特的哲学，核心是**安全性和明确性**，主要体现在所有权、可变性和类型系统上。以下是关键知识点：

---

### 1. 🔒 变量声明基础
- **`let` 关键字**：声明变量的唯一方式
  ```rust
  let x = 5;          // 类型自动推断为 i32
  let name: &str = "Alice"; // 显式标注类型
  ```
- **默认不可变**（重要安全特性）：
  ```rust
  let count = 10;
  count = 20; // 编译错误！不能修改不可变变量
  ```

---

### 2. 🔄 可变性 (`mut`)
- **显式声明可变变量**：
  ```rust
  let mut counter = 0;
  counter += 1; // 允许修改
  ```
- **设计哲学**：
  - 默认不可变防止意外修改
  - 需要修改时必须显式声明 `mut`
  - 减少并发环境下的数据竞争风险

---

### 3. 👥 变量遮蔽 (Shadowing)
- **同作用域内重用变量名**（改变类型或值）：
  ```rust
  let spaces = "   ";
  let spaces = spaces.len(); // 从字符串变为整数，合法！
  
  let mut spaces = "   ";
  spaces = spaces.len(); // 错误！不能改变类型
  ```
- **与重新赋值的区别**：
  - 遮蔽创建新绑定（可改变类型）
  - `mut` 变量只能改变值（不能改变类型）

---

### 4. 📦 常量 (`const`)
- **编译时确定的常量值**：
  ```rust
  const MAX_POINTS: u32 = 100_000; // 必须显式标注类型
  ```
- 特点：
  - 命名规范：全大写+下划线分隔
  - 生命周期：整个程序运行期
  - 只能使用常量表达式初始化

---

### 5. 🌐 静态变量 (`static`)
- **全局变量**（有内存地址）：
  ```rust
  static HELLO: &str = "Hello";
  ```
- **可变静态变量**（`unsafe`）：
  ```rust
  static mut COUNTER: u32 = 0;
  
  unsafe {
      COUNTER += 1; // 需要 unsafe 块
  }
  ```

---

###  6. 🧩解构赋值
- **从复合类型提取值**：
  ```rust
  // 元组解构
  let (x, y, z) = (1, 2, 3); 
  
  // 结构体解构
  struct Point { x: i32, y: i32 }
  let p = Point { x: 10, y: 20 };
  let Point { x: a, y: b } = p;
  ```

---

###  7. 🎯忽略值 (`_`)
- **忽略未使用的变量**：
  ```rust
  let _unused = 42; // 编译器不会警告
  ```
- **部分忽略**：
  ```rust
  let (x, _, z) = (1, 2, 3); // 忽略第二个值
  ```

---

### 8. ⚙ 所有权机制（核心！）
- **移动语义**（默认）：
  ```rust
  let s1 = String::from("hello");
  let s2 = s1; // s1 的所有权转移给 s2
  // println!("{}", s1); // 错误！s1 已失效
  ```
- **克隆**（深拷贝）：
  ```rust
  let s3 = s2.clone(); // 显式复制数据
  ```
- **复制语义**（仅限栈数据）：
  ```rust
  let a = 5;
  let b = a; // 自动复制，因为 i32 实现 Copy trait
  ```

---

### 9. 🔗 引用与借用
- **不可变引用**（共享）：
  ```rust
  let s = String::from("hello");
  let len = calculate_length(&s); // 借用 s
  
  fn calculate_length(s: &String) -> usize {
      s.len()
  }
  ```
- **可变引用**（独占）：
  ```rust
  let mut s = String::from("hello");
  modify_string(&mut s); // 可变借用
  
  fn modify_string(s: &mut String) {
      s.push_str(" world");
  }
  ```
- **借用规则**（编译期检查）：
  1. 同一时间，一个数据要么有多个不可变引用，要么只能有一个可变引用
  2. 引用必须总是有效的（无悬垂指针）

---

### 10. ⏳ 生命周期注解（高级）
- **确保引用有效性**：
  ```rust
  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
      if x.len() > y.len() { x } else { y }
  }
  ```

---

### 11. 💡 最佳实践总结
| 场景                | 推荐方式                        |
| ------------------- | ------------------------------- |
| 不需要修改的值      | `let x = value;`                |
| 需要修改的值        | `let mut x = value;`            |
| 重用变量名/改变类型 | 变量遮蔽                        |
| 全局常量            | `const`                         |
| 全局可变状态        | `static mut` + `unsafe`（慎用） |
| 避免所有权转移      | 使用引用 `&` 或 `&mut`          |
| 需要深度复制        | `.clone()`                      |
| 忽略未使用变量      | `_` 前缀                        |

---

### 12. 🧪 示例：综合运用
```rust
fn main() {
    // 不可变变量
    let name = "Alice";
    
    // 可变变量
    let mut age = 30;
    age += 1;
    
    // 变量遮蔽 (改变类型)
    let age = format!("{} years", age); // 从 i32 变为 String
    
    // 常量
    const MAX_AGE: u32 = 150;
    
    // 解构赋值
    let (x, y) = (10, 20);
    
    // 所有权转移
    let s1 = String::from("hello");
    let s2 = s1; // s1 失效
    
    // 借用
    let len = calculate_length(&s2);
    
    println!("{} is {}, max allowed: {}", name, age, MAX_AGE);
    println!("String '{}' has length {}", s2, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

Rust 的变量系统是其内存安全和并发安全的基石。理解这些概念（尤其所有权和借用）是掌握 Rust 的关键！🚀



## Rust：数据类型

Rust 是静态类型语言，具有强大而灵活的类型系统，旨在确保内存安全和高效执行。以下是 Rust 数据类型体系的全面介绍：

### 1.  标量类型 (Scalar Types)

#### 1.1 整型 (Integer Types)

| 类型            | 有符号范围      | 无符号范围 | 大小     |
| --------------- | --------------- | ---------- | -------- |
| `i8`/`u8`       | -128..127       | 0..255     | 8位      |
| `i16`/`u16`     | -32,768..32,767 | 0..65,535  | 16位     |
| `i32`/`u32`     | -2.1e9..2.1e9   | 0..4.3e9   | 32位     |
| `i64`/`u64`     | -9.2e18..9.2e18 | 0..1.8e19  | 64位     |
| `i128`/`u128`   | -1.7e38..1.7e38 | 0..3.4e38  | 128位    |
| `isize`/`usize` | 平台相关        | 平台相关   | 指针大小 |

**字面值表示：**
```rust
let decimal = 98_222;      // 十进制
let hex = 0xff;            // 十六进制
let octal = 0o77;          // 八进制
let binary = 0b1111_0000;  // 二进制
let byte = b'A';           // 字节(u8)
```

#### 1.2 浮点型 (Floating-Point Types)

| 类型  | 精度   | 大小 | 范围                |
| ----- | ------ | ---- | ------------------- |
| `f32` | 单精度 | 32位 | ±1.2e-38..±3.4e38   |
| `f64` | 双精度 | 64位 | ±2.2e-308..±1.8e308 |

```rust
let x = 2.0;      // 默认 f64
let y: f32 = 3.0; // 显式指定 f32
```

#### 1.3 布尔型 (Boolean)

```rust
let t = true;
let f: bool = false; // 显式类型注解
```

#### 1.4 字符类型 (Character)

- Unicode 标量值（4字节）
- 使用单引号 `' '` 表示

```rust
let c = 'z';
let z = 'ℤ';
let heart = '❤';
```

### 2. 复合类型 (Compound Types)

#### 2.1 元组 (Tuple)

- 固定长度
- 可包含不同类型

```rust
// 类型注解
let tup: (i32, f64, u8) = (500, 6.4, 1);

// 解构
let (x, y, z) = tup;

// 索引访问
let first = tup.0; // 500
```

#### 2.2 数组 (Array)

- 固定长度
- 元素类型必须相同
- 栈上分配

```rust
// 类型注解 [类型; 长度]
let a: [i32; 5] = [1, 2, 3, 4, 5];

// 初始化相同值
let b = [3; 5]; // [3, 3, 3, 3, 3]

// 访问元素
let first = a[0];
```

**数组越界检查：**
```rust
let index = 10;
// 运行时检查，越界会panic
let element = a[index]; 
```

### 3. 自定义类型

#### 3.1 结构体 (Struct)

```rust
// 定义
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// 实例化
let user1 = User {
    email: String::from("user@example.com"),
    username: String::from("someusername"),
    active: true,
    sign_in_count: 1,
};

// 更新语法
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername"),
    ..user1
};
```

#### 3.2 枚举 (Enum)

```rust
// 定义
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

// 使用
let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));

// 带数据的枚举
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

#### 3.3  联合体 (Union) - 不安全

```rust
union MyUnion {
    f1: u32,
    f2: f32,
}
```

### 4. 引用类型

#### 4.1 引用 (Reference)

- 借用数据的所有权
- 编译时检查借用规则

```rust
let s1 = String::from("hello");
let len = calculate_length(&s1); // 不可变引用

fn calculate_length(s: &String) -> usize {
    s.len()
}

let mut s = String::from("hello");
change(&mut s); // 可变引用

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

#### 4.2 裸指针 (Raw Pointer) - 不安全

```rust
let mut num = 5;
let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

### 5. 智能指针

#### 5.1 `Box<T>` - 堆上分配

```rust
let b = Box::new(5); // 堆上存储5
```

#### 5.2 `Rc<T>` - 引用计数

```rust
use std::rc::Rc;

let a = Rc::new(String::from("hello"));
let b = Rc::clone(&a); // 共享所有权
```

#### 5.3 `RefCell<T>` - 内部可变性

```rust
use std::cell::RefCell;

let c = RefCell::new(5);
*c.borrow_mut() = 10; // 运行时借用检查
```

### 6. 特殊类型

#### 6.1 单元类型 `()`

- 表示"无值"
- 类似其他语言的 `void`

```rust
fn no_return() -> () {
    // 没有返回值
}
```

#### 6.2 Never 类型 `!`

- 表示永不返回的函数
- 用于发散函数

```rust
fn forever() -> ! {
    loop {
        // 无限循环
    }
}
```

### 7. 类型别名

```rust
type Kilometers = i32;

let distance: Kilometers = 5;
```

### 8. 类型转换

#### 8.1 显式转换 (as)

```rust
let x: i32 = 5;
let y: u64 = x as u64;
```

#### 8.2 类型推断

```rust
let x = 5; // 默认 i32
let y: u8 = 10; // 显式类型
```

#### 8.3 From/Into trait

```rust
let s = String::from("hello");
let bytes: Vec<u8> = s.into(); // 使用Into trait
```

### 9. 泛型类型

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

### 10. 总结表

| 类别         | 类型示例                  | 特点                         |
| ------------ | ------------------------- | ---------------------------- |
| 标量类型     | i32, f64, bool, char      | 单一值                       |
| 复合类型     | tuple, array              | 固定大小，多值               |
| 自定义类型   | struct, enum              | 用户定义数据结构             |
| 引用类型     | &T, &mut T                | 借用所有权                   |
| 智能指针     | Box<T>, Rc<T>, RefCell<T> | 堆分配、共享所有权、内部可变 |
| 特殊类型     | (), !                     | 无值、永不返回               |
| 动态大小类型 | str, [T]                  | 运行时确定大小               |
| 泛型         | T, U                      | 抽象类型参数                 |

Rust 的类型系统通过以下方式确保安全：
1. **所有权机制**：防止内存泄漏和数据竞争
2. **借用检查器**：在编译时验证引用有效性
3. **生命周期注解**：确保引用不会变成悬垂引用
4. **类型推断**：减少冗余的类型注解
5. **模式匹配**：确保所有情况都被处理

理解 Rust 的类型系统是掌握这门语言的关键，它为内存安全和并发安全提供了坚实基础。



## Rust：控制流

Rust 提供了多种控制流结构，让您能够根据条件执行代码或重复执行代码块。这些结构包括条件表达式、循环和模式匹配。

### 1. 条件表达式（`if`）

#### 1.1 基本 `if` 表达式
```rust
let number = 7;

if number < 5 {
    println!("条件成立");
} else {
    println!("条件不成立");
}
```

#### 1.2 多分支 `else if`
```rust
let number = 6;

if number % 4 == 0 {
    println!("能被4整除");
} else if number % 3 == 0 {
    println!("能被3整除");
} else if number % 2 == 0 {
    println!("能被2整除");
} else {
    println!("不能被2、3、4整除");
}
```

#### 1.3 `if` 在 `let` 语句中使用
```rust
let condition = true;
let number = if condition { 5 } else { 6 };

println!("number 的值为: {}", number);
```

### 2. 循环（Loop）

#### 2.1 `loop` - 无限循环
```rust
let mut count = 0;
loop {
    count += 1;
    println!("计数: {}", count);
    
    if count == 5 {
        println!("达到5次，退出循环!");
        break;
    }
}
```

#### 2.2 从循环返回值

```rust
let mut counter = 0;

let result = loop {
    counter += 1;
    
    if counter == 10 {
        break counter * 2; // 返回20
    }
};

println!("结果是: {}", result); // 输出: 20
```

#### 2.3 `while` - 条件循环
```rust
let mut number = 3;

while number != 0 {
    println!("{}!", number);
    number -= 1;
}

println!("发射!");
```

#### 2.4 `for` - 遍历集合
```rust
let a = [10, 20, 30, 40, 50];

for element in a.iter() {
    println!("值为: {}", element);
}
```

#### 2.5 使用范围表达式
```rust
// 从1到4（不含5）
for number in 1..5 {
    println!("{}!", number);
}

// 从1到4（含5）
for number in 1..=5 {
    println!("{}!", number);
}

// 反向遍历
for number in (1..5).rev() {
    println!("{}!", number);
}
```

### 3. 模式匹配（`match`）

#### 3.1 基本匹配
```rust
let coin = Coin::Penny;
let value = match coin {
    Coin::Penny => 1,
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quarter => 25,
};
```

#### 3.2 绑定值的模式
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // ...
}

enum Coin {
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Quarter(state) => {
            println!("州硬币: {:?}!", state);
            25
        },
        _ => 0,
    }
}
```

#### 3.3 匹配 `Option<T>`
```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five); // Some(6)
let none = plus_one(None); // None
```

#### 3.4 通配模式 `_`
```rust
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => (), // 其他情况不做任何操作
}
```

### 4. `if let` 简洁控制流

#### 4.1 基本用法
```rust
let some_value = Some(3);

match some_value {
    Some(3) => println!("是3"),
    _ => (),
}

// 等价的 if let 表达式
if let Some(3) = some_value {
    println!("是3");
}
```

#### 4.2 带 `else` 的 `if let`
```rust
let coin = Coin::Quarter(UsState::Alaska);
let mut count = 0;

if let Coin::Quarter(state) = coin {
    println!("州的25美分硬币: {:?}!", state);
} else {
    count += 1;
}
```

### 5. 控制流最佳实践

1. **优先使用 `for` 循环**：遍历集合时更安全，避免索引越界
2. **使用 `match` 处理所有情况**：确保代码覆盖所有可能性
3. **`if let` 简化单一匹配**：当只关心一种模式时使用
4. **循环标签处理嵌套循环**：
   ```rust
   'outer: loop {
       'inner: loop {
           break 'outer; // 直接跳出外层循环
       }
   }
   ```

### 6. 控制流总结表

| 结构     | 用途           | 特点                     |
| -------- | -------------- | ------------------------ |
| `if`     | 条件分支       | 可返回表达式值           |
| `loop`   | 无限循环       | 可用`break`返回值        |
| `while`  | 条件循环       | 每次迭代前检查条件       |
| `for`    | 遍历集合或范围 | 安全且简洁               |
| `match`  | 多模式匹配     | 必须穷尽所有可能         |
| `if let` | 单一模式匹配   | 简化只关心一种模式的情况 |

### 7. 完整示例：猜数字游戏

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("猜数字游戏!");
    
    // 生成1-100的随机数
    let secret_number = rand::thread_rng().gen_range(1..=100);
    
    loop {
        println!("请输入你的猜测:");
        
        let mut guess = String::new();
        
        io::stdin()
            .read_line(&mut guess)
            .expect("读取失败");
        
        // 转换类型并处理错误
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("请输入有效数字!");
                continue;
            }
        };
        
        println!("你猜的是: {}", guess);
        
        // 比较猜测和秘密数字
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("太小了!"),
            Ordering::Greater => println!("太大了!"),
            Ordering::Equal => {
                println!("恭喜你，猜对了!");
                break;
            }
        }
    }
}
```

Rust 的控制流结构设计既安全又富有表达力，特别是 `match` 的穷尽性检查和循环中的所有权规则，可以有效避免许多常见错误。



## Rust：函数

Rust 的函数系统是其核心特性之一，强调**明确性**、**安全性**和**表达力**。以下是 Rust 函数的完整介绍：

### 1. 基本函数定义

#### 1.1 基本语法
```rust
// 无参数无返回值
fn greet() {
    println!("Hello, world!");
}

// 带参数
fn greet_name(name: &str) {
    println!("Hello, {}!", name);
}

// 带返回值
fn add(a: i32, b: i32) -> i32 {
    a + b  // 注意：没有分号，这是表达式
}

// 显式返回
fn sub(a: i32, b: i32) -> i32 {
    return a - b;  // 使用return关键字
}
```

#### 1.2 函数调用

```rust
fn main() {
    greet();
    greet_name("Alice");
    let sum = add(5, 3);
    println!("5 + 3 = {}", sum);
}
```

### 2. 函数参数详解

#### 2.1 参数类型注解（必需）
```rust
fn print_coordinates((x, y): (i32, i32)) {
    println!("坐标: ({}, {})", x, y);
}
```

#### 2.2 引用参数（避免所有权转移）
```rust
fn calculate_length(s: &String) -> usize {
    s.len()  // 借用而不获取所有权
}

fn change_string(s: &mut String) {
    s.push_str(" world");
}
```

#### 2.3 默认参数（不支持）
Rust **没有默认参数**，替代方案：
```rust
// 替代方案1：使用Option
fn greet_opt(name: Option<&str>) {
    match name {
        Some(n) => println!("Hello, {}!", n),
        None => println!("Hello, stranger!"),
    }
}

// 替代方案2：使用不同函数
fn greet() { greet_name("world"); }
fn greet_name(name: &str) { /* ... */ }
```

### 3. 返回值处理

#### 3.1 隐式返回（无分号的表达式）
```rust
fn five() -> i32 {
    5  // 没有分号，作为返回值
}
```

#### 3.2 显式返回（return 关键字）

```rust
fn early_return(x: i32) -> i32 {
    if x < 0 {
        return -1;  // 提前返回
    }
    x * 2
}
```

#### 3.3 返回元组（多值返回）
```rust
fn calculate(x: i32, y: i32) -> (i32, i32) {
    (x + y, x * y)
}

// 解构返回值
let (sum, product) = calculate(4, 5);
```

#### 3.4 返回 Result（错误处理）
```rust
fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err(String::from("除数不能为零"))
    } else {
        Ok(a / b)
    }
}
```

### 4. 特殊函数类型

#### 4.1 关联函数（类似静态方法）
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 关联函数（无self参数）
    fn square(size: u32) -> Self {
        Rectangle {
            width: size,
            height: size,
        }
    }
}

// 使用双冒号调用
let sq = Rectangle::square(10);
```

#### 4.2 方法（带self参数）
```rust
impl Rectangle {
    // 不可变借用self
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    // 可变借用self
    fn resize(&mut self, width: u32, height: u32) {
        self.width = width;
        self.height = height;
    }
}

// 使用点号调用
let mut rect = Rectangle { width: 30, height: 50 };
println!("面积: {}", rect.area());
rect.resize(40, 60);
```

#### 4.3 函数指针
```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

// 函数指针类型
let func_ptr: fn(i32) -> i32 = add_one;
println!("5 + 1 = {}", func_ptr(5));
```

### 5. 高级函数特性

#### 5.1 泛型函数
```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

#### 5.2 闭包作为参数（函数式编程）
```rust
fn apply_twice<F>(f: F, x: i32) -> i32 
where
    F: Fn(i32) -> i32,
{
    f(f(x))
}

let result = apply_twice(|x| x * 2, 5); // 20
```

#### 5.3 返回闭包
```rust
fn make_adder(n: i32) -> impl Fn(i32) -> i32 {
    move |x| x + n  // move捕获环境
}

let add_five = make_adder(5);
println!("5 + 10 = {}", add_five(10)); // 15
```

### 6. 函数与所有权规则

#### 6.1 所有权转移
```rust
fn take_ownership(s: String) {
    println!("取得所有权: {}", s);
} // s被丢弃

fn main() {
    let s = String::from("hello");
    take_ownership(s);
    // println!("{}", s); // 错误！s的所有权已转移
}
```

#### 6.2 借用避免转移
```rust
fn borrow_string(s: &String) {
    println!("借用: {}", s);
}

fn main() {
    let s = String::from("hello");
    borrow_string(&s);
    println!("仍可使用: {}", s); // 有效
}
```

### 7. 函数最佳实践

1. **命名规范**：
   - 函数名：蛇形命名法（snake_case）
   - 类型名：大驼峰命名法（PascalCase）

2. **参数顺序**：
   ```rust
   // 好的顺序：输入参数 → 输出参数
   fn transform(input: &str, output: &mut String) { /* ... */ }
   ```

3. **错误处理**：
   - 优先返回 `Result` 而不是 panic!
   - 使用 `?` 运算符传播错误

4. **文档注释**：
   ```rust
   /// 计算两个数的和
   /// 
   /// # 示例
   /// ```
   /// let s = add(2, 3);
   /// assert_eq!(s, 5);
   /// ```
   fn add(a: i32, b: i32) -> i32 {
       a + b
   }
   ```

### 8. 完整示例

```rust
/// 计算斐波那契数列第n项
/// 
/// # 参数
/// - `n`: 斐波那契数列位置（从0开始）
/// 
/// # 返回值
/// 第n项的值
/// 
/// # 示例
/// ```
/// assert_eq!(fibonacci(10), 55);
/// ```
fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => {
            let mut a = 0;
            let mut b = 1;
            for _ in 2..=n {
                let next = a + b;
                a = b;
                b = next;
            }
            b
        }
    }
}

fn main() {
    for n in 0..10 {
        println!("fib({}) = {}", n, fibonacci(n));
    }
    
    // 使用闭包
    let fib_closure = |n: u32| -> u64 {
        if n == 0 { 0 }
        else if n == 1 { 1 }
        else { fibonacci(n - 1) + fibonacci(n - 2) }
    };
    
    println!("闭包计算: fib(5) = {}", fib_closure(5));
}
```

Rust 的函数系统通过以下方式确保安全性和表达力：
1. **强类型系统**：所有参数和返回值必须明确类型
2. **所有权规则**：明确区分值、引用和可变引用
3. **错误处理**：使用 `Result` 和 `Option` 强制处理潜在错误
4. **零成本抽象**：泛型和闭包在编译时展开，无运行时开销

掌握 Rust 的函数是理解该语言所有权系统和高效编程的关键！
