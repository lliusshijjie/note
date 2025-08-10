# GDB

 **GDB (GNU Debugger)**，它是 Linux/Unix 环境下最经典、最强大的**命令行调试工具**，主要用于调试 C 和 C++ 程序，但也支持其他语言（如 Go, Rust, D, Fortran 等）。

## 一、**为什么需要 GDB？**

* **定位错误：** 程序崩溃（Segmentation Fault, Bus Error）、逻辑错误（结果不对）、死锁、挂起等。

* **理解程序行为：** 跟踪代码执行流程，观察变量在运行时的变化。

* **分析核心转储：** 程序崩溃后产生的 `core` 文件包含了崩溃时的状态，GDB 可以加载它进行分析。

* **没有图形界面依赖：** 在服务器环境或远程调试时，命令行工具至关重要。

  

## 二、**GDB 的核心功能：**

### 1. **启动程序：**

* `gdb <program>`: 启动 GDB 并加载可执行文件 `program`。

* `gdb <program> <core>`: 加载可执行文件和崩溃产生的核心转储文件 `core` 进行事后分析。

* `gdb --args <program> <arg1> <arg2> ...`: 启动 GDB 并加载程序，同时指定运行参数。

  

### 2. **控制程序执行：**

* `run [args]` / `r [args]`: 从头开始运行程序（可带参数）。这是调试的起点。

* `start [args]`: 在 `main` 函数处设置一个临时断点并开始运行（停在 `main` 第一行）。

* `continue` / `c`: 从当前停止点继续运行程序，直到遇到下一个断点、信号或程序结束。

* `next` / `n`: **单步执行（Step Over）**。执行下一行代码。如果该行包含函数调用，**不会**进入该函数内部，而是将整个函数当作一步执行完。

* `step` / `s`: **单步进入（Step Into）**。执行下一行代码。如果该行包含函数调用，**会**进入该函数的内部（前提是该函数有调试信息）。

* `finish` / `fin`: **单步跳出（Step Out）**。继续执行，直到当前函数执行完毕并返回，然后暂停。

* `until [location]` / `u [location]`: 继续运行，直到到达指定的行号或地址。常用于快速跳出循环。

* `kill` / `k`: 终止正在调试的程序（但 GDB 会话还在）。

  

### 3. **设置断点 (Breakpoints):** 这是调试的核心，让程序在特定位置暂停。

*   `break <location>` / `b <location>`: 在指定位置设置断点。`<location>` 可以是：
    
    *   **函数名:** `b main`, `b my_function`
    *   **源文件行号:** `b file.c:20`
    *   **相对当前文件的行号:** `b 20` (如果当前在 `file.c` 上下文中)
    *   **地址:** `b *0x4005a7`
    
* `break ... if <condition>`: 设置**条件断点**。仅当 `<condition>` 为真时，断点才触发。例如：`b myloop.c:10 if i == 50`。

* `watch <expression>`: 设置**观察点 (Watchpoint)**。当 `<expression>` 的值**被改变**时暂停程序。例如：`watch my_variable`, `watch *(int*)0x7fffffffe234`。非常有用于追踪变量何时被意外修改。

* `catch ...`: 设置**捕获点 (Catchpoint)**。用于捕获特定事件，如抛出异常 (`catch throw`)、加载共享库 (`catch load`)、系统调用 (`catch syscall open`) 等。

* `info breakpoints` / `i b`: 列出所有设置的断点、观察点、捕获点及其编号。

* `delete [breakpoint-numbers]` / `d [breakpoint-numbers]`: 删除指定的断点（编号来自 `info b`）。不带参数删除所有断点。

* `disable [breakpoint-numbers]`: 禁用断点（不删除，可重新启用）。

* `enable [breakpoint-numbers]`: 启用被禁用的断点。

  

### 4. **检查程序状态和数据：** 程序暂停时，查看发生了什么。

