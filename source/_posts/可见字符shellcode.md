---
title: 可见字符shellcode
date: '2025-03-09 13:40:00'
updated: '2025-04-07 21:39:55'
---
用可见字符即 ascii 码范围内的字节来组织 shellcode

## 工具
### ae64
AE64 is a tool which can transform any amd64 architecture shellcode into pure alphanumeric shellcode using self-modify code technology, so the page need to be writable.

安装

```shell
git clone https://github.com/veritas501/ae64.git --depth 1
cd ae64
sudo python3 setup.py install
```

用法（注意 pwntools 生成 shellcode 默认 32 位！！要加上 context）

```python
from ae64 import AE64
from pwn import *
context.arch='amd64'

# get bytes format shellcode
shellcode = asm(shellcraft.sh())

# get alphanumeric shellcode
enc_shellcode = AE64().encode(shellcode)
print(enc_shellcode.decode('latin-1'))
```

默认值（默认是 rax，要改成实际 call 的寄存器）

```python
enc_shellcode = AE64().encode(shellcode)
# equal to 
enc_shellcode = AE64().encode(shellcode,'rax', 0, 'fast')

'''
def encode(self, shellcode: bytes, register: str = 'rax', offset: int = 0, strategy: str = 'fast') -> bytes:
"""
encode given shellcode into alphanumeric shellcode (amd64 only)
@param shellcode: bytes format shellcode
@param register: the register contains shellcode pointer (can with offset) (default=rax)
@param offset: the offset (default=0)
@param strategy: encode strategy, can be "fast" or "small" (default=fast)
@return: encoded shellcode
"""
'''
```

## 指令一览
```plain
1.数据传送:
push/pop eax…
pusha/popa

2.算术运算:
inc/dec eax…
sub al, 立即数
sub byte ptr [eax… + 立即数], al dl…
sub byte ptr [eax… + 立即数], ah dh…
sub dword ptr [eax… + 立即数], esi edi
sub word ptr [eax… + 立即数], si di
sub al dl…, byte ptr [eax… + 立即数]
sub ah dh…, byte ptr [eax… + 立即数]
sub esi edi, dword ptr [eax… + 立即数]
sub si di, word ptr [eax… + 立即数]

3.逻辑运算:
and al, 立即数
and dword ptr [eax… + 立即数], esi edi
and word ptr [eax… + 立即数], si di
and ah dh…, byte ptr [ecx edx… + 立即数]
and esi edi, dword ptr [eax… + 立即数]
and si di, word ptr [eax… + 立即数]

xor al, 立即数
xor byte ptr [eax… + 立即数], al dl…
xor byte ptr [eax… + 立即数], ah dh…
xor dword ptr [eax… + 立即数], esi edi
xor word ptr [eax… + 立即数], si di
xor al dl…, byte ptr [eax… + 立即数]
xor ah dh…, byte ptr [eax… + 立即数]
xor esi edi, dword ptr [eax… + 立即数]
xor si di, word ptr [eax… + 立即数]

4.比较指令:
cmp al, 立即数
cmp byte ptr [eax… + 立即数], al dl…
cmp byte ptr [eax… + 立即数], ah dh…
cmp dword ptr [eax… + 立即数], esi edi
cmp word ptr [eax… + 立即数], si di
cmp al dl…, byte ptr [eax… + 立即数]
cmp ah dh…, byte ptr [eax… + 立即数]
cmp esi edi, dword ptr [eax… + 立即数]
cmp si di, word ptr [eax… + 立即数]

5.转移指令:
push 56h
pop eax
cmp al, 43h
jnz lable

<=> jmp lable

6.交换al, ah
push eax
xor ah, byte ptr [esp] // ah ^= al
xor byte ptr [esp], ah // al ^= ah
xor ah, byte ptr [esp] // ah ^= al
pop eax

7.清零:
push 44h
pop eax
sub al, 44h ; eax = 0

push esi
push esp
pop eax
xor [eax], esi ; esi = 0
```

