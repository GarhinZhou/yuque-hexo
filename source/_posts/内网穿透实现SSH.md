---
title: 内网穿透实现SSH
date: '2025-11-11 15:37:57'
updated: '2025-12-27 08:56:29'
---
之前暑假买的阿里云服务器有用了...

之前试过用那些远程桌面软件连但是发现要么就是要VIP，要么就是直连没法连（应该是校园网的问题）

查了一下可以用公网服务器来当跳板，来连接内网的服务，从而实现随时随地连接上校园网内网的服务器

## FRP
这个工具挺方便的，只需要把他部署在内网服务器和公网服务器就可以了，一个文件下下来里面包含了frpc和frps两个binary，是用rust写的

### 公网服务器配置
最好不要用公网服务器直接下载软件，通过ftp上传之后再安装

```plain
tar -xf frp_0.54.0_linux_amd64.tar.gz
mv frp_0.54.0_linux_amd64 frp
cd frp
```

在frps目录编辑配置文件：

```plain
vim frps.toml
```

```plain
bindPort = 

auth.method = "token"
auth.token = ""

log.to = "/home/garhin/logs/frps.log"
log.level = "info"
log.maxDays = 3
#加了log之后直接运行frp就不会有回显了
```

token是为了让内网服务器和公网服务器的链接更可信

然后添加系统服务，让他自己在后台启动和运行：

systemd 是一个用于 Linux 操作系统的系统和服务管理器

对应的系统服务在enable之后就会在开机的时候自动启动

系统服务文件在目录`/etc/systemd/system/`以.service为后缀名

```plain
vim /etc/systemd/system/frps.service
```

配置frps的服务文件

```plain
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为frps的安装路径
ExecStart = /home/garhin/frp/frps -c /home/garhin/frp/frps.toml

Restart = on-failure
RestartSec = 3s

LimitNOFILE = 65536
MemoryLimit = 512M

[Install]
WantedBy = multi-user.target
```

然后再启动就可以了：

```plain
sudo systemctl daemon-reload
sudo systemctl enable frps
```

### 内网服务器配置
```plain
tar -xf frp_0.54.0_linux_amd64.tar.gz
mv frp_0.54.0_linux_amd64 frp
cd frp
vim frpc.toml
```

```plain
serverAddr = ""
serverPort = 

auth.method = "token"
auth.token = ""

log.to = "/home/garhin/frp/logs/frpc.log"
log.level = "info"
log.maxDays = 3

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort =
```

```plain
vim /etc/systemd/system/frpc.service
```

别搞混了frpc是client内网服务器，frps是server公网服务器

```plain
[Unit]
# 服务名称，可自定义
Description = frp client
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为frps的安装路径
ExecStart = /home/garhin/frp/frpc -c /home/garhin/frp/frpc.toml

Restart = on-failure
RestartSec = 3s

[Install]
WantedBy = multi-user.target
```

```plain
sudo systemctl daemon-reload
sudo systemctl enable frpc
```

### 配置细节
其中bindport和serverport是内网服务器和公网服务器的通信端口

而内网服务器配置里面的remoteport就是外部通过公网服务器连接内网服务器的端口

记得在服务器控制台把这两个端口的出和入都打开

之后就可以通过SSH随时随地连接内网服务器了

```plain
ssh -o port=remoteport user@server_ip
```

## 搭配食用--termux
termux是安卓系统的一个命令行软件，用的pkg包管理器

换源很简单：

```plain
pkg set-mirror tuna
```

可以安装tmux方便使用

```plain
pkg install tmux
```

### termux+远程tmux实现远程pwn脚本动调
其中因为gdb.attach(link)，会唤起新的bash，正常来说ssh里面直接跑会报错

```plain
[+] Starting local process './malloc' argv=[b'./malloc'] : pid 48172
[*] '/home/garhin/CTF/malloc/libc.so.6'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
[*] running in new terminal: ['/usr/bin/gdb', '-q', './malloc', '48172']
[ERROR] Could not find a terminal binary to use. Set context.terminal to your terminal.
Traceback (most recent call last):
  File "/home/garhin/CTF/malloc/./malloc.py", line 10, in <module>
    gdb.attach(link)
  File "/usr/lib/python3/dist-packages/pwnlib/context/__init__.py", line 1581, in setter
    return function(*a, **kw)
           ^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3/dist-packages/pwnlib/gdb.py", line 1100, in attach
    gdb_pid = misc.run_in_new_terminal(cmd, preexec_fn = preexec_fn)
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3/dist-packages/pwnlib/util/misc.py", line 362, in run_in_new_terminal
    log.error('Could not find a terminal binary to use. Set context.terminal to your terminal.')
  File "/usr/lib/python3/dist-packages/pwnlib/log.py", line 439, in error
    raise PwnlibException(message % args)
pwnlib.exception.PwnlibException: Could not find a terminal binary to use. Set context.terminal to your terminal.
[*] Stopped process './malloc' (pid 48172)
```

所以要先开tmux（没想到ubuntu24自带tmux诶），再在里面跑脚本

## tmux使用
tmux的操作都要先用快捷键ctrl+b启动，再按对应键实现功能

因为ssh当中没有鼠标数据，只能用快捷键来实现窗口焦点的转换

ctrl+b o：切换同一个session的不同pane

ctrl+b space：切换水平分割和垂直分割（上下分屏和左右分屏）

ctrl+b ctrl+o：互换pane位置

