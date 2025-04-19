## linux 系统学习

### 一. linux 文件系统



**Linux系统中一切皆文件**！

**目录结构：**

<img src="C:\Users\刘世杰\AppData\Roaming\Typora\typora-user-images\image-20250411224228364.png" alt="image-20250411224228364" style="zoom:50%;" />

**/bin (Binary)**

功能：存放基础命令的可执行文件（所有用户可用），如 ls、cp、bash 等。

特点：系统启动或修复时必需的命令，通常为静态链接的程序。



**/sbin (System Binary)**

功能：存放系统管理命令（需root权限），如 fdisk、ifconfig、init。

关键命令：系统初始化、磁盘管理、网络配置等。



**/lib 和 /lib64**

功能：存放/bin和/sbin中程序所需的共享库文件（如 .so 文件）。

注意：/lib64 专用于64位系统（如x86_64架构）。



**/etc (Etcetera)**

功能：系统的全局配置文件。

常见文件：

/etc/passwd：用户账户信息

/etc/fstab：磁盘挂载配置

/etc/ssh/sshd_config：SSH服务配置



**/dev (Devices)**

功能：包含设备文件，如硬盘（/dev/sda）、终端（/dev/tty）、随机数设备（/dev/random）等。

特点：通过文件抽象硬件设备。



**/proc (Process)**

功能：虚拟文件系统，提供内核和进程信息的实时接口。

示例：

/proc/cpuinfo：CPU信息

/proc/meminfo：内存使用情况

/proc/[PID]：特定进程的详细信息



**/sys**

功能：虚拟文件系统，用于访问和配置内核参数（如电源管理、设备驱动）。

与/proc区别：更结构化，常用于驱动开发。



**/boot**

功能：存放启动文件，如内核镜像（vmlinuz）、初始内存盘（initramfs）和引导加载器（如GRUB）。

关键性：损坏可能导致系统无法启动。



**用户相关目录：**
**/home**

功能：普通用户的个人目录，每个用户拥有一个子目录（如 /home/username）。

注意：root用户的家目录是 /root。



**/root**

功能：超级用户（root）的家目录，而非/home/root。



**/usr (Unix System Resources)**

功能：存放用户级应用程序和资源（占磁盘空间最大）。

子目录：

/usr/bin：非必需的用户命令（如 python、gcc）。

/usr/sbin：非必需的系统管理命令。

/usr/lib：/usr/bin 的库文件。

/usr/local：用户手动安装的软件（优先级高于系统包管理器）。



**/var (Variable)**

功能：存放频繁变化的文件，如日志、缓存、邮件等。

关键子目录：

/var/log：系统日志（如 syslog、auth.log）。

/var/cache：应用程序缓存。

/var/spool：队列数据（如打印任务、邮件）。



__运行时和临时目录__
__/run__

功能：存放运行时数据（如PID文件、套接字文件），重启后清空。

替代：现代系统已取代旧的 /var/run。



__/tmp (Temporary)__

功能：临时文件目录，所有用户可读写（默认10天未访问自动删除）。



__其他重要目录__
__/opt (Optional)__

功能：第三方大型软件的安装目录（如Oracle数据库、MATLAB）。

特点：软件的所有文件集中在一个子目录内（如 /opt/google/chrome）。



__/mnt 和 /media__

功能：临时挂载点。

/mnt：管理员手动挂载设备（如硬盘分区）。

/media：自动挂载可移动设备（如U盘、光盘）。



__/srv (Service)__

功能：存放服务相关的数据（如Web服务器的 /srv/http，FTP的 /srv/ftp）。



### 二. Vim 文本编辑器详细使用指南

Vim (Vi IMproved) 是 Linux 系统中最强大、最常用的文本编辑器之一，它是 vi 编辑器的增强版本。Vim 以其高效的操作方式和强大的扩展性著称，尤其适合程序员和系统管理员使用。



#### Vim 的基本模式

Vim 有多种工作模式，主要模式包括：

1. **普通模式 (Normal mode)** - 默认进入的模式，用于导航和操作文本

2. **插入模式 (Insert mode)** - 用于输入和编辑文本

3. **可视模式 (Visual mode)** - 用于选择文本块

4. **命令行模式 (Command-line mode)** - 用于执行命令

   

#### 模式切换

- `i` - 在光标前进入插入模式
- `a` - 在光标后进入插入模式
- `o` - 在当前行下方新建一行并进入插入模式
- `O` - 在当前行上方新建一行并进入插入模式
- `ESC` - 返回普通模式
- `v` - 进入可视模式
- `V` - 进入行可视模式
- `Ctrl+v` - 进入块可视模式
- `:` - 进入命令行模式



#### 基本操作:

1. 移动光标

- `h` - 左移

- `j` - 下移

- `k` - 上移

- `l` - 右移

- `w` - 移动到下一个单词开头

- `b` - 移动到上一个单词开头

- `e` - 移动到单词末尾

- `0` - 移动到行首

- `$` - 移动到行尾

- `gg` - 移动到文件开头

- `G` - 移动到文件末尾

- `:n` - 跳转到第 n 行

  

2. 编辑操作

- `x` - 删除当前字符

- `dd` - 删除当前行

- `yy` - 复制当前行

- `p` - 粘贴

- `u` - 撤销

- `Ctrl+r` - 重做

- `.` - 重复上次操作

  

3. 搜索与替换

- `/pattern` - 向前搜索 pattern

- `?pattern` - 向后搜索 pattern

- `n` - 下一个匹配项

- `N` - 上一个匹配项

- `:%s/old/new/g` - 全局替换 old 为 new

- `:%s/old/new/gc` - 全局替换，每次替换前确认

  

#### 高级功能

1. 多窗口操作

- `:split` - 水平分割窗口
- `:vsplit` - 垂直分割窗口
- `Ctrl+w` 后接方向键 - 在窗口间切换
- `:q` - 关闭当前窗口

2. 标签页

- `:tabnew` - 新建标签页
- `gt` - 切换到下一个标签页
- `gT` - 切换到上一个标签页

3. 宏录制

- `q<letter>` - 开始录制宏到指定字母
- `q` - 停止录制
- `@<letter>` - 执行录制的宏



#### Vim 配置文件

Vim 的配置文件是 `~/.vimrc`，可以自定义 Vim 的行为。常见配置：

```vim
" 显示行号
set number

" 语法高亮
syntax on

" 自动缩进
set autoindent

" 制表符宽度
set tabstop=4
set shiftwidth=4
set expandtab

" 显示光标所在行列
set cursorline
set cursorcolumn

" 搜索时忽略大小写
set ignorecase
set smartcase
```

## 五、插件管理

现代 Vim 通常使用插件管理器如 Vundle 或 vim-plug 来扩展功能：

1. 安装 vim-plug:
   ```sh
   curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
   https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
   ```

2. 在 `.vimrc` 中添加插件:
   ```vim
   call plug#begin('~/.vim/plugged')
   Plug 'tpope/vim-fugitive'  " Git 集成
   Plug 'scrooloose/nerdtree' " 文件浏览器
   Plug 'ycm-core/YouCompleteMe' " 代码补全
   call plug#end()
   ```

3. 在 Vim 中运行 `:PlugInstall` 安装插件

## 六、常用命令

- `:w` - 保存文件
- `:q` - 退出 Vim
- `:wq` 或 `:x` - 保存并退出
- `:q!` - 不保存强制退出
- `:help <command>` - 查看命令帮助