* `print <expression>` / `p <expression>`: 打印变量或表达式的值。例如：`p my_var`, `p *ptr`, `p my_struct.member`, `p strlen(my_string)`, `p/x my_var` (十六进制格式), `p/d my_var` (十进制), `p/t my_var` (二进制), `p my_array[5]@10` (打印数组从索引 5 开始的 10 个元素)。

* `display <expression>`: 每次程序暂停时**自动打印**指定表达式的值。例如：`display my_counter`。

* `info locals` / `i locals`: 打印当前栈帧（函数）的所有局部变量。

* `info args` / `i args`: 打印当前函数的参数值。

* `info registers` / `i r`: 打印 CPU 寄存器的值（底层调试）。

* `info frame` / `i f`: 显示当前栈帧的详细信息（函数、参数地址、局部变量地址等）。

* `backtrace` / `bt`: 打印**调用栈 (Call Stack)**。显示当前暂停位置是如何被调用到达的（从当前函数回溯到 `main`）。这是理解程序流程和崩溃位置的关键。

* `frame <frame-number>` / `f <frame-number>`: 切换到调用栈中的特定栈帧（编号来自 `bt`）。然后可以使用 `info locals`, `info args`, `print` 等命令查看该帧的上下文。例如 `f 1`。

*   `x/<format> <address>`: **检查内存 (Examine Memory)**。从指定地址开始按特定格式显示内存内容。非常底层但强大。
    * **格式:** 由 `<count><format><size>` 组成。
    
    * `<count>`: 要显示多少个单位。
    
    * `<format>`: `x`(十六进制), `d`(十进制), `u`(无符号十进制), `o`(八进制), `t`(二进制), `a`(地址), `i`(指令), `c`(字符), `s`(字符串)。
    
    * `<size>`: `b`(byte), `h`(halfword, 2 bytes), `w`(word, 4 bytes), `g`(giant word, 8 bytes)。
    
    *   例子：
        * `x/10xw 0x7fffffffe220`: 从地址 `0x7fffffffe220` 开始，以十六进制(`x`)格式显示 10(`10`) 个 4 字节(`w`)的值。
        
        * `x/s 0x4006d4`: 将以 `0x4006d4` 地址开始的字节解释为字符串(`s`)并打印。
        
        * `x/i $pc`: 显示当前程序计数器(`$pc`)指向的指令(`i`)。
        
          

### 5. **多线程/多进程调试：**

* `info threads` / `i threads`: 列出所有线程及其状态。

* `thread <thread-id>`: 切换到指定 ID 的线程进行调试。

* `set scheduler-locking on|off|step`: 控制线程调度锁定。`on` 确保只有当前线程运行；`step` 在单步时锁定；`off` 所有线程自由运行。调试多线程竞态条件时常用 `on` 或 `step`。

* `attach <pid>`: 将 GDB 附加到一个**正在运行**的进程（通过进程 ID `pid`）。非常有用于调试后台服务或挂起的程序。

* `detach`: 从已附加的进程中分离，让该进程继续独立运行。

* `fork` / `vfork`: GDB 可以跟踪子进程（`set follow-fork-mode child/parent`）或在 fork 后暂停（`set detach-on-fork on/off`）。

  

### 6. **调试核心转储 (Core Dump):**

* 确保系统允许生成核心转储 (`ulimit -c unlimited`)。

* 程序崩溃后，会产生 `core` 或 `core.<pid>` 文件。

* 使用 `gdb <program> <core-file>` 加载程序和核心文件。

* 立即使用 `bt` 查看崩溃时的调用栈，定位出错位置。

* 使用 `info locals`, `print`, `x` 等命令检查崩溃时的变量和内存状态。

  

### 7. **脚本化和自动化：**

* **命令文件:** 将 GDB 命令写入一个文本文件（如 `cmds.gdb`），启动时用 `gdb -x cmds.gdb --args ./program` 执行，或运行时用 `source cmds.gdb` 加载。用于自动化测试或重复调试任务。

