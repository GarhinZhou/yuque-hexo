---
title: 去码头整点linux
date: '2025-09-05 11:33:17'
updated: '2025-12-20 14:34:12'
---
好早之前就想装个实机的linux了，感觉挺方便的

之前电脑保修期没过还不敢这么搞，现在炸了也没事（

打算整个ubuntu24，之前都是用的22，不过都是自己用的话24应该也差不多，pwn题环境的话就patch出来吧

iso镜像就从官网下，PE U盘工具就用balena etcher

重启按F2进BIOS关掉fastboot、security boot然后改boot方式为UEFI USB

进去安装系统之后就跟着过去就行了

要注意的点：

1. 不联网安装变量最小，可以很好杜绝因为网络环境导致的安装失败

真想不明白为啥最新版ubuntu24系统安装只能用清华吞拿源，装到一半流量过多就被ban IP了...

1. 自定义分盘：1T

| 挂载点 | 大小 | 类型 | 备注 |
| --- | --- | --- | --- |
| / | 100GB | ext4 | 系统根目录 |
| /home | 615GB | ext4 | 用户空间 |
| 空 | 8G | swap | 休眠需要 |
| /boot/eft | 1.13GB | FAT32 | UEFI自动生成 |
| /usr | 300GB | ext4 | 默认安装目录 |


swap实际上是内存的交换空间，内存满了就会把不常使用的数据从内存放到swap里面

1. ubuntu24把源的路径换到了/etc/apt/sources.list.d/ubuntu.sources下（换源之后记得apt update看看换没换成功）
2. 联网安装的话安装前时区不能选错

ubuntu24 源的写法不太一样了：

```bash
Types: deb
Architectures: amd64 i386
URIs: http://mirrors.aliyun.com/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
Architectures: arm64
URIs: https://mirrors.aliyun.com/ubuntu-ports/
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

Types: deb
Architectures: arm64
URIs: https://mirrors.aliyun.com/ubuntu-ports/
Suites: noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

> 架构生命周期：Ubuntu 官方的态度
>
> Ubuntu 的 ubuntu-ports 并不是支持“所有”非 x86 架构，它只支持那些处于活跃维护期的架构
>
> 目前官方重点支持： arm64 (ARMv8), armhf, ppc64el, s390x 以及新晋的 riscv64
>
> MIPS 的处境： MIPS 架构在通用 Linux 发行版中的地位正在被 RISC-V 迅速取代
>
> Debian 已经将 MIPS 移出了核心支持列表，而基于 Debian 的 Ubuntu 步子迈得更快，直接在 24.04 的官方镜像中剔除了 MIPS
>

## 获取一些软件
因为我是用我的鸡哥游戏本装的ubuntu，所以比较要紧的其实是测试一下steam上的游戏能不能玩（不是

ubuntu24的新linux内核好像对N卡的兼容性挺好的，直接装系统的话显卡驱动也不用管

后续：然而并不是，自带的显卡驱动甚至只能发挥显卡半成功力...

软件的话直接下deb包然后在路径下`sudo apt install ./deb`就可以安装了

### IDA Pro 9.1
https://mrx.hk/posts/f66fa9e6643dec79c46f35361986b601/

没想到到了ubuntu这边资源怎么比windows的还要新，好像是之前一次性全破解了

两个用来破解的文件：

idapro.hexlic：

```json
{"header":{"version":1},"payload":{"email":"elv@ven","licenses":[{"add_ons":[{"code":"HEXX86","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-01","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXX64","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-02","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXARM","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-03","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXARM64","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-04","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXMIPS","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-05","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXMIPS64","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-06","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXPPC","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-07","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXPPC64","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-08","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXRV64","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-09","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXARC","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-10","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"},{"code":"HEXARC64","end_date":"2033-12-31 23:59:59","id":"48-1337-DEAD-11","owner":"48-2137-ACAB-99","start_date":"2024-08-10 00:00:00"}],"description":"license","edition_id":"ida-pro","end_date":"2033-12-31 23:59:59","features":[],"id":"48-2137-ACAB-99","issued_on":"2024-08-10 00:00:00","license_type":"named","owner":"","product":"IDA","product_id":"IDAPRO","product_version":"9.1","seats":1,"start_date":"2024-08-10 00:00:00"}],"name":"elf"},"signature":"E70382A9C9FCAAA36CB35DD93C0358FE4A68DBC511115633C7D05783D7FA95D45F7A688F12A91DAFEC2E9DE87BD7527686977306490BA39B45CA40FDD9E4785C730863F8C84573C1BE369DF9BB3DB95C7A7F5C39E8F2031B35BE2FC38C5A8361283871B82250BB04883D133F095DA3CB0A4A5E2DAF94A407597F095328091D0D"}
```

keygen.py：（里面的name和email可以改）

```python
import json
import hashlib
import os

