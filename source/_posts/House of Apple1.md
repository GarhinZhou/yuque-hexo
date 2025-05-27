---
title: House of Apple1
date: '2025-05-14 23:19:30'
updated: '2025-05-25 17:08:09'
---
## 利用条件
1. 程序从 main 函数返回或能调用 exit 函数
2. 能泄露出 heap 地址和 libc 地址
3. 能使用一次 largebin attack（一次即可）

## 利用原理
当程序从 main 函数返回或者执行 exit 函数的时候，均会调用 fcloseall 函数，该调用链为：

```c
exit
	fcloseall
		_IO_cleanup
			_IO_flush_all_lockp
				_IO_OVERFLOW
```

