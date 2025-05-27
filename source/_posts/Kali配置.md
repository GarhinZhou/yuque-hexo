---
title: Kali配置
date: '2025-04-10 18:24:32'
updated: '2025-04-10 18:33:54'
---
换源路径：

```python
vim /etc/apt/sources.list
```

1. 中科大源

```python
deb http://mirrors.ustc.edu.cn/kali kali-rolling main non-free non-free-firmware contrib
deb-src http://mirrors.ustc.edu.cn/kali kali-rolling main non-free non-free-firmware contrib
```

2. 阿里源

```python
deb http://mirrors.aliyun.com/kali kali-rolling main non-free non-free-firmware contrib
deb-src http://mirrors.aliyun.com/kali kali-rolling main non-free non-free-firmware contrib
```

3. 清华源

```python
deb http://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/kali kali-rolling main contrib non-free non-free-firmware
```

