---
title: Godot逆向
date: '2025-09-05 11:03:48'
updated: '2025-10-09 20:11:40'
---
遇到一个特别喜欢的小游戏，想着把立绘解包出来存着

一开始还不知道游戏是用什么开发的，先是用MultiExtractor：

https://www.multiextractor.com/index.html

看了看能不能把图片解压出来，结果只能拿到icon，看了眼是Godot，接着上吾爱破解找了工具：

godot逆向工具：

github：https://github.com/GDRETools/gdsdecomp

可能是因为小游戏所以没加密，直接用这个工具打开exe就看到整个项目的资源了

在上面可以直接搜文件：

![](/images/e94c1f3b77110dfdf0f0664a81abfff0.png)

再在下面选好路径就可以把整个项目解包到对应的路径了

但是如果游戏有加密的话就得用IDA把密钥先逆出来了：

用IDA找字符串 gdscript::load_byte_code ，看引用函数第一个里面的

![](/images/ef149212226273965f86026c02d59756.png)

![](/images/538889c4a1257abfc3874d201ae8ae98.png)

3EE365C93414725F6690676212188D1AED7D90B695B7D585FD066DF0FBC6DAE7

![](/images/0998661ef3b324de23b35ee08a2669a8.png)

![](/images/a2418bf6ddfac9479f370a09ee14ff2f.png)