* **GDB 脚本语言 (Python 集成):** 现代 GDB 内置了 Python 支持 (`gdb` Python 模块)，允许编写复杂的调试脚本、自定义命令、美化打印结构体等，极大地扩展了功能。

  

### 8. **内存调试 (高级):**

* 可以与内存调试器如 **Valgrind** 结合使用（例如，Valgrind 发现错误后生成核心转储，再用 GDB 加载分析）。

* 使用 GDB 命令检查堆内存分配情况（需要理解堆管理器结构）。

  

## **三、GDB 基本工作流程示例：**

1. **编译带调试信息：** `gcc -g -o myprog myprog.c` (关键是 `-g` 选项)。

2. **启动 GDB：** `gdb ./myprog`

3. **设置断点：** `(gdb) b main` (在 `main` 函数入口暂停) 或 `(gdb) b file.c:42` (在 `file.c` 的第 42 行暂停)。

4. **运行程序：** `(gdb) run [arguments]`

5. **程序在断点处暂停。**

6.  **检查状态：**
    *   `(gdb) bt` (查看调用栈)
    *   `(gdb) info locals` (查看局部变量)
    *   `(gdb) p suspicious_var` (打印特定变量)
    
7.  **控制执行：**
    *   `(gdb) n` (执行下一行，不进入函数)
    *   `(gdb) s` (执行下一行，进入函数)
    *   `(gdb) c` (继续运行到下一个断点或结束)
    *   `(gdb) finish` (运行完当前函数)
    
8.  **发现错误后：**
    * 修改源代码。
    
    * 重新编译带 `-g`。
    
    * 在 GDB 中重新 `run` (或者先 `kill` 再 `run`)。
    
      

## **四、常见问题与技巧：**

* **"No debugging symbols found" / 变量显示为 `<optimized out>`:** 编译时**必须**加上 `-g` 选项生成调试信息。高优化级别（`-O2`, `-O3`）可能会使某些变量无法观察（被优化掉），调试时建议使用 `-O0 -g`。

* **权限问题 (附加进程/核心转储):** 可能需要 `sudo` 或调整系统设置 (`/proc/sys/kernel/core_pattern`, `ptrace_scope`)。

* **TUI (Text User Interface):** 使用 `gdb -tui` 或 `layout src` 命令进入文本图形界面，同时显示源代码和命令窗口，更直观。

* **`.gdbinit` 文件：** 在用户主目录 (`~/.gdbinit`) 或当前目录创建此文件，GDB 启动时会自动执行其中的命令。常用于设置喜欢的格式、定义别名、加载 Python 脚本等。

* **自定义打印 (Pretty Printing):** 使用 Python 脚本定义如何美观地打印复杂的 C++ STL 容器或自定义数据结构。通常有现成的脚本可用（如 GDB 自带的或项目提供的）。

* **反向调试 (Reverse Debugging):** 使用 `record` / `rr` (Mozilla Record and Replay Framework) 等工具记录执行轨迹，然后允许**反向执行**（`reverse-step`, `reverse-next`, `reverse-continue`），对于复现偶发性错误极其强大（但可能影响性能）。

  

## **五、总结：**

GDB 是一个功能极其强大且灵活的调试器，是深入理解程序行为、定位复杂 Bug（尤其是崩溃、内存错误、并发问题）的必备工具。虽然它最初是命令行界面，学习曲线相对图形化调试器陡峭一些，但其强大的功能、对底层细节的控制能力以及在无图形环境下的可用性，使其成为专业开发者和系统程序员的首选。

掌握 GDB 的核心命令（`run`, `break`, `next`, `step`, `print`, `backtrace`, `frame`, `info`）是基础。随着经验的增长，你可以逐步探索其更高级的功能（多线程、内存检查、脚本、反向调试）。坚持使用和实践是掌握 GDB 的关键！