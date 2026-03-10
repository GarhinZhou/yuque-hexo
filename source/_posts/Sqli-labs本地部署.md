---
title: Sqli-labs本地部署
date: '2025-11-07 22:18:23'
updated: '2025-11-07 22:44:06'
---
真没想到 github 上面还有找不同：

![](/images/1692a3578e6136d1b0e4f806a22c6523.png)

![](/images/c4ecd6d9f3982ddd5a6043c7bd8bfff4.png)

一开始还真没细看 stars 数...

## 安装 PHPStudy
[https://www.xp.cn/download.html](https://www.xp.cn/download.html)

## 下载 Sqli-labs
直接下载 zip：

[https://github.com/Audi-1/sqli-labs](https://github.com/Audi-1/sqli-labs)

解压完之后把里面的文件夹改名成 sql-labs

然后放到 PHPStudy 的根目录

![](/images/098d97b7bc60c4c860bbaad8b3aa88aa.png)

设置里面的用户密码（D:\phpstudy_pro\WWW\sqli-labs\sql-connections\db-creds.inc）：

![](/images/e17783a013a43aaae0ba50cf0bec4cd6.png)

然后修改数据库的密码为设置的密码：

![](/images/9e0bfddf3d2758df0e40e963f81afa55.png)

最后配置 PHP 版本（改成 5.X 版本）：

![](/images/437603c90fc84d820c957cee22fa6246.png)

然后访问`[http://127.0.0.1/sqli-labs/](http://127.0.0.1/sqli-labs/Less-1)`就可以看见界面了：

![](/images/64125af0ad51b75d30ba9fe3d3bf2281.png)

点`Setup/reset Database for labs`初始化完就可以做题了：

![](/images/cbfbeed031a202b9f9dbfc3a6aba940a.png)

