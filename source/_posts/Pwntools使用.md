---
title: Pwntools使用
date: '2024-12-10 18:46:34'
updated: '2025-05-11 11:57:46'
---
## <font style="background-color:#FBDE28;">重点！！！</font>
python 脚本不能以 pwn.py 为名称，不然会因为`from pwn import *`

导致 pwntools 损坏！！！

打比赛打着打着 pwntools 崩了，真的太恐怖了...

## 0x01 配置上下文
`from pwn import *`引入pwntools库

`context(arch='架构', os='系统', endian='字节序', word_size=字大小)`配置环境

`context.log_level = 'debug'`配置后将会在屏幕打印所有发送和接收的数据

常用环境配置代码：

```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote("",)
# link=process("./")
libc=ELF('./libc.so.6')
elf=ELF('./')
```

### 字节序（来自GPT）
在计算机架构中，`endian`（字节序）指的是多字节数据在内存中存储的顺序方式。具体来说，`endian` 影响的是数据的字节如何在内存中排列，尤其是对于需要多个字节来表示的数值类型（如 `int` 或 `float`）而言。有两种常见的字节序：

#### 1. Big Endian（大端字节序）:
+ 在大端字节序中，高字节（Most Significant Byte，MSB）存储在内存的低地址处，低字节（Least Significant Byte，LSB）存储在高地址处。
+ 举个例子，如果我们有一个 32 位整数 `0x12345678`，在内存中的存储顺序会是：

```plain
地址       值
0x00    0x12
0x01    0x34
0x02    0x56
0x03    0x78
```

高字节 `0x12` 存储在最小地址 `0x00`，低字节 `0x78` 存储在最大地址 `0x03`。

#### 2. Little Endian（小端字节序）:
+ 在小端字节序中，低字节存储在内存的低地址处，高字节存储在高地址处。
+ 对于同样的整数 `0x12345678`，在小端字节序的内存中存储顺序如下：

```plain
地址       值
0x00    0x78
0x01    0x56
0x02    0x34
0x03    0x12
```

低字节 `0x78` 存储在最低地址 `0x00`，高字节 `0x12` 存储在最大地址 `0x03`。

不同的硬件架构可能使用不同的字节序。比如：

+ **Intel x86 和 x86_64** 系列处理器通常使用小端字节序（Little Endian）。
+ **某些 PowerPC 架构和 ARM 的某些模式** 使用大端字节序（Big Endian），但 ARM 也可以支持小端字节序（取决于配置）。

在多字节数据的传输、存储和网络通信时，字节序不一致可能导致数据解释错误。因此，了解字节序非常重要，尤其在进行跨平台编程时。

#### 在 ARM 上的字节序设置
ARM 处理器支持 **双字节序**，即它可以在 **大端模式（Big Endian）** 或 **小端模式（Little Endian）** 之间切换，具体取决于系统配置或应用需求。



## 0x02 监听进程
`link=remote("IP",端口)`

`link=process("程序路径")`

`link.interactive()`直接通过终端与进程交互



## 0x03 pack 模块
`p64()`、`p32()`、`p16()`、`p8() `打包函数

反之解包函数：`u64()`、`u32()`、`u16()`、`u8()`



## 0x04 asm 模块
`asm('汇编指令')`汇编转机器码

`disasm('机器码')`机器码转汇编指令



## 0x05 SigreturnFrame 模块
通过下面的语句生成 SigreturnFrame 对象：

```python
frame=SigreturnFrame()
```

然后可以直接对里面的成员赋值：

```python
frame.rdi=...
```

![](/images/a3f790f6e8f8951bd5d0b1742a8f7724.png)

注意在输入栈的时候要以字节流输入：

```python
payload=bytes(frame)
```



## 0x06 Shellcraft 模块
包含一些生成shellcode的函数，其中的子模块声明架构，比如`shellcraft.arm` 是ARM架构的，`shellcraft.amd64`是AMD64架构，`shellcraft.i386`是Intel 80386架构的，以及有一个`shellcraft.common`是所有架构通用的

常用的，也是最简单的shellcode，即`调用/bin/sh`可以通过shellcraft得到：

```python
print(shellcraft.sh()) # 打印出shellcode
print(asm(shellcraft.sh())) # 打印出汇编后的shellcode
```

注意：由于各个平台，特别是32位和64位的shellcode不一样，所以最好先设置context



## 0x07 gdb 模块
```python
gdb.attach(link,'gdb指令')
```