# originally made by alula
# v3 - auth.lol
license = {
    "header": {"version": 1},
    "payload": {
        "name": "Garhin",
        "email": "garhinzhou@163.com",
        "licenses": [
            {
                "description": "license",
                "edition_id": "ida-pro",
                "id": "48-2137-ACAB-99",
                "license_type": "named",
                "product": "IDA",
                "seats": 1,
                "start_date": "2024-08-10 00:00:00",
                "end_date": "2033-12-31 23:59:59",  # This can't be more than 10 years!
                "issued_on": "2024-08-10 00:00:00",
                "owner": "",
                "product_id": "IDAPRO",
                "product_version": "9.1",
                "add_ons": [
                    # {
                    #     "id": "48-1337-DEAD-01",
                    #     "code": "HEXX86L",
                    #     "owner": "48-0000-0000-00",
                    #     "start_date": "2024-08-10 00:00:00",
                    #     "end_date": "2033-12-31 23:59:59",
                    # },
                    # {
                    #     "id": "48-1337-DEAD-02",
                    #     "code": "HEXX64L",
                    #     "owner": "48-0000-0000-00",
                    #     "start_date": "2024-08-10 00:00:00",
                    #     "end_date": "2033-12-31 23:59:59",
                    # },
                ],
                "features": [],
            }
        ],
    },
}

def add_every_addon(license):
    platforms = [
        "W",  # Windows
        "L",  # Linux
        "M",  # macOS
    ]
    addons = [
        "HEXX86",
        "HEXX64",
        "HEXARM",
        "HEXARM64",
        "HEXMIPS",
        "HEXMIPS64",
        "HEXPPC",
        "HEXPPC64",
        "HEXRV64",
        "HEXARC",
        "HEXARC64",
        # Probably cloud?
        # "HEXCX86",
        # "HEXCX64",
        # "HEXCARM",
        # "HEXCARM64",
        # "HEXCMIPS",
        # "HEXCMIPS64",
        # "HEXCPPC",
        # "HEXCPPC64",
        # "HEXCRV",
        # "HEXCRV64",
        # "HEXCARC",
        # "HEXCARC64",
    ]

    i = 0
    for addon in addons:
        i += 1
        license["payload"]["licenses"][0]["add_ons"].append(
            {
                "id": f"48-1337-DEAD-{i:02}",
                "code": addon,
                "owner": license["payload"]["licenses"][0]["id"],
                "start_date": "2024-08-10 00:00:00",
                "end_date": "2033-12-31 23:59:59",
            }
        )
        # for addon in addons:
        #     for platform in platforms:
        #         i += 1
        #         license["payload"]["licenses"][0]["add_ons"].append(
        #             {
        #                 "id": f"48-1337-DEAD-{i:02}",
        #                 "code": addon + platform,
        #                 "owner": license["payload"]["licenses"][0]["id"],
        #                 "start_date": "2024-08-10 00:00:00",
    #                 "end_date": "2033-12-31 23:59:59",
    #             }
    #         )

add_every_addon(license)

def json_stringify_alphabetical(obj):
    return json.dumps(obj, sort_keys=True, separators=(",", ":"))

def buf_to_bigint(buf):
    return int.from_bytes(buf, byteorder="little")

