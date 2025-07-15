---
title: Gadgets获取
date: '2025-01-12 20:02:12'
updated: '2025-06-15 13:30:04'
---
## one_gadget
```python
one_gadget libc文件路径
```

![](/images/5ce2dac791e6fe0ca72b39e541251aa7.png)

注意：要满足`execve("/bin/sh",0,0)`才能 getshell

## ROPgadget
### 找 gadget
```shell
ROPgadget --binary 文件名 --only "pop|ret" | grep rdi
```

### 找 strings
```shell
ROPgadget --binary 文件名 --string '/bin/sh'
```

### 静态编译一把梭
```python
ROPgadget --binary 文件名 --ropchain
```

## Ropper
### 安装
#### 安装 libssl1.1
```c
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt-get update
sudo apt-get install libssl1.1
```

#### 安装 cmake（ppa安装）
添加签名密钥

```shell
wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | sudo apt-key add -
```

将存储库添加到源列表并进行更新

```shell
sudo apt-add-repository 'deb https://apt.kitware.com/ubuntu/ bionic main'
sudo apt-get update
```

然后再使用apt安装就是最新版本的cmake啦

```c
sudo apt install cmake
```

#### 安装 keystone-engine
```c
git clone https://github.com/keystone-engine/keystone.git
cd keystone
mkdir build
cd build
../make-share.sh
sudo make install
sudo ldconfig
cd ../bindings/python
sudo make install3
```

#### 安装 ropper
```c
sudo pip3 install filebytes==0.9.18
git clone https://github.com/sashs/Ropper.git
cd Ropper
pip3 install ropper
```

### 使用
![](/images/8779be3b04515d90484fd614476aa258.png)

`file 文件路径`可以批量导入文件

`jmp 寄存器`查找文件中 jmp 以及 call 对应寄存器的指令

> 可以用?表示任意字符，即 r?x 来筛选对应寄存器
>
> 用%表示任意字符串，即利用 mov rdi , [%] 来找 gadgets
>

`search 指令`用search可以更加精准的查找相关的指令

![](/images/5656cfe6a24c752edec680b4fdb94ba3.png)

