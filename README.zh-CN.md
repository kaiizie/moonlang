# MoonLang Compiler (C++ Source)

**[English](README.md)** | **中文** · **官方网站：[moon-lang.com](https://moon-lang.com)**

---

MoonLang 是一门采用 C++ 和 LLVM 实现的轻量级静态编程语言，支持双语法风格（`: end` 和 `{ }`）。支持 Windows、Linux、macOS 和嵌入式平台（ARM/RISC-V/ESP32），编译后仅 15KB–300KB，可构建从 MCU 到桌面的全场景应用。

本仓库为**编译器源码**（前端 + LLVM 后端 + 运行时），不含标准库 .moon、示例与可执行文件。

---

## 目录

- [支持的功能](#支持的功能)
- [示例](#示例)
- [语法简介](#语法简介)
  - [数据类型与变量](#数据类型与变量)
  - [双语法风格与控制流](#双语法风格与控制流)
  - [函数与 Lambda](#函数与-lambda)
  - [类与面向对象](#类与面向对象)
  - [模块与注释](#模块与注释)
- [部署与使用 moonc](#部署与使用-moonc)
  - [编译为独立可执行文件](#编译为独立可执行文件)
  - [编译目标与选项](#编译目标与选项)
  - [导出 DLL 与 SO（共享库）](#导出-dll-与-so共享库)
- [如何编译本仓库](#如何编译本仓库)
  - [Windows (MSVC)](#windows-msvc)
  - [Linux](#linux)
  - [macOS](#macos)
- [构建脚本说明](#构建脚本说明)
- [目录结构概要](#目录结构概要)
- [许可证与版权](#许可证与版权)

---

## 支持的功能

运行时提供完整的内置能力。可通过构建宏 `MOON_NO_XXX` 或编译选项 `moonc --no-xxx` 按需关闭模块以缩小体积。

### 语言与类型

| 类别 | 说明 |
|------|------|
| **类型** | `int`、`float`、`string`、`bool`、`null`、`list`、`dict`、函数、类、闭包、**BigInt**（大整数） |
| **字面量** | 十六进制 `0x`、二进制 `0b`、八进制 `0o`；列表/字典字面量；多行字符串 |
| **运算符** | 算术、比较、逻辑、位运算（`& \| ^ ~ << >>`）、乘方 `^`、字符串拼接 |
| **控制流** | `if/elif/else`、`while`、`for-in`、`for-to`、`switch`、`break`、`continue` |
| **函数** | 具名函数、Lambda、默认参数、闭包、可变参数（通过列表） |
| **面向对象** | 类、继承、`super`、`self`、构造函数 `init` |
| **模块** | `import "path" as name`、`from "path" import a, b` |
| **异常** | try/catch/finally、`throw`、异常值 |

### I/O 与文件系统

| 功能 | 说明 |
|------|------|
| **控制台** | `print`、`input` |
| **文件** | `read_file`、`write_file`、`append_file`（支持二进制） |
| **路径** | `exists`、`is_file`、`is_dir`、`list_dir`、`create_dir`、`file_size`、`getcwd`、`cd` |
| **路径工具** | `join_path`、`basename`、`dirname`、`extension`、`absolute_path`、`copy_file`、`move_file`、`remove_file`、`remove_dir` |

### 日期与时间

| 类别 | 说明 |
|------|------|
| **时间** | `time`、`now`（毫秒）、`unix_time`、`sleep` |
| **格式化/解析** | `date_format`、`date_parse`，支持格式串和可选时区 |
| **分量** | `year`、`month`、`day`、`hour`、`minute`、`second`、`millisecond`、`weekday`、`day_of_year`、`week_of_year`、`quarter` |
| **运算** | `make_time`、`add_seconds/minutes/hours/days/months/years`、`diff_seconds`、`diff_days` |
| **范围** | `start_of_day`、`end_of_day`、`start_of_month`、`end_of_month` |
| **辅助** | `days_in_month`、`is_leap_year`、`is_weekend`、`is_today`、`is_same_day`、`timezone`、`utc_offset`、`set_timezone` |

### 数学与数值

| 类别 | 说明 |
|------|------|
| **基础** | `abs`、`min`、`max`、`pow`、`sqrt`、`floor`、`ceil`、`round`、`sign`、`clamp`、`lerp` |
| **三角** | `sin`、`cos`、`tan`、`asin`、`acos`、`atan`、`atan2`、`degrees`、`radians` |
| **对数/指数** | `log`、`log10`、`log2`、`exp`、`sinh`、`cosh`、`tanh`、`hypot` |
| **统计** | `mean`、`median`（对列表） |
| **随机** | `random_int`、`random_float` |

### 字符串与字节

| 类别 | 说明 |
|------|------|
| **字符串操作** | `len`、子串、`split`、`join`、`replace`、`trim`、`upper`、`lower`、`capitalize`、`title`、`contains`、`starts_with`、`ends_with`、`index_of`、`find`、`repeat`、`chr`、`ord` |
| **判断** | `is_alpha`、`is_digit`、`is_alnum`、`is_space`、`is_lower`、`is_upper` |
| **填充** | `pad_left`、`pad_right`、`ltrim`、`rtrim` |
| **二进制** | `bytes_to_string`（字节列表转字符串）；字符串支持二进制（长度存于头） |
| **WebSocket** | `ws_parse_frame`、`ws_create_frame`（底层帧解析/构造） |

### 列表与字典

| 类别 | 说明 |
|------|------|
| **列表** | 下标、赋值、`append`、`insert`、`pop`、`remove`、`len`、`slice`、`contains`、`index_of`、`reverse`、`sort`、`sum`、`first`、`last`、`take`、`drop`、`shuffle`、`choice`、`unique`、`flatten`、`zip`、`count` |
| **函数式** | `list_map`、`list_filter`、`list_reduce`（回调） |
| **字典** | `get`、`set`、`has_key`、`keys`、`values`、`items`、`delete`、`merge` |
| **范围** | `range(n)` 或 `range(start, end)` 或 `range(start, end, step)` |

### 网络

| 类别 | 说明 |
|------|------|
| **TCP** | `tcp_connect`、`tcp_listen`、`tcp_accept`、`tcp_send`、`tcp_recv`、`tcp_close`；非阻塞：`tcp_set_nonblocking`、`tcp_has_data`、`tcp_select`、`tcp_accept_nonblocking`、`tcp_recv_nonblocking` |
| **UDP** | `udp_socket`、`udp_bind`、`udp_send`、`udp_recv`、`udp_close` |
| **TLS (OpenSSL)** | `tls_connect`、`tls_listen`、`tls_accept`、`tls_send`、`tls_recv`、`tls_recv_all`、`tls_close`；校验/主机名、证书/密钥/CA 加载、`tls_wrap_client` / `tls_wrap_server` |

由 `MOON_HAS_NETWORK` 控制；TLS 由 `MOON_HAS_TLS` 控制（可选 OpenSSL）。Windows 使用 IOCP/WSAPoll，Linux 使用 epoll。

### GUI

| 类别 | 说明 |
|------|------|
| **基础** | `gui_init`、`gui_create`、`gui_show`、`gui_set_title`、`gui_set_size`、`gui_set_position`、`gui_close`、`gui_run`、`gui_quit`、`gui_alert`、`gui_confirm` |
| **高级** | `gui_create_advanced`（无边框、透明、置顶、可调大小等）；**WebView**：`gui_load_url`、`gui_load_html`、`gui_on_message`（JS 与 MoonLang 互通） |
| **托盘** | `gui_tray_create`、`gui_tray_remove`、`gui_tray_set_menu`、`gui_tray_on_click`、`gui_show_window` |

- **Windows**：WebView2（嵌入式 Chromium）。
- **Linux**：WebKitGTK。
- **macOS**：WKWebView。

由 `MOON_HAS_GUI` 控制。`--target=embedded` 与 `--target=mcu` 下不包含 GUI。

### 正则表达式

| 类别 | 说明 |
|------|------|
| **匹配** | `regex_match`、`regex_search`、`regex_test`、`regex_groups`、`regex_named`、`regex_find_all`、`regex_find_all_groups` |
| **替换** | `regex_replace`、`regex_replace_all` |
| **分割** | `regex_split`、`regex_split_n` |
| **预编译** | `regex_compile`，再使用 `regex_match_compiled`、`regex_search_compiled`、`regex_find_all_compiled`、`regex_replace_compiled`、`regex_free` |
| **工具** | `regex_escape`、`regex_error` |

基于 PCRE2。可用 `--no-regex` 或 `MOON_NO_REGEX` 关闭。

### JSON

| 函数 | 说明 |
|------|------|
| `json_encode` | 值 → JSON 字符串 |
| `json_decode` | JSON 字符串 → 值（list/dict/number/string/bool/null） |

可用 `--no-json` 或 `MOON_NO_JSON` 关闭。

### 异步与并发

| 类别 | 说明 |
|------|------|
| **异步** | `async(fn, ...args)` 在协程池中执行；`yield`；`wait_all`；`num_goroutines`、`num_cpu` |
| **Channel** | Go 风格：`chan()` 或 `chan(n)`（带缓冲）、`chan_send`、`chan_recv`、`chan_close`、`chan_is_closed` |
| **定时器** | `set_timeout(callback, ms)`、`set_interval(callback, ms)`、`clear_timer(id)` |
| **同步** | `mutex()`、`lock`、`unlock`、`trylock`、`mutex_free`；**原子操作**：`atomic_counter(initial)`、`atomic_add`、`atomic_get`、`atomic_set`、`atomic_cas` |

由 `MOON_HAS_ASYNC` 控制。`--target=mcu` 下不包含。

### DLL 与 FFI

| 类别 | 说明 |
|------|------|
| **DLL** | `dll_load(path)`、`dll_close`、`dll_func(handle, name)`；调用：`dll_call_int`、`dll_call_double`、`dll_call_str`、`dll_call_void`；`alloc_str`/`free_str` 用于 C 字符串；`ptr_to_str`、`read_ptr`、`read_int32`、`write_ptr`、`write_int32` |
| **FFI** | 在源码中声明 C 函数签名，直接调用 C 并传递/返回值。 |

由 `MOON_HAS_DLL` / `MOON_HAS_FFI` 控制。`--target=mcu` 下不包含。

### 系统与杂项

| 类别 | 说明 |
|------|------|
| **进程** | `argv`、`env`、`set_env`、`exit`、`shell`、`shell_output`、`platform`、`getpid`、`system`、`exec` |
| **格式化** | `format(fmt, ...)`（类 sprintf） |
| **内存 (MCU)** | `mem_stats`、`mem_reset`、`target_info` |
| **GC** | 引用计数 + 环检测；`gc_collect`、`gc_enable`、`gc_set_threshold`、`gc_stats` |

### 硬件 (HAL，嵌入式)

在带 HAL 的平台上（如 ESP32、树莓派 Pico、STM32）：

| 类别 | 说明 |
|------|------|
| **GPIO** | `gpio_init(pin, mode)`、`gpio_write`、`gpio_read`、`gpio_deinit`；模式：INPUT、OUTPUT、INPUT_PULLUP、INPUT_PULLDOWN |
| **PWM** | `pwm_init(pin, freq)`、`pwm_write(pin, duty)`、`pwm_deinit` |
| **ADC** | `adc_init`、`adc_read`、`adc_deinit` |
| **I2C** | `i2c_init(sda, scl, freq)`、`i2c_write(addr, data)`、`i2c_read(addr, length)`、`i2c_deinit` |
| **SPI** | `spi_init(mosi, miso, sck, freq)`、`spi_transfer(data)`、`spi_deinit` |
| **UART** | `uart_init(tx, rx, baud)`、`uart_write`、`uart_read`、`uart_available`、`uart_deinit` |
| **时间** | `delay_ms`、`delay_us`、`millis`、`micros` |
| **系统** | `hal_init_runtime`、`hal_deinit_runtime`、`hal_platform_name`、`hal_debug_print` |

由 `MOON_HAS_HAL` 控制（可选）。

### 构建时功能开关

| 选项 / 目标 | 效果 |
|-------------|------|
| 默认 | 完整：GUI、网络、TLS、正则、JSON、文件系统、浮点、DLL、异步 |
| `--target=embedded` | 无 GUI；保留网络等 |
| `--target=mcu` | 无 GUI、网络、DLL、异步、JSON；可选 `MOON_NO_FLOAT` |
| `--no-gui` | 不包含 GUI |
| `--no-network` | 不包含 TCP/UDP/TLS |
| `--no-regex` | 不包含正则（可省 PCRE2） |
| `--no-json` | 不包含 JSON |
| `MOON_NO_FILESYSTEM` / `MOON_NO_FFI` 等 | 在 C++ 构建中关闭对应模块 |

---

## 示例

以下均为可运行演示：保存为 `.moon` 后执行 `moonc demo.moon`，再运行生成的可执行文件（如 `demo.exe` 或 `./demo`）。

### 1. Hello world

```moonscript
print("Hello, MoonLang!")
# 输出: Hello, MoonLang!
```

### 2. 变量、循环与列表

```moonscript
numbers = [10, 20, 30]
for n in numbers {
    print(n * 2)
}
# 输出: 20, 40, 60
```

### 3. 函数与字符串

```moonscript
function greet(name) {
    return "Hello, " + name + "!"
}
print(greet("World"))
# 输出: Hello, World!
```

### 4. 文件与 JSON

读取 JSON 文件、取字段、写结果文件：

```moonscript
content = read_file("config.json")
if content != null {
    data = json_decode(content)
    print(data["version"])   # 例如 "1.0"
}
write_file("out.txt", "done")
```

运行前先创建 `config.json`，内容如 `{"version": "1.0"}`。

### 5. TCP 客户端

通过 TCP 发简单 HTTP GET：

```moonscript
sock = tcp_connect("127.0.0.1", 8080)
if sock != null and sock != -1 {
    tcp_send(sock, "GET / HTTP/1.0\r\n\r\n")
    reply = tcp_recv(sock)
    print(reply)
    tcp_close(sock)
} else {
    print("Connection failed")
}
```

需有服务在 8080 端口监听，或自行修改主机/端口。

### 6. 正则

匹配与替换：

```moonscript
s = "foo 123 bar 456"
m = regex_match(s, "\\d+")
print(m)   # 123
out = regex_replace_all(s, "\\d+", "X")
print(out) # foo X bar X
```

### 7. 异步与 Channel

Go 风格 channel：一个协程发送，主流程接收：

```moonscript
ch = chan(0)
async(() => {
    chan_send(ch, "hello from async")
})
msg = chan_recv(ch)
print(msg)   # hello from async
chan_close(ch)
```

### 8. 类（面向对象）

```moonscript
class Counter {
    function init() { self.n = 0 }
    function inc() { self.n += 1; return self.n }
}
c = new Counter()
print(c.inc())  # 1
print(c.inc())  # 2
```

### 9. 导出供 DLL/SO 使用

在构建共享库时使用（`moonc mylib.moon --shared -o mylib.dll`）：

```moonscript
export function add(a, b) { return a + b }
export function greet(name) { return "Hello, " + name }
```

---

## 语法简介

### 数据类型与变量

```moonscript
# 整数、浮点、字符串、布尔、空值
x = 42
pi = 3.14159
name = "MoonLang"
flag = true
nothing = null

# 十六进制 0x、二进制 0b、八进制 0o
hex = 0xFF
bin = 0b1010

# 列表与字典
numbers = [1, 2, 3]
person = {"name": "Alice", "age": 25}

# 多重赋值
x = y = z = 10
```

函数内使用全局变量需用 `global` 声明，否则赋值为局部变量。

### 双语法风格与控制流

同一文件内二选一，不可混用：

- **传统语法**：`:` 开始，`end` 结束  
- **大括号语法**：`{` `}` 包裹代码块  

```moonscript
# if / elif / else
if x > 10 { print("大") } elif x > 5 { print("中") } else { print("小") }

# while
while i < 10 { print(i); i += 1 }

# for-in / for-to
for item in [1, 2, 3] { print(item) }
for i = 1 to 10 { print(i) }

# switch
switch n { case 1: print("One") case 2, 3: print("Two or Three") default: print("Other") }

# break / continue
for i = 1 to 100 { if i > 10 { break } }
```

### 函数与 Lambda

```moonscript
# 函数定义（传统 / 大括号均可）
function add(a, b) { return a + b }
function greet(name, greeting="Hello") { return greeting + ", " + name }

# Lambda、闭包、默认参数
square = (x) => x * x
x = 10
add_x = (y) => x + y
```

### 类与面向对象

```moonscript
class Animal {
    function init(name, species) {
        self.name = name
        self.species = species
    }
    function speak() { print(self.name, "says something") }
}

class Dog extends Animal {
    function init(name) { super.init(name, "Canine") }
    function speak() { print(self.name, "says Woof!") }
}

dog = new Dog("Max")
dog.speak()
```

### 模块与注释

```moonscript
# 导入模块
import "stdlib/io.moon" as io
from "stdlib/net.moon" import TcpClient, TcpServer

# 单行注释 #
# 多行注释 """
```

---

## 部署与使用 moonc

构建本仓库得到 `moonc.exe`（或 Linux/macOS 下的 `moonc`）后，用于将 .moon 源码编译为原生程序或库。

### 编译为独立可执行文件

```batch
moonc hello.moon
moonc hello.moon -o myapp.exe
moonc hello.moon --icon app.ico
```

- 自动打包 `import` 的模块、`read_file()` 引用的文件；GUI 使用 `gui_load_url("file:///...")` 时会将 HTML 等资源嵌入。
- 常用选项：`-o` 输出名、`--icon` 图标、`--company` / `--copyright` / `--description` / `--file-version` / `--product-name` / `--product-version` 版本信息。

### 编译目标与选项

| 目标 / 选项 | 说明 |
|-------------|------|
| 默认 | 完整功能，桌面 EXE/ELF |
| `--target=embedded` | 嵌入式 Linux，无 GUI |
| `--target=mcu` | MCU 最小化（无 GUI/网络/DLL） |
| `--no-gui` / `--no-network` / `--no-regex` / `--no-json` | 按需关闭功能 |

```batch
moonc app.moon
moonc app.moon --target=embedded
moonc app.moon --no-gui --no-regex
```

### 导出 DLL 与 SO（共享库）

可将 MoonLang 模块编译为**共享库**（Windows 上为 DLL，Linux 上为 `.so`，macOS 上为 `.dylib`），供 C、C++、Python 等语言调用。

**1. 在源码中导出函数**

只有用 `export` 标记的函数才会暴露：

```moonscript
export function add(a, b) { return a + b }
export function greet(name) { return "Hello, " + name }
```

**2. 编译出共享库**

| 平台   | 命令（输出） |
|--------|----------------|
| Windows | `moonc mylib.moon --shared -o mylib.dll` |
| Linux   | `moonc mylib.moon --shared -o mylib.so`  |
| macOS   | `moonc mylib.moon --shared -o mylib.dylib` |

可选：生成 C 头文件，声明导出符号：

```batch
moonc mylib.moon --shared -o mylib.dll --header mylib.h
```

**3. 在 C 中调用**

生成的头文件会声明运行时初始化函数和你的导出函数。示例：

```c
#include "mylib.h"

int main(void) {
    moon_runtime_init(0, NULL);
    MoonValue* r = mylib_add(moon_int(2), moon_int(3));  // 5
    moon_release(r);
    moon_runtime_cleanup();
    return 0;
}
```

将你的程序与 `mylib.dll`（或 `mylib.so` / `mylib.dylib`）以及 MoonLang 运行时一起链接即可。DLL/SO 内包含编译后的代码，使用同一套运行时 ABI。

**4. 选项小结**

| 选项 | 说明 |
|------|------|
| `--shared` | 生成共享库而非可执行文件 |
| `-o <文件>` | 输出路径（如 `mylib.dll`、`mylib.so`、`libfoo.dylib`） |
| `--header <文件>` | 生成 C 头文件，声明导出函数 |

---

## 如何编译本仓库

### Windows (MSVC)

#### 1. 安装环境

- **Visual Studio 2022**  
  - 下载：[Visual Studio 2022](https://visualstudio.microsoft.com/zh-hans/downloads/)  
  - 安装时勾选工作负载 **“使用 C++ 的桌面开发”**（Desktop development with C++）。  
  - 脚本会使用：`C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat`；若使用 Professional/Enterprise，路径中的 `Community` 需改为对应版本。

- **LLVM（预编译包）**  
  - 下载：[LLVM 官方发布页](https://github.com/llvm/llvm-project/releases)，选择 Windows 的 MSVC 版本，例如：  
    `clang+llvm-21.1.8-x86_64-pc-windows-msvc`（或更新版本）。  
  - 解压到任意目录，例如：`C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc`。  
  - 记下解压后的**完整路径**，下一步会用到。

#### 2. 配置 LLVM 路径

- 打开 **`rebuild_all.bat`**，找到约第 29 行：  
  `set LLVM_DIR=C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc`  
  将等号右边改为你解压 LLVM 的路径（不要带末尾反斜杠）。

- 打开 **`llvm_libs.rsp`**，文件中每一行都是 LLVM 的 `.lib` 路径，形如：  
  `"C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc\lib\LLVMXXX.lib"`  
  将前缀 `C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc` 全部替换为你的 LLVM 解压路径。

**可选 — 自动检测 LLVM 并编译：**  
在仓库根目录执行 `python build_auto.py`。脚本会检测 LLVM（环境变量 `LLVM_PATH`/`LLVM_DIR` 或常见安装目录），生成 `llvm_libs.rsp` 和 **`rebuild_auto.bat`**（已写入正确路径），然后执行构建。加 `--no-run` 仅生成文件不编译；加 `--llvm C:\path\to\llvm` 可指定 LLVM 路径。

#### 3. 执行构建

- 在资源管理器中进入本仓库**根目录**（与 `rebuild_all.bat` 同级）。  
- 双击运行 **`rebuild_all.bat`**，或在“适用于 VS 的 x64 本机工具命令提示”中执行：  
  `rebuild_all.bat`  

- 脚本会依次：  
  - 检测并设置 MSVC 环境  
  - 若有 `lib\pcre2`、`lib\openssl` 则启用 PCRE2/OpenSSL  
  - 根据 `version.json` 生成 `llvm\version.h`  
  - 编译运行时（含可选 PCRE2/OpenSSL）生成 `llvm\moonrt.lib`  
  - 编译编译器并链接 LLVM，生成 **`moonc.exe`**（在仓库根目录或脚本输出目录）  
  - 若存在 **`pack_dist.bat`**，会打包到 **`dist\moonscript\`**

#### 4. 可选依赖（按需准备）

| 依赖 | 用途 | 说明 |
|------|------|------|
| **PCRE2** | 正则引擎 | 将 PCRE2 源码放到 `lib\pcre2\` 并按脚本要求构建，或运行 `lib\pcre2\download_pcre2.bat`（若有）。无则使用内置 std::regex。 |
| **OpenSSL** | TLS/HTTPS | 将 OpenSSL 的库与头文件放到 `lib\openssl\`（如 `lib`、`include`），详见 `lib\openssl\README.md`（若有）。无则 TLS 相关功能不可用。 |
| **WebView2** | Windows GUI | 将 WebView2 相关文件放到 `webview2\`。无则 GUI 可能降级或不可用。 |

---

### Linux

#### 1. 安装依赖

在仓库根目录下使用 **`build/build_linux.sh`** 前，需安装：

- **仅构建运行时库**（不生成 `moonc`）：  
  `sudo apt install build-essential cmake`  
  （Ubuntu/Debian；其他发行版使用对应包管理器安装 g++、cmake。）

- **构建编译器**（生成 `moonc`）：  
  还需安装 LLVM 开发包：  
  `sudo apt install llvm-dev`  
  （或 `libllvm-dev`，视发行版而定。）

- **带 GUI 的版本**（需 WebKitGTK）：  
  `sudo apt install libgtk-3-dev libwebkit2gtk-4.0-dev`  
  若包名为 `libwebkit2gtk-4.1-dev` 也可，脚本会检测。  
  可选：`libappindicator3-dev`（系统托盘）。

一键安装示例（Ubuntu/Debian）：

```bash
# 仅运行时 + 编译器（无 GUI）
sudo apt install build-essential cmake llvm-dev

# 运行时 + 编译器 + GUI
sudo apt install build-essential cmake llvm-dev pkg-config libgtk-3-dev libwebkit2gtk-4.0-dev
```

#### 2. 执行构建

在仓库根目录执行：

```bash
cd build
chmod +x build_linux.sh
./build_linux.sh --help    # 查看所有选项
```

常用命令：

| 命令 | 说明 |
|------|------|
| `./build_linux.sh` | 仅构建运行时库（不生成 moonc） |
| `./build_linux.sh --compiler` | 构建运行时 + 编译器（需已装 llvm-dev），生成可执行文件 `moonc` |
| `./build_linux.sh --compiler --no-gui` | 构建编译器且不依赖 GUI（适合无桌面或服务器） |
| `./build_linux.sh --clean` | 清理构建目录 |
| `./build_linux.sh --install` | 安装到系统路径（具体路径见脚本输出） |

构建产物在 `build/linux_build/` 下，`moonc` 可执行文件位于该目录或脚本输出的路径。

---

### macOS

#### 1. 安装环境

- **Xcode Command Line Tools**（必选）：  
  ```bash
  xcode-select --install
  ```
  弹窗中按提示安装。

- **Homebrew**（推荐，用于安装 CMake、LLVM、OpenSSL）：  
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
  Apple Silicon 为 `/opt/homebrew`，Intel 为 `/usr/local`。

- **CMake、LLVM、OpenSSL**：  
  ```bash
  brew install cmake llvm openssl@3
  ```

- **将 LLVM 加入 PATH**（否则可能找不到 `llvm-config`）：  
  - Apple Silicon：  
    `export PATH="/opt/homebrew/opt/llvm/bin:$PATH"`  
  - Intel：  
    `export PATH="/usr/local/opt/llvm/bin:$PATH"`  
  可将上述命令写入 `~/.zshrc` 或 `~/.bash_profile`。

#### 2. 执行构建

在仓库根目录执行：

```bash
cd build
chmod +x build_macos.sh
./build_macos.sh --help    # 查看所有选项
```

常用命令：

| 命令 | 说明 |
|------|------|
| `./build_macos.sh` | 仅构建运行时库 |
| `./build_macos.sh --compiler` | 构建运行时 + 编译器（需 LLVM 在 PATH 中） |
| `./build_macos.sh --compiler --no-gui` | 构建编译器且不依赖 GUI |
| `./build_macos.sh --no-tls` | 不链接 OpenSSL，禁用 TLS 支持 |
| `./build_macos.sh --clean` | 清理构建目录 |
| `./build_macos.sh --install` | 安装到系统路径 |

构建产物在 `build/macos_build/` 下。若未安装 OpenSSL 或加 `--no-tls`，TLS 相关功能将被关闭。

---

## 构建脚本说明

| 文件 | 用途 |
|------|------|
| `rebuild_all.bat` | Windows 完整重建 |
| `link_llvm.cmd` / `llvm_libs.rsp` | 链接 moonc.exe |
| `pack_dist.bat` | 打开发布包 |
| `build/build_linux.sh` | Linux 构建（`--compiler`、`--no-gui`、`--clean`、`--install`） |
| `build/build_macos.sh` | macOS 构建（`--compiler`、`--no-gui`、`--no-tls`、`--clean`、`--install`） |

---

## 目录结构概要

| 路径 | 说明 |
|------|------|
| `lexer.*` `parser.*` `ast.h` `token.h` `alias_loader.*` | 编译器前端 |
| `llvm/` | LLVM 代码生成与运行时 |
| `lib/` | 第三方库（pcre2、openssl、db） |
| `webview2/` | Windows GUI 依赖 |
| `build/` | Linux/macOS 构建 |

---

## 许可证与版权

本仓库采用 **GPLv3** 发布。详见 [LICENSE](LICENSE)。

- **开源使用**：依 GPLv3 复制、修改与分发；分发修改版须同样以 GPLv3 开放源码。
- **闭源/商用修改**：若修改本编译器并以闭源形式发布或商用，须向版权方取得**商业授权并支付授权费**。联系：**forkp@greenteng.com**。

Copyright (c) 2026 moon-lang.com