def bigint_to_buf(i):
    return i.to_bytes((i.bit_length() + 7) // 8, byteorder="little")

# Yup, you only have to patch 5c -> cb in libida64.so
pub_modulus_hexrays = buf_to_bigint(
    bytes.fromhex(
        "edfd425cf978546e8911225884436c57140525650bcf6ebfe80edbc5fb1de68f4c66c29cb22eb668788afcb0abbb718044584b810f8970cddf227385f75d5dddd91d4f18937a08aa83b28c49d12dc92e7505bb38809e91bd0fbd2f2e6ab1d2e33c0c55d5bddd478ee8bf845fcef3c82b9d2929ecb71f4d1b3db96e3a8e7aaf93"
    )
)
pub_modulus_patched = buf_to_bigint(
    bytes.fromhex(
        "edfd42cbf978546e8911225884436c57140525650bcf6ebfe80edbc5fb1de68f4c66c29cb22eb668788afcb0abbb718044584b810f8970cddf227385f75d5dddd91d4f18937a08aa83b28c49d12dc92e7505bb38809e91bd0fbd2f2e6ab1d2e33c0c55d5bddd478ee8bf845fcef3c82b9d2929ecb71f4d1b3db96e3a8e7aaf93"
    )
)

private_key = buf_to_bigint(
    bytes.fromhex(
        "77c86abbb7f3bb134436797b68ff47beb1a5457816608dbfb72641814dd464dd640d711d5732d3017a1c4e63d835822f00a4eab619a2c4791cf33f9f57f9c2ae4d9eed9981e79ac9b8f8a411f68f25b9f0c05d04d11e22a3a0d8d4672b56a61f1532282ff4e4e74759e832b70e98b9d102d07e9fb9ba8d15810b144970029874"
    )
)

def decrypt(message):
    decrypted = pow(buf_to_bigint(message), exponent, pub_modulus_patched)
    decrypted = bigint_to_buf(decrypted)
    return decrypted[::-1]

def encrypt(message):
    encrypted = pow(buf_to_bigint(message[::-1]), private_key, pub_modulus_patched)
    encrypted = bigint_to_buf(encrypted)
    return encrypted

exponent = 0x13

def sign_hexlic(payload: dict) -> str:
    data = {"payload": payload}
    data_str = json_stringify_alphabetical(data)

    buffer = bytearray(128)
    # first 33 bytes are random
    for i in range(33):
        buffer[i] = 0x42

    # compute sha256 of the data
    sha256 = hashlib.sha256()
    sha256.update(data_str.encode())
    digest = sha256.digest()

    # copy the sha256 digest to the buffer
    for i in range(32):
        buffer[33 + i] = digest[i]

    # encrypt the buffer
    encrypted = encrypt(buffer)

    return encrypted.hex().upper()

def generate_patched_dll(filename):
    if not os.path.exists(filename):
        print(f"Didn't find {filename}, skipping patch generation")
        return

    with open(filename, "rb") as f:
        data = f.read()

        if data.find(bytes.fromhex("EDFD42CBF978")) != -1:
            print(f"{filename} looks to be already patched :)")
            return
        
        if data.find(bytes.fromhex("EDFD425CF978")) == -1:
            print(f"{filename} doesn't contain the original modulus.")
            return

        data = data.replace(
            bytes.fromhex("EDFD425CF978"), bytes.fromhex("EDFD42CBF978")
        )

        patched_filename = f"{filename}.patched"
        with open(patched_filename, "wb") as f:
            f.write(data)

        print(f"Generated modulus patch to {patched_filename}! To apply the patch, replace the original file with the patched file")

# message = bytes.fromhex(license["signature"])
# print(decrypt(message).hex())
# print(encrypt(decrypt(message)).hex())

license["signature"] = sign_hexlic(license["payload"])

serialized = json_stringify_alphabetical(license)

# write to ida.hexlic
filename = "idapro.hexlic"

with open(filename, "w") as f:
    f.write(serialized)

print(f"Saved new license to {filename}!")

generate_patched_dll("ida32.dll")
generate_patched_dll("ida.dll")
generate_patched_dll("libida32.so")
generate_patched_dll("libida.so")
generate_patched_dll("libida32.dylib")
generate_patched_dll("libida.dylib")
```

这两个文件在安装完IDA之后放到IDA的主目录下，再运行keygen.py，它就把libida.so和libida32.so两个库patch了

然后再打开IDA就能用了

不过好像因为缺了点依赖库没有，一开始打不开，用命令行执行看了一眼是缺了xcb库

```bash
sudo apt install libxcb*
```

安装完就可以打开了

怎么感觉好像9.1反编译快一点

为了方便打开二进制文件，接着配置文件打开方式：

```plain
vim ~/.local/share/applications/ida-pro.desktop

[Desktop Entry]
Name=IDA Pro
Comment=Interactive Disassembler
Exec=/home/garhin/ida-pro-9.1/ida %f
Icon=/home/garhin/ida-pro-9.1/appico.png
Terminal=false
Type=Application
Categories=Development;Debugger;
MimeType=application/x-executable;application/x-elf;application/x-sharedlib;
StartupNotify=true

chmod +x ~/.local/share/applications/ida-pro.desktop
update-desktop-database ~/.local/share/applications
```

然后就可以在二进制文件的打开方式里面找到 IDAPro 了

![](/images/6c826d2726d7f03cf12a9200248e10c8.png)

### Pwntools
```bash
sudo apt install python3-pwntools
```

### Checksec
千万别用`apt install checksec`装，装出来的界面丑爆了还难看清

![](/images/f17735565e6d2105f7f3fe56a16fb518.jpeg)

谁家命令行程序这样布局的，界面小点堆一块了

pwntools 的 checksec 界面好看还高效：

![](/images/4271256d4de7b22f31e1df676f11ecd0.jpeg)

如果嫌前面加个 pwn 麻烦的话可以建个脚本把指令塞进去再塞到 bin 里面

### Seccomp-tools
```python
sudo apt install ruby-dev
sudo gem install seccomp-tools
```

### Pwndbg
不知道为啥我这边不开代理的话github完全上不去（我记得以前不开代理也能上去来着...）

而且这里开了代理但是命令行里面git也是连不上，后面在clash那边开了虚拟网卡模式才连上了...

```bash
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

### Patchelf
```bash
sudo apt install patchelf
```

### Libcsearcher
```bash
git clone https://github.com/lieanu/LibcSearcher.git
cd LibcSearcher
python setup.py develop
```

之前一直没怎么用过这个，不过这个里面的libc都是旧一点的版本了，新版本的都找不到

```python
from LibcSearcher import *

#第二个参数，为已泄露的实际地址,或最后12位(比如：d90)，int类型
obj = LibcSearcher("fgets", 0X7ff39014bd90)

obj.dump("system")        #system 偏移
obj.dump("str_bin_sh")    #/bin/sh 偏移
obj.dump("__libc_start_main_ret")
```

### glibc-all-in-one
这次看看patch的时候还会不会因为路径太长报错，要是还报错就放弃了...自己下下来解压吧...

```bash
git clone https://github.com/matrix1001/glibc-all-in-one.git
cd glibc-all-in-one
```

用法：

```bash
sudo python3 update_list #更新列表
cat list #查看可用版本
sudo ./download <版本> #下载对应版本glibc
```

patchelf没有报错！赢！

### ropper
```python
pipx install ropper
```

### one_gadget
```python
sudo apt -y install ruby
sudo gem install one_gadget
```

### Timeshift
用来创建快照的，防止哪一天被pwn烂了（

```bash
sudo apt install timeshift
```

### WineHQ
```bash
sudo dpkg --add-architecture i386 #确保
sudo mkdir -pm755 /etc/apt/keyrings #创建key保存目录
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key #下载key
sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/noble/winehq-noble.sources #添加ubuntu对应系统版本源
sudo apt install --install-recommends winehq-stable #安装wine
winecfg #wine配置界面
```

这下尬住了，不知道是网易云的问题还是wine没兼容的问题，调DPI动不到显示出来的界面，但是界面按钮什么的却是按DPI放大了

![](/images/27794383988de493d6401b3ec35ee91c.png)

地狱构图（

### Docker
```python
sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo groupadd docker
sudo usermod -aG docker $USER
```

### Qemu
```python
# 安装 QEMU 和 KVM 加速
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
 
# 验证安装
qemu-system-x86_64 --version
```

