---
title: WSL
date: '2024-12-10 23:57:38'
updated: '2025-04-17 21:57:28'
---
## Hyper-V 安装
Windows11 家庭版没有 Hyper-V，可以用指令安装

```python
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V -All /LimitAccess /ALL
pause
```

把上面的指令拷进 txt 再改后缀名 bat，管理员权限执行，等一会就可以了

## 断网解决
```python
wsl --install --no-distribution
```

重装一遍 WSL，不会动到虚拟机

## 指令
`wsl --update` 更新WSL（WSL太久不更新可能打不开虚拟机）

`wsl -l -v`<font style="color:#000000;">查看列表以及状态</font>

![](/images/6e0fc9968086c8d082cacd4f87bdd62f.png)

`wsl --shutdown NAME`关闭虚拟机

## 快捷键
`ctrl`+`d`logout 退出 root 用户或退出虚拟机

系统任意界面`ctrl`+`X`调出 Windows 的高级用户菜单，再按`A`就可以以管理员身份启动 powershell 来对 wsl 进行操作

![](/images/243de5d324abb1c12c91b7c809230991.png)

## 迁移虚拟机至其他盘
在微软商店下载安装的 WSL Ubuntu22.04 LTS 默认是在系统的 C 盘的，占了太多 C 盘的空间，

可以把它迁移到其他盘

注意：要记住`虚拟机文件名 NAME`和`原来虚拟机内用户名 USER`，建议全程在管理员运行的 Powershell 当中操作

1. 先检查 wsl 里面要迁移的虚拟机是否关机
2. 导出对应虚拟机的备份

`wsl --export NAME PATH\temp.tar`

3. 确认对应目录下有 temp.tar 文件之后，注销原来的虚拟机

`wsl --unregister NAME`

4. 将备份恢复到想要迁移到的路径去

`wsl --import NAME TARGETPATH PATH\temp.tar`

5. 把默认登录用户改回原样

方法一：非导入情况

`NAME config --default-user USER`

方法二：通用

编辑配置文件

```python
sudo vim /etc/wsl.conf
```

在文件尾部加上：

```python
[user]
default = 用户
```

