---
title: WASM
date: '2025-02-05 10:35:39'
updated: '2025-04-07 21:33:51'
---
# 特点
## 一些指令
`get 数字` （数字对应变量的索引）入栈

`set 数字` 出栈

## 函数表
在 WebAssembly 中，函数表是一个存储模块内所有函数的表格，它主要由函数指针构成。每个函数都有一个唯一的索引，通过该索引可以访问对应的函数。

+ 函数指针：函数表中的每个条目都是函数指针，指向了实际的函数实现。
+ 索引：每个函数在 WebAssembly 中都会被分配一个索引，这个索引用于标识函数，而函数表的条目通过索引来访问。

## 实现
WebAssembly 的函数表通常是通过一个数组或列表来实现的，数组中的每个元素存储的是一个指向 WebAssembly 函数的指针。函数表的大小通常是动态变化的，随着函数的导入、导出和动态添加。

+ 表的结构：函数表在 WebAssembly 中是一个名为 `table` 的对象，它通常由多个函数指针组成。每个 WebAssembly 模块在构建时，会为其定义一个或多个函数表。
+ 类型的统一：函数表中的所有函数必须是相同类型的。这意味着，函数表中的每个函数都必须遵循相同的签名（即相同的参数和返回类型）。WebAssembly 强制所有的函数表只能存放相同类型的函数指针。

## 函数表的操作
WebAssembly 中对函数表的操作主要包括以下几个方面：

+ 创建函数表：函数表可以通过 `table` 指令创建，并定义其大小和类型。可以定义一个初始大小，也可以定义一个最大大小，表示表的容量范围。

![](/images/ef114eec049cd317bd9df37169943fb5.jpeg)

+ 添加/删除函数：在 WebAssembly 模块的执行过程中，可以向函数表中添加新的函数（例如，动态加载的函数）。但是，函数表的大小是有限制的，因此添加函数时需要保证表的容量不会超出最大限制。
+ 访问函数表中的函数：每个函数在函数表中都有一个对应的索引，模块通过索引来调用特定的函数。访问函数表通常是通过 `call_indirect` 指令进行的，它允许通过索引间接调用函数。
+ 修改函数表：通过对函数表的动态修改，可以在运行时替换函数的实现。这样可以支持例如回调函数、动态链接等高级特性。

# 用 Ghidra 鸡爪（？）分析
## 安装 Ghidra
[Github链接](https://github.com/NationalSecurityAgency/ghidra)

Ghidra 是由美国国家安全局研究局创建和维护的软件逆向工程 (SRE) 框架 。该框架包括一套功能齐全的高端软件分析工具，使用户能够在包括 Windows、macOS 和 Linux 在内的各种平台上分析编译代码。功能包括反汇编、汇编、反编译、绘图和脚本，以及数百个其他功能。Ghidra 支持多种处理器指令集和可执行格式，并且可以在用户交互和自动化模式下运行。用户还可以使用 Java 或 Python 开发自己的 Ghidra 扩展组件和/或脚本。

### 过程
先安装好 JDK (Java Developer Kit)，看环境配置篇

然后运行下载下来之后的压缩包当中的 ghidra 文件（Linux 系统无后缀，Windows 系统后缀 bat），就可以启动 ghidra

## 安装 WABT（WebAssembly Binary Toolkit）
WABT 发音“wabbit”

[Github链接](https://github.com/WebAssembly/wabt)

提供了关于 WebAssembly 的许多转换工具

下载下来的压缩包解压后，打开环境变量，在系统变量的 Path 当中加上 bin 文件夹的路径，确认三次保存

### 把 wat 文件转换成 wasm 文件
```bash
# parse test.wat and write to .wasm binary file with the same name
$ wat2wasm test.wat

# parse test.wat and write to binary file test.wasm
$ wat2wasm test.wat -o test.wasm

# parse spec-test.wast, and write verbose output to stdout (including the
# meaning of every byte)
$ wat2wasm spec-test.wast -v
```

## 安装 Ghidra 的 wasm 拓展
[Github链接](https://github.com/nneonneo/ghidra-wasm-plugin)

打开 ghidra - "File" - "Install Extensions"，点右上角的绿色加号，选好下载的 wasm 拓展 zip 文件，确认后重启 ghidra

新建项目，导入 wasm 文件 language 栏有 wasm 表示拓展安装成功，进到 CodeBrowser 分析之后就可以看到伪代码了

# 用 IDA 分析
用 `WABT`把 `.wat` 文件或者 `.wasm` 文件转成`.c` 文件，然后用`gcc`将`.c`文件编译成`.o`目标文件，再丢进 IDA 分析

注意，只有`.c`文件是不足以编译成`ELF`文件的，因为原来的`.wasm`文件当中缺少了函数的实现，所以还得把wabt项目内的`wasm-rt.h`，`wasm-rt-impl.h`两个文件复制到`.c` 文件的目录下再进行编译

而且，`WABT` 在将`.wat` 文件或者 `.wasm` 文件转成`.c` 文件时可能会用上`<alloca.h>`头文件，`<alloca.h>` 是一个用于动态分配栈内存的头文件，这个头文件在某些操作系统中是可用的，尤其是 UNIX 系统（如 Linux）但它并不是标准 C 库的一部分，因此在一些平台上可能找不到。可以在 Linux 平台用 gcc 编译成目标文件再丢进 IDA 分析，就可以解决

# 符号表
在`.wat` 文件编译成`.wasm` 文件时丢失了符号表，要静态分析程序的话需要对照`.wat` 文件里面的符号对伪代码进行还原



