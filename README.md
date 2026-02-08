# MoonLang Compiler (C++ Source)

**English** | **[中文](README.zh-CN.md)** · **Official site: [moon-lang.com](https://moon-lang.com)**

---

MoonLang is a lightweight static programming language implemented in C++ with LLVM. It supports dual syntax styles (`: end` and `{ }`), and runs on Windows, Linux, macOS, and embedded platforms (ARM/RISC-V/ESP32). Compiled binaries are 15KB–300KB, suitable for MCU to desktop applications.

This repository contains **compiler source only** (frontend + LLVM backend + runtime). It does not include the standard library `.moon` files, examples, or executables.

---

## Contents

- [Supported features](#supported-features)
- [Examples](#examples)
- [Syntax overview](#syntax-overview)
  - [Data types and variables](#data-types-and-variables)
  - [Dual syntax and control flow](#dual-syntax-and-control-flow)
  - [Functions and Lambda](#functions-and-lambda)
  - [Classes and OOP](#classes-and-oop)
  - [Modules and comments](#modules-and-comments)
- [Deploying and using moonc](#deploying-and-using-moonc)
  - [Building a standalone executable](#building-a-standalone-executable)
  - [Build targets and options](#build-targets-and-options)
  - [Exporting DLL and SO (shared libraries)](#exporting-dll-and-so-shared-libraries)
- [Building this repository](#building-this-repository)
  - [Windows (MSVC)](#windows-msvc)
  - [Linux](#linux)
  - [macOS](#macos)
- [Build scripts](#build-scripts)
- [Directory layout](#directory-layout)
- [License and copyright](#license-and-copyright)

---

## Supported features

The runtime provides a full set of built-in capabilities. Modules can be disabled at build time via `MOON_NO_XXX` or at compile time via `moonc --no-xxx` for smaller binaries.

### Language & types

| Area | Description |
|------|-------------|
| **Types** | `int`, `float`, `string`, `bool`, `null`, `list`, `dict`, functions, classes, closures, **BigInt** (arbitrary precision) |
| **Literals** | Hex `0x`, binary `0b`, octal `0o`; list/dict literals; multi-line strings |
| **Operators** | Arithmetic, comparison, logical, bitwise (`& \| ^ ~ << >>`), power `^`, string concat |
| **Control flow** | `if/elif/else`, `while`, `for-in`, `for-to`, `switch`, `break`, `continue` |
| **Functions** | Named functions, lambdas, default args, closures, variadic-style via list |
| **OOP** | Classes, inheritance, `super`, `self`, constructors `init` |
| **Modules** | `import "path" as name`, `from "path" import a, b` |
| **Errors** | Try/catch/finally, `throw`, exception value |

### I/O & filesystem

| Function / area | Description |
|-----------------|-------------|
| **Console** | `print`, `input` |
| **Files** | `read_file`, `write_file`, `append_file` (binary-safe) |
| **Paths** | `exists`, `is_file`, `is_dir`, `list_dir`, `create_dir`, `file_size`, `getcwd`, `cd` |
| **Path utils** | `join_path`, `basename`, `dirname`, `extension`, `absolute_path`, `copy_file`, `move_file`, `remove_file`, `remove_dir` |

### Date & time

| Area | Description |
|------|-------------|
| **Time** | `time`, `now` (ms), `unix_time`, `sleep` |
| **Format/parse** | `date_format`, `date_parse` with format string and optional timezone |
| **Components** | `year`, `month`, `day`, `hour`, `minute`, `second`, `millisecond`, `weekday`, `day_of_year`, `week_of_year`, `quarter` |
| **Arithmetic** | `make_time`, `add_seconds/minutes/hours/days/months/years`, `diff_seconds`, `diff_days` |
| **Ranges** | `start_of_day`, `end_of_day`, `start_of_month`, `end_of_month` |
| **Helpers** | `days_in_month`, `is_leap_year`, `is_weekend`, `is_today`, `is_same_day`, `timezone`, `utc_offset`, `set_timezone` |

### Math & numbers

| Area | Description |
|------|-------------|
| **Basic** | `abs`, `min`, `max`, `pow`, `sqrt`, `floor`, `ceil`, `round`, `sign`, `clamp`, `lerp` |
| **Trig** | `sin`, `cos`, `tan`, `asin`, `acos`, `atan`, `atan2`, `degrees`, `radians` |
| **Log/exp** | `log`, `log10`, `log2`, `exp`, `sinh`, `cosh`, `tanh`, `hypot` |
| **Stats** | `mean`, `median` (on list) |
| **Random** | `random_int`, `random_float` |

### Strings & bytes

| Area | Description |
|------|-------------|
| **String ops** | `len`, substring, `split`, `join`, `replace`, `trim`, `upper`, `lower`, `capitalize`, `title`, `contains`, `starts_with`, `ends_with`, `index_of`, `find`, `repeat`, `chr`, `ord` |
| **Predicates** | `is_alpha`, `is_digit`, `is_alnum`, `is_space`, `is_lower`, `is_upper` |
| **Padding** | `pad_left`, `pad_right`, `ltrim`, `rtrim` |
| **Binary** | `bytes_to_string` (byte list → string), strings are binary-safe (length stored in header) |
| **WebSocket** | `ws_parse_frame`, `ws_create_frame` (low-level frame parse/create) |

### Lists & dicts

| Area | Description |
|------|-------------|
| **List** | Index, set, `append`, `insert`, `pop`, `remove`, `len`, `slice`, `contains`, `index_of`, `reverse`, `sort`, `sum`, `first`, `last`, `take`, `drop`, `shuffle`, `choice`, `unique`, `flatten`, `zip`, `count` |
| **Functional** | `list_map`, `list_filter`, `list_reduce` (with callback) |
| **Dict** | `get`, `set`, `has_key`, `keys`, `values`, `items`, `delete`, `merge` |
| **Range** | `range(n)` or `range(start, end)` or `range(start, end, step)` |

### Network

| Area | Description |
|------|-------------|
| **TCP** | `tcp_connect`, `tcp_listen`, `tcp_accept`, `tcp_send`, `tcp_recv`, `tcp_close`; non-blocking: `tcp_set_nonblocking`, `tcp_has_data`, `tcp_select`, `tcp_accept_nonblocking`, `tcp_recv_nonblocking` |
| **UDP** | `udp_socket`, `udp_bind`, `udp_send`, `udp_recv`, `udp_close` |
| **TLS (OpenSSL)** | `tls_connect`, `tls_listen`, `tls_accept`, `tls_send`, `tls_recv`, `tls_recv_all`, `tls_close`; verify/hostname, cert/key/CA load, `tls_wrap_client` / `tls_wrap_server` |

Controlled by `MOON_HAS_NETWORK`; TLS by `MOON_HAS_TLS` (optional OpenSSL). On Windows: IOCP/WSAPoll; on Linux: epoll.

### GUI

| Area | Description |
|------|-------------|
| **Basic** | `gui_init`, `gui_create`, `gui_show`, `gui_set_title`, `gui_set_size`, `gui_set_position`, `gui_close`, `gui_run`, `gui_quit`, `gui_alert`, `gui_confirm` |
| **Advanced** | `gui_create_advanced` (options: frameless, transparent, topmost, resizable, etc.), **WebView**: `gui_load_url`, `gui_load_html`, `gui_on_message` (JS ↔ MoonLang bridge) |
| **Tray** | `gui_tray_create`, `gui_tray_remove`, `gui_tray_set_menu`, `gui_tray_on_click`, `gui_show_window` |

- **Windows**: WebView2 (embedded Chromium).
- **Linux**: WebKitGTK.
- **macOS**: WKWebView.

Controlled by `MOON_HAS_GUI`. Disabled in `--target=embedded` and `--target=mcu`.

### Regular expressions

| Area | Description |
|------|-------------|
| **Match** | `regex_match`, `regex_search`, `regex_test`, `regex_groups`, `regex_named`, `regex_find_all`, `regex_find_all_groups` |
| **Replace** | `regex_replace`, `regex_replace_all` |
| **Split** | `regex_split`, `regex_split_n` |
| **Compiled** | `regex_compile`, then `regex_match_compiled`, `regex_search_compiled`, `regex_find_all_compiled`, `regex_replace_compiled`, `regex_free` |
| **Utils** | `regex_escape`, `regex_error` |

Uses PCRE2. Disable with `--no-regex` or `MOON_NO_REGEX`.

### JSON

| Function | Description |
|----------|-------------|
| `json_encode` | Value → JSON string |
| `json_decode` | JSON string → value (list/dict/number/string/bool/null) |

Disable with `--no-json` or `MOON_NO_JSON`.

### Async & concurrency

| Area | Description |
|------|-------------|
| **Async** | `async(fn, ...args)` — run on coroutine pool; `yield`; `wait_all`; `num_goroutines`, `num_cpu` |
| **Channels** | Go-style: `chan()` or `chan(n)` (buffered), `chan_send`, `chan_recv`, `chan_close`, `chan_is_closed` |
| **Timers** | `set_timeout(callback, ms)`, `set_interval(callback, ms)`, `clear_timer(id)` |
| **Sync** | `mutex()`, `lock`, `unlock`, `trylock`, `mutex_free`; **Atomics**: `atomic_counter(initial)`, `atomic_add`, `atomic_get`, `atomic_set`, `atomic_cas` |

Controlled by `MOON_HAS_ASYNC`. Disabled in `--target=mcu`.

### DLL & FFI

| Area | Description |
|------|-------------|
| **DLL** | `dll_load(path)`, `dll_close`, `dll_func(handle, name)`; call: `dll_call_int`, `dll_call_double`, `dll_call_str`, `dll_call_void`; `alloc_str` / `free_str` for C strings; `ptr_to_str`, `read_ptr`, `read_int32`, `write_ptr`, `write_int32` |
| **FFI** | Declare C signatures in source; call C functions and pass/return values. |

Controlled by `MOON_HAS_DLL` / `MOON_HAS_FFI`. Disabled in `--target=mcu`.

### System & misc

| Area | Description |
|------|-------------|
| **Process** | `argv`, `env`, `set_env`, `exit`, `shell`, `shell_output`, `platform`, `getpid`, `system`, `exec` |
| **Format** | `format(fmt, ...)` (sprintf-style) |
| **Memory (MCU)** | `mem_stats`, `mem_reset`, `target_info` |
| **GC** | Reference counting + cycle collection; `gc_collect`, `gc_enable`, `gc_set_threshold`, `gc_stats` |

### Hardware (HAL, embedded)

When built with HAL (e.g. ESP32, Raspberry Pi Pico, STM32):

| Area | Description |
|------|-------------|
| **GPIO** | `gpio_init(pin, mode)`, `gpio_write`, `gpio_read`, `gpio_deinit`; modes: INPUT, OUTPUT, INPUT_PULLUP, INPUT_PULLDOWN |
| **PWM** | `pwm_init(pin, freq)`, `pwm_write(pin, duty)`, `pwm_deinit` |
| **ADC** | `adc_init`, `adc_read`, `adc_deinit` |
| **I2C** | `i2c_init(sda, scl, freq)`, `i2c_write(addr, data)`, `i2c_read(addr, length)`, `i2c_deinit` |
| **SPI** | `spi_init(mosi, miso, sck, freq)`, `spi_transfer(data)`, `spi_deinit` |
| **UART** | `uart_init(tx, rx, baud)`, `uart_write`, `uart_read`, `uart_available`, `uart_deinit` |
| **Time** | `delay_ms`, `delay_us`, `millis`, `micros` |
| **System** | `hal_init_runtime`, `hal_deinit_runtime`, `hal_platform_name`, `hal_debug_print` |

Controlled by `MOON_HAS_HAL` (optional).

### Build-time feature flags

| Flag / target | Effect |
|---------------|--------|
| Default | Full: GUI, network, TLS, regex, JSON, filesystem, float, DLL, async |
| `--target=embedded` | No GUI; network and rest available |
| `--target=mcu` | No GUI, network, DLL, async, JSON; optional `MOON_NO_FLOAT` |
| `--no-gui` | No GUI module |
| `--no-network` | No TCP/UDP/TLS |
| `--no-regex` | No regex (saves PCRE2) |
| `--no-json` | No JSON |
| `MOON_NO_FILESYSTEM` / `MOON_NO_FFI` / etc. | Disable individual modules in C++ build |

---

## Examples

The snippets below are runnable: save as `.moon`, then `moonc demo.moon` and run the generated executable (e.g. `demo.exe` or `./demo`).

### 1. Hello world

```moonscript
print("Hello, MoonLang!")
# Output: Hello, MoonLang!
```

### 2. Variables, loop, and list

```moonscript
numbers = [10, 20, 30]
for n in numbers {
    print(n * 2)
}
# Output: 20, 40, 60
```

### 3. Function and string

```moonscript
function greet(name) {
    return "Hello, " + name + "!"
}
print(greet("World"))
# Output: Hello, World!
```

### 4. File and JSON

Read a JSON file, read a field, write a result file:

```moonscript
content = read_file("config.json")
if content != null {
    data = json_decode(content)
    print(data["version"])   # e.g. "1.0"
}
write_file("out.txt", "done")
```

Create `config.json` with `{"version": "1.0"}` before running.

### 5. TCP client

Simple HTTP GET over TCP:

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

Run with a server listening on port 8080, or change host/port.

### 6. Regex

Match and replace:

```moonscript
s = "foo 123 bar 456"
m = regex_match(s, "\\d+")
print(m)   # 123
out = regex_replace_all(s, "\\d+", "X")
print(out) # foo X bar X
```

### 7. Async and channel

Go-style channel: one coroutine sends, main receives:

```moonscript
ch = chan(0)
async(() => {
    chan_send(ch, "hello from async")
})
msg = chan_recv(ch)
print(msg)   # hello from async
chan_close(ch)
```

### 8. Class (OOP)

```moonscript
class Counter {
    function init() { self.n = 0 }
    function inc() { self.n += 1; return self.n }
}
c = new Counter()
print(c.inc())  # 1
print(c.inc())  # 2
```

### 9. Export for DLL/SO

Used when building a shared library (`moonc mylib.moon --shared -o mylib.dll`):

```moonscript
export function add(a, b) { return a + b }
export function greet(name) { return "Hello, " + name }
```

---

## Syntax overview

### Data types and variables

```moonscript
# Integer, float, string, bool, null
x = 42
pi = 3.14159
name = "MoonLang"
flag = true
nothing = null

# Hex 0x, binary 0b, octal 0o
hex = 0xFF
bin = 0b1010

# List and dict
numbers = [1, 2, 3]
person = {"name": "Alice", "age": 25}

# Multiple assignment
x = y = z = 10
```

Use `global` to assign to a global variable inside a function; otherwise the assignment creates a local variable.

### Dual syntax and control flow

Use one style per file (do not mix):

- **Traditional**: `:` to start, `end` to close  
- **Brace syntax**: `{` `}` around the block  

```moonscript
# if / elif / else
if x > 10 { print("big") } elif x > 5 { print("mid") } else { print("small") }

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

### Functions and Lambda

```moonscript
# Function (traditional or brace)
function add(a, b) { return a + b }
function greet(name, greeting="Hello") { return greeting + ", " + name }

# Lambda, closure, default args
square = (x) => x * x
x = 10
add_x = (y) => x + y
```

### Classes and OOP

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

### Modules and comments

```moonscript
# Import
import "stdlib/io.moon" as io
from "stdlib/net.moon" import TcpClient, TcpServer

# Single-line # and multi-line """
```

---

## Deploying and using moonc

After building this repo you get `moonc.exe` (or `moonc` on Linux/macOS), used to compile `.moon` source into native executables or libraries.

### Building a standalone executable

```batch
moonc hello.moon
moonc hello.moon -o myapp.exe
moonc hello.moon --icon app.ico
```

- Automatically bundles `import`ed modules and files referenced by `read_file()`; for GUI, resources referenced by `gui_load_url("file:///...")` are embedded.
- Common options: `-o` output name, `--icon`, `--company`, `--copyright`, `--description`, `--file-version`, `--product-name`, `--product-version`.

### Build targets and options

| Target / option | Description |
|-----------------|-------------|
| Default | Full desktop EXE/ELF |
| `--target=embedded` | Embedded Linux, no GUI |
| `--target=mcu` | MCU minimal (no GUI/network/DLL) |
| `--no-gui` / `--no-network` / `--no-regex` / `--no-json` | Disable features as needed |

```batch
moonc app.moon
moonc app.moon --target=embedded
moonc app.moon --no-gui --no-regex
```

### Exporting DLL and SO (shared libraries)

You can compile a MoonLang module into a **shared library** (DLL on Windows, `.so` on Linux, `.dylib` on macOS) and call its exported functions from C, C++, Python, or other languages.

**1. Export functions in source**

Only functions marked with `export` are exposed:

```moonscript
export function add(a, b) { return a + b }
export function greet(name) { return "Hello, " + name }
```

**2. Build the shared library**

| Platform | Command (output) |
|----------|------------------|
| Windows | `moonc mylib.moon --shared -o mylib.dll` |
| Linux   | `moonc mylib.moon --shared -o mylib.so`  |
| macOS   | `moonc mylib.moon --shared -o mylib.dylib` |

Optional: generate a C header for the exported symbols:

```batch
moonc mylib.moon --shared -o mylib.dll --header mylib.h
```

**3. Use from C**

The generated header declares the runtime init and your exports. Example usage:

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

Link your program against `mylib.dll` (or `mylib.so` / `mylib.dylib`) and the MoonLang runtime. The DLL/SO embeds the compiled code and uses the same runtime ABI.

**4. Summary**

| Option | Description |
|--------|-------------|
| `--shared` | Build a shared library instead of an executable |
| `-o <file>` | Output path (e.g. `mylib.dll`, `mylib.so`, `libfoo.dylib`) |
| `--header <file>` | Emit a C header with declarations for exported functions |

---

## Building this repository

### Windows (MSVC)

#### 1. Install environment

- **Visual Studio 2022**
  - Download: [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/).
  - During setup, select the **“Desktop development with C++”** workload.
  - The script uses `C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat`. If you use Professional or Enterprise, change `Community` in that path accordingly.

- **LLVM (prebuilt)**
  - Download from [LLVM releases](https://github.com/llvm/llvm-project/releases) — choose the Windows MSVC build (e.g. `clang+llvm-21.1.8-x86_64-pc-windows-msvc` or newer).
  - Extract to a folder such as `C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc`.
  - Note the **full path**; you will set it in the next step.

#### 2. Set LLVM path

- Open **`rebuild_all.bat`** and find the line (around line 29):  
  `set LLVM_DIR=C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc`  
  Set the right-hand side to your LLVM path (no trailing backslash).

- Open **`llvm_libs.rsp`**. Every line is a path to an LLVM `.lib` file. Replace the prefix  
  `C:\LLVM-dev\clang+llvm-21.1.8-x86_64-pc-windows-msvc`  
  with your LLVM path in **all** lines.

**Optional — auto-detect LLVM and build:**  
Run `python build_auto.py` in the repo root. It detects LLVM (from `LLVM_PATH`/`LLVM_DIR`, or common install locations), writes `llvm_libs.rsp` and **`rebuild_auto.bat`** (with the correct path), then runs the build. Use `--no-run` to only generate files; use `--llvm C:\path\to\llvm` to force a path.

#### 3. Build

- Go to the repository **root** (where `rebuild_all.bat` lives).
- Double-click **`rebuild_all.bat`** or run it from a “x64 Native Tools Command Prompt for VS”:
  ```
  rebuild_all.bat
  ```
- The script will:
  - Set up the MSVC environment
  - Use PCRE2/OpenSSL if present under `lib\pcre2` and `lib\openssl`
  - Generate `llvm\version.h` from `version.json`
  - Build the runtime and produce `llvm\moonrt.lib`
  - Build and link the compiler to produce **`moonc.exe`** (in the repo root or the script’s output directory)
  - If **`pack_dist.bat`** exists, run it to populate **`dist\moonscript\`**

#### 4. Optional dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| **PCRE2** | Regex engine | Place PCRE2 source in `lib\pcre2\` and build as required, or run `lib\pcre2\download_pcre2.bat` if present. Without it, the build uses std::regex. |
| **OpenSSL** | TLS/HTTPS | Place OpenSSL libraries and headers in `lib\openssl\` (see `lib\openssl\README.md` if present). Without it, TLS is disabled. |
| **WebView2** | Windows GUI | Place WebView2 files in `webview2\`. Without it, GUI may be limited or unavailable. |

---

### Linux

#### 1. Install dependencies

From the repo root, use **`build/build_linux.sh`**. Install:

- **Runtime only** (no `moonc`):  
  `sudo apt install build-essential cmake`  
  (Ubuntu/Debian; other distros: install g++, cmake as appropriate.)

- **Compiler** (to get `moonc`):  
  Also install LLVM:  
  `sudo apt install llvm-dev`  
  (or `libllvm-dev`, depending on distro.)

- **With GUI** (WebKitGTK):  
  `sudo apt install libgtk-3-dev libwebkit2gtk-4.0-dev`  
  (or `libwebkit2gtk-4.1-dev`; the script detects either.)  
  Optional: `libappindicator3-dev` for system tray.

Example (Ubuntu/Debian):

```bash
# Runtime + compiler, no GUI
sudo apt install build-essential cmake llvm-dev

# Runtime + compiler + GUI
sudo apt install build-essential cmake llvm-dev pkg-config libgtk-3-dev libwebkit2gtk-4.0-dev
```

#### 2. Build

From the repo root:

```bash
cd build
chmod +x build_linux.sh
./build_linux.sh --help    # list options
```

Common commands:

| Command | Description |
|---------|-------------|
| `./build_linux.sh` | Build runtime only (no moonc) |
| `./build_linux.sh --compiler` | Build runtime + compiler (requires llvm-dev); produces `moonc` |
| `./build_linux.sh --compiler --no-gui` | Build compiler without GUI (e.g. for servers) |
| `./build_linux.sh --clean` | Clean build directory |
| `./build_linux.sh --install` | Install to system (path shown by script) |

Output is under `build/linux_build/`; the `moonc` binary is in that directory or the path printed by the script.

---

### macOS

#### 1. Install environment

- **Xcode Command Line Tools** (required):
  ```bash
  xcode-select --install
  ```
  Follow the prompt to install.

- **Homebrew** (recommended, for CMake, LLVM, OpenSSL):
  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```
  Apple Silicon: `/opt/homebrew`; Intel: `/usr/local`.

- **CMake, LLVM, OpenSSL**:
  ```bash
  brew install cmake llvm openssl@3
  ```

- **Add LLVM to PATH** (so `llvm-config` is found):
  - Apple Silicon:  
    `export PATH="/opt/homebrew/opt/llvm/bin:$PATH"`
  - Intel:  
    `export PATH="/usr/local/opt/llvm/bin:$PATH"`
  Add the appropriate line to `~/.zshrc` or `~/.bash_profile` to make it permanent.

#### 2. Build

From the repo root:

```bash
cd build
chmod +x build_macos.sh
./build_macos.sh --help    # list options
```

Common commands:

| Command | Description |
|---------|-------------|
| `./build_macos.sh` | Build runtime only |
| `./build_macos.sh --compiler` | Build runtime + compiler (LLVM must be in PATH) |
| `./build_macos.sh --compiler --no-gui` | Build compiler without GUI |
| `./build_macos.sh --no-tls` | Do not link OpenSSL; TLS disabled |
| `./build_macos.sh --clean` | Clean build directory |
| `./build_macos.sh --install` | Install to system |

Output is under `build/macos_build/`. If OpenSSL is not installed or you use `--no-tls`, TLS support is disabled.

---

## Build scripts

| File | Purpose |
|------|---------|
| `rebuild_all.bat` | Windows full rebuild |
| `link_llvm.cmd` / `llvm_libs.rsp` | Link moonc.exe |
| `pack_dist.bat` | Package distribution |
| `build/build_linux.sh` | Linux build (`--compiler`, `--no-gui`, `--clean`, `--install`) |
| `build/build_macos.sh` | macOS build (`--compiler`, `--no-gui`, `--no-tls`, `--clean`, `--install`) |

---

## Directory layout

| Path | Description |
|------|-------------|
| `lexer.*` `parser.*` `ast.h` `token.h` `alias_loader.*` | Compiler frontend |
| `llvm/` | LLVM codegen and runtime |
| `lib/` | Third-party (pcre2, openssl, db) |
| `webview2/` | Windows GUI |
| `build/` | Linux/macOS build |

---

## License and copyright

This project is under **GNU General Public License v3.0 (GPLv3)**. See [LICENSE](LICENSE).

- **Open-source use**: You may copy, modify, and distribute under GPLv3; modified versions must be released under GPLv3 with source.
- **Proprietary / commercial use of modified compiler**: If you **modify this compiler and distribute or use it in closed-source or commercial form**, you must obtain a **commercial license and pay the applicable fee** from the copyright holder. Contact: **forkp@greenteng.com**.

Copyright (c) 2026 moon-lang.com
