---
title: Glibc-all-in-one使用
date: '2025-02-16 18:50:36'
updated: '2025-06-25 16:52:17'
---
## 安装
克隆项目仓库

```shell
git clone https://github.com/matrix1001/glibc-all-in-one.git
cd glibc-all-in-one
```

更新 glibc 版本列表

```shell
python3 update_list
```

更新完成后会生成两个文件：list 和 old_list，分别包含当前和旧版本的 glibc 列表。

## 换源
```shell
vim download
vim update_list
```

把这两个文件里面下载的地址换成阿里云源：

```shell
https://mirrors.aliyun.com/ubuntu/pool/main/g/glibc/
```

## 使用
### 查看可用 glibc 版本
```shell
cat list
```

### 下载 glibc
```shell
./download 2.34-0ubuntu3_amd64
```

下载后的 glibc 文件存在 libs 目录下

### 解压 deb 包
glibc-all-in-one 不知道抽什么风下不了 deb 包，

上[吞拿站](https://mirror.tuna.tsinghua.edu.cn/ubuntu/pool/main/g/glibc/)下，下下来再用 extract 解压

注意包名：libc6_2.27-3ubuntu1_amd64.deb

```shell
./extract libc6_2.34-0ubuntu3_amd64.deb libs路径
```

在高版本的 libc deb 包解压的时候会用到 zstd，这时候可能 extract 会报错：

![](/images/07f150b0807e5905b4f6deec1698bc91.png)

直接`sudo apt install zstd`安装完再解压就行了

