---
title: Pwntools使用
date: '2024-12-10 18:46:34'
updated: '2025-09-20 17:23:50'
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

#### Big Endian（大端字节序）:
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

#### Little Endian（小端字节序）:
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

### 数据发送形式
`str(system_addr).encode()` 和`p64(system_addr)` 都用于将数据转换为字节字符串，但它们的用途和结果不同。

`str(system_addr).encode()`

+ 将整数 system_addr 转换为字符串，然后将该字符串编码为字节字符串。
+ 适用于需要将整数以字符串形式发送的场景。

```python
system_addr = 0x7ffff7a33440
encoded_str = str(system_addr).encode()
print(encoded_str)  # 输出: b'140737351819584'
```

`p64(system_addr)`

+ 将整数 system_addr 转换为64位小端字节序的字节字符串。
+ 适用于需要将整数以二进制形式发送的场景，常用于内存地址和二进制协议。

```python
from pwn import p64

system_addr = 0x7ffff7a33440
binary_str = p64(system_addr)
print(binary_str)  # 输出: b'@\x34\xa3\xf7\xff\x7f\x00\x00'
```

总结

+ `str(system_addr).encode()` 适用于将整数以字符串形式发送
+ `p64(system_addr)` 适用于将整数以二进制形式发送，常用于内存地址和二进制协议

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

## 0x08 ROP 模块
之前一直没用过这个，这段时间刷ctfshow看见一些exp是用这个的，感觉挺方便的欸

一些比较简单的ROP就不用自己找gadgets了，这样看感觉自己之前写的exp好low（？

而且还不用纠结32位和64位的参数传递！不过要记得在脚本设置context

但是总觉得这种操作不确定性太强了，没有自己找gadgets那么踏实（shellcraft也是这种感觉，感觉没有自己写踏实），不过有这么方便的方式当然还是方便点好（懒）

官方文档：https://docs.pwntools.com/en/stable/rop/rop.html#pwnlib.rop.rop.ROP

```python
rop=ROP("./binary")

#按ROP顺序写，例子：
rop.raw(b'a' * offsets)
rop.read(0,0x8049804 + 4,4)
rop.read(0,0x80498e0,len(dynstr))
rop.read(0,0x80498e0 + 0x100,len("/bin/sh\x00"))
rop.raw(0x8048376)
rop.raw(0xdeadbeef)

#控制寄存器
rop.eax = 0xdeadbeef

#设置函数调用参数
rop.target(1,2,3)

#打印出已构建ROP链
print(rop.dump())
```

它甚至可以筛选掉一些含有程序不能输入或者输入会被截断的字节的gadgets...它真的，我哭死...

```python
rop = ROP(elf, badchars=b'\x02')
#badchars就是不能输入程序的字节，含有这些字节的gadgets会被筛掉
```

最后发送ROP链用`rop.chain()`

注意：一条ROP链就得用一次`rop=ROP("./binary")`

## 0x09 fmtstr_payload
栈上格式化字符串

参数：

```plain
Parameters
offset (int) – 通过格式符遇到字符串的最小偏移（可以用%p-%p-%p爆破出来偏移）
writes (dict) – 词典类型{键:值}，即{要修改的地址:修改值}
numbwritten (int) – printf已经输出的字符数，用来计算%c相关的数
write_size (str) – 每次修改长度byte, short or int(hhn, hn or n)
overflows (int) – how many extra overflows (at size sz) to tolerate to reduce the length of the format string
strategy (str) – either ‘fast’ or ‘small’ (‘small’ is default, ‘fast’ can be used if there are many writes)
no_dollars (bool) – 值（True/False）是否使用$来指定偏移，不用的话其实就是把%12312c这个拆出来指定数量的%c来凑到指定偏移
```

例如修改printf函数的plt改成system的：

```python
fmtstr_payload(offset,{printf_got:system_plt})
```

题目：

```c
while ( 1 )
{
    memset(buf, 0, sizeof(buf));
    read(0, buf, 0x64u);
    printf(buf);
}
```

这时候就可以直接把printf改成system，再次输入就是system指令了

## 0x0a 一些函数
### get_section_by_name()
感觉pwntools有好多功能没用上啊，

这个`get_section_by_name()`可以用来获取对应名称的节

ret2dlresolve中可以用来直接获取`.dynstr`节然后替换对应的符号字符串来伪造`.dynstr`

```python
dynstr =elf.get_section_by_name('.dynstr').data()
dynstr = dynstr.replace(b'read',b'system')
```

### Ret2dlresolvePayload()
没想到pwntools有这个，可以直接生成ret2dlresolve的payload

```python
rop = ROP(elf)
dlresolve = Ret2dlresolvePayload(elf, symbol="system", args=["/bin/sh"])
rop.read(0, dlresolve.data_addr)
rop.ret2dlresolve(dlresolve)
raw_rop = rop.chain()

link.recvuntil('Welcome to CTFshowPWN!\n')
link.sendline(flat({112:raw_rop, 256:dlresolve.payload}))
```

