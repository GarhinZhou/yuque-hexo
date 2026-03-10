---
title: RUST PWN
date: '2025-12-04 12:03:17'
updated: '2025-12-19 17:12:39'
---
感觉 RUST PWN 跟 C 语言没太大差别，因为主流的 rust 程序用的也是 glibc 库，主要其实是 IDA 的伪代码看起来很难看，关键还是要动调和从 binary 的字符串下手

# 强网杯 S9 2025 somebox
跟着 WP 看：[https://xia0ji233.pro/2025/11/26/qwb2025_final/index.html](https://xia0ji233.pro/2025/11/26/qwb2025_final/index.html)

这题好综合...一运行程序就不对味了，一看是一堆 hex，应该是把正常输出加密了，

考验逆向和密码这我真不会...真正用到 pwn 的实际上是限制条件 shellcode 

前面的交互脚本看 WP 就已经写了两百行...

```python
# 输入：已有的若干项（无符号 64 位，Python int 足够）
from math import gcd
M = 1 << 64
def mod64(x):
    return x & (M - 1)


# 解线性同余 k*dx ≡ dy (mod M)
# 返回候选 k 的列表（每个在 [0, M-1]）
def solve_for_k(dx, dy):
    dx = mod64(dx);
    dy = mod64(dy)
    if dx == 0:
        # 0 * k ≡ dy (mod M)  -> 有解当且仅当 dy ≡ 0 (mod M)
        return [0] if dy == 0 else []  # 特殊：dx==0 不约束 k，如果 dy==0 则任意 k 可，返回 placeholder
    g = gcd(dx, M)
    if dy % g != 0:
        return []
    # 除以 g，在模 M' 下求逆
    dxp = dx // g
    dyp = dy // g
    Mp = M // g
    # 计算 (dxp)^(-1) mod Mp
    # 使用扩展欧几里得或 pow since Mp 不是素数但 gcd(dxp, Mp)==1
    inv = pow(dxp, -1, Mp)  # Python 3.8+
    k0 = (dyp * inv) % Mp
    # 所有解为 k0 + t*Mp, t=0..g-1
    return [(k0 + t * Mp) % M for t in range(g)]


def find_k_b_from_sequence(seq):
    n = len(seq)
    if n < 2:
        return None  # 信息太少（任意 k,b 可）
    # 尝试用第一个能约束的相邻三项（或多处）来生成候选 k
    candidates = None
    for i in range(n - 2):
        dx = (seq[i + 1] - seq[i]) % M
        dy = (seq[i + 2] - seq[i + 1]) % M
        ks = solve_for_k(dx, dy)
        if ks == []:
            return []  # 在此处无解 -> 整个序列不可能来自同一线性映射
        # 若 dx==0 且 dy==0 意味着这三项对 k 不产生约束，继续找下一组
        if dx == 0 and dy == 0:
            continue
        # 否则 ks 为候选集合（注意：若 dx==0 and dy==0 we skipped）
        candidates = ks
        break

    # 如果一直没有约束（全部相等或每组三项都 dx==0,dy==0），处理特殊情形
    if candidates is None:
        # 所有相邻差都为0 => seq 都相同 => (k-1)*c + b ≡ 0 (mod M),无限多解
        return "ALL_EQUAL"  # 提示无限多解（或需要更多约束）

    # 验证候选，保留能让所有已知项成立的那些
    valid = []
    for k in candidates:
        b = (seq[1] - (k * seq[0])) % M
        ok = True
        for i in range(n - 1):
            if ((k * seq[i] + b) - seq[i + 1]) % M != 0:
                ok = False;
                break
        if ok:
            valid.append((k, b))
    return valid
```

```python
#!/usr/bin/env python3

from pwn import *
from m import find_k_b_from_sequence
context.log_level = 'debug'
context.arch = 'amd64'
import os
import sys
from pwn import *

def ror64(value, shift, bits=64):
    """64位循环右移 - rol64的逆操作"""
    shift %= bits
    return ((value >> shift) | (value << (bits - shift))) & ((1 << bits) - 1)


def rol64(value, shift, bits=64):
    """64位循环左移"""
    shift %= bits
    return ((value << shift) | (value >> (bits - shift))) & ((1 << bits) - 1)


def mix(v6):
    v7 = v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2) ^ ((v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2)) >> 4) ^ (
                (v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2) ^ ((v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2)) >> 4)) >> 8) ^ (
                     (v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2) ^ ((v6 ^ (v6 >> 1) ^ ((v6 ^ (v6 >> 1)) >> 2)) >> 4) ^ ((
                                                                                                                                  v6 ^ (
                                                                                                                                      v6 >> 1) ^ (
                                                                                                                                              (
                                                                                                                                                          v6 ^ (
                                                                                                                                                              v6 >> 1)) >> 2) ^ (
                                                                                                                                              (
                                                                                                                                                          v6 ^ (
                                                                                                                                                              v6 >> 1) ^ (
                                                                                                                                                                      (
                                                                                                                                                                                  v6 ^ (
                                                                                                                                                                                      v6 >> 1)) >> 2)) >> 4)) >> 8)) >> 16)
    v7 = v7 & 0xFFFFFFFFFFFFFFFF
    v7h = v7 >> 32
    v8 = rol64(v7 ^ v7h, 7)
    return v8


def inverse_xor_shift(y, k):
    """逆异或变换：从 y 恢复 x，满足 y = x ^ (x >> k)"""
    x = y
    shift = k
    while shift < 64:
        x = x ^ (x >> shift)
        shift *= 2
    return x & 0xFFFFFFFFFFFFFFFF


def inverse_mix(v8):
    """mix 函数的逆向算法"""
    # 步骤1: 循环右移7位得到 u
    u = ror64(v8, 7)

    # 步骤2: 从 u 恢复 v7
    u_high = (u >> 32) & 0xFFFFFFFF
    u_low = u & 0xFFFFFFFF
    v7_high = u_high
    v7_low = u_low ^ u_high
    v7 = (v7_high << 32) | v7_low

    # 步骤3-7: 逐步逆异或变换恢复 v6
    x4 = inverse_xor_shift(v7, 16)
    x3 = inverse_xor_shift(x4, 8)
    x2 = inverse_xor_shift(x3, 4)
    x1 = inverse_xor_shift(x2, 2)
    x0 = inverse_xor_shift(x1, 1)

    return x0


OOO = bytes.fromhex('''
3D 3D 3D 3D 3D 3D 3D 3D 3D 3D 3D 3D 3D 3D 3D 3D
3D 20 54 4F 59 20 45 4E 43 52 59 50 54 45 44 20
53 45 52 56 49 43 45 20 3D 3D 3D 3D 3D 3D 3D 3D
3D 3D 3D 3D 3D 3D 3D 3D 3D 0A 31 29 20 45 63 68
6F 20 62 61 63 6B 20 77 68 61 74 20 79 6F 75 20
73 65 6E 64 0A 32 29 20 41 64 6D 69 6E 20 6C 6F
67 69 6E 0A 33 29 20 4D 61 67 69 63 43 6F 64 65
20 72 75 6E 6E 65 72 0A 34 29 20 53 68 6F 77 20
63 6F 6E 66 69 67 0A 35 29 20 51 75 69 74 0A 0A
2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D
2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D
2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D 2D
2D 2D 2D 2D 2D 2D 2D 2D 2D 0A 59 6F 75 72 20 63
68 6F 69 63 65 3A 20
''')




def setKey(hexdata):
    data = bytes.fromhex(hexdata.decode())
    global KEYA, KEYB, KEYC, KEYX
    kc = [0, 0, 0, 0]
    kb = [0, 0, 0, 0]
    ka = [0, 0, 0, 0]
    for i in range(0xC0 // 8):
        if (i % 4 == 0):
            print()
        num1 = u64(data[i * 8:i * 8 + 8])
        num2 = u64(OOO[i * 8:i * 8 + 8])
        KEYX.append(num1 ^ num2)
        KEYC_TMP.append(inverse_mix(num1 ^ num2))
        # print(hex(KEYX[i]),hex(KEYC[i]))

    print("----------")
    for i in range(4, 8):
        num1 = u64(data[i * 8:i * 8 + 8])
        num2 = u64(OOO[i * 8:i * 8 + 8])
        if i == 5 or i == 7:
            KEYB[i & 3] = KEYC_TMP[i]
            # print(hex(KEYB[i & 3]))
    k1 = [KEYC_TMP[0], KEYC_TMP[4], KEYC_TMP[8], KEYC_TMP[12]]
    k2 = [KEYC_TMP[1], KEYC_TMP[5], KEYC_TMP[9], KEYC_TMP[13]]
    k3 = [KEYC_TMP[2], KEYC_TMP[6], KEYC_TMP[10], KEYC_TMP[14]]
    k4 = [KEYC_TMP[3], KEYC_TMP[7], KEYC_TMP[11], KEYC_TMP[15]]
    res1 = find_k_b_from_sequence(k1)
    res2 = find_k_b_from_sequence(k2)
    res3 = find_k_b_from_sequence(k3)
    res4 = find_k_b_from_sequence(k4)
    print(res1)
    print(res2)
    print(res3)
    print(res4)
    if len(res1) == 1 and len(res2) == 1 and len(res3) == 1 and len(res4) == 1 or 1==1:
        KEYA[0] = res1[0][0]
        KEYA[1] = res2[0][0]
        KEYA[2] = res3[0][0]
        KEYA[3] = res4[0][0]
        KEYB[0] = res1[0][1]
        KEYB[1] = res2[0][1]
        KEYB[2] = res3[0][1]
        KEYB[3] = res4[0][1]
        print("KEYA:", [hex(i) for i in KEYA])
        print("KEYB:", [hex(i) for i in KEYB])


def getKeyStream():
    global KEYA, KEYB, KEYC, KEYX
    KEYX = [i for i in KEYC]
    i = 0
    while (1):
        realkey = mix(KEYX[i & 3])
        yield realkey.to_bytes(8, 'little')
        KEYX[i & 3] = (KEYB[i & 3] + KEYA[i & 3] * KEYX[i & 3]) & 0xFFFFFFFFFFFFFFFF
        i += 1


KEYGEN = getKeyStream()


def decrypt(hexdata):
    KEY = b''
    data = bytes.fromhex(hexdata.decode())
    dec = bytearray(data)
    for i in range(len(data)):
        if i % 8 == 0:
            KEY = next(KEYGEN)
        dec[i] ^= KEY[i % len(KEY)]
    # print(dec.hex())
    return dec


def encrypt(data):
    KEY = b''
    enc = bytearray(data)
    for i in range(len(data)):
        if i % 8 == 0:
            KEY = next(KEYGEN)
        enc[i] ^= KEY[i % len(KEY)]
    # print(dec)
    return enc.hex().replace(" ", "")

for i in range(1):
    KEYA = [0, 0, 0, 0]
    KEYB = [0, 0, 0, 0]
    KEYC_TMP = []  # [0x9E3779B97F4A7C15, 0x0000000000000000, 0x94D049BB133111EB, 0x0000000000000000]
    KEYC = [0x9E3779B97F4A7C15, 0x0000000000000000, 0x94D049BB133111EB, 0x0000000000000000]
    KEYX = []

    r = process("./somebox")
    firstData = r.recvline()
    setKey(firstData)
    if (KEYA[1] == 0):
        r.close()
        sys.exit(0)
    decrypt(firstData)
    while True:
        inp = input('$')
        if inp == 'exit':
            break
        r.sendline(encrypt(inp.encode()))
        b = r.recvline()
        print(decrypt(b).decode())
```

