---
title: Linux
date: '2024-12-17 11:01:27'
updated: '2025-04-27 16:38:29'
---
## 换源
## 特殊文件、路径
`proc/self`当中的`environ`是环境变量，动态 flag 经常出现在这里

## 快捷键和常用指令
`bash`当中可以使用快捷键`Tab`补全文件名或者命令名

`sudo su` 切换到 root 用户

`ctrl+d` 退出 root 用户；也可以表示eof，不需要回车

## 流
&号后面接文件描述符是指流向文件描述符

```python
cat flag >> &1
```

