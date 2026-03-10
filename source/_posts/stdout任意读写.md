---
title: stdout任意读写
date: '2025-11-29 09:41:59'
updated: '2025-12-27 09:13:23'
---
# 任意读（泄漏 libc 地址）
## puts
```c
_IO_puts
    _IO_sputn -> _IO_XSPUTN -> _IO_new_file_xsputn
        _IO_OVERFLOW -> _IO_new_file_overflow
            _IO_do_write -> new_do_write
                _IO_SYSWRITE
```

代码链：

```c
int
_IO_puts (const char *str)
{
  int result = EOF;
  size_t len = strlen (str);
  _IO_acquire_lock (stdout);

  if ((_IO_vtable_offset (stdout) != 0
       || _IO_fwide (stdout, -1) == -1)
      && _IO_sputn (stdout, str, len) == len
      ...
}
```

上面这里的(_IO_vtable_offset (stdout) != 0 一定不满足，因为 stdout 和 stdin 的这个值默认都是 0

![](/images/539d0640d294ddda0ded5b81936862da.png)

所以得看后面的_IO_fwide (stdout, -1) == -1)，这个函数是宏定义的：

![](/images/20a0802f7ec0386e9280cc4ce9dac47c.png)

总结就是检查 stdout 的 _mode（了解了一下，这个 _mode 是用来表示当前的字符流的模式，有未设置 0、窄字节 1 和宽字节-1 模式），这里 <font style="color:#DF2A3F;">_mode=0 或者 -1 就能通过检查</font>

```c
size_t
_IO_new_file_xsputn (FILE *f, const void *data, size_t n)
{
  const char *s = (const char *) data;
  size_t to_do = n;
  int must_flush = 0;
  size_t count = 0;

  if (n <= 0)
    return 0;
  if ((f->_flags & _IO_LINE_BUF) && (f->_flags & _IO_CURRENTLY_PUTTING))
    {
      count = f->_IO_buf_end - f->_IO_write_ptr;
      if (count >= n)
	{
	  const char *p;
	  for (p = s + n; p > s; )
	    {
	      if (*--p == '\n')
		{
		  count = p - s + 1;
		  must_flush = 1;
		  break;
		}
	    }
	}
    }
  else if (f->_IO_write_end > f->_IO_write_ptr)
    count = f->_IO_write_end - f->_IO_write_ptr; /* Space available. */

  /* Then fill the buffer. */
  if (count > 0)
    {
      if (count > to_do)
	count = to_do;
      f->_IO_write_ptr = __mempcpy (f->_IO_write_ptr, s, count);
      s += count;
      to_do -= count;
    }
  if (to_do + must_flush > 0)
    {
      size_t block_size, do_write;
      /* Next flush the (full) buffer. */
      if (_IO_OVERFLOW (f, EOF) == EOF)
          ...
}
```

这里正常来说 to_do 都是大于 0 的，所以不用特意匹配

```c
int
_IO_new_file_overflow (FILE *f, int ch)
{
  if (f->_flags & _IO_NO_WRITES) /* SET ERROR */
    {
      f->_flags |= _IO_ERR_SEEN;
      __set_errno (EBADF);
      return EOF;
    }
  /* If currently reading or no buffer allocated. */
  if ((f->_flags & _IO_CURRENTLY_PUTTING) == 0 || f->_IO_write_base == NULL)
    {
      /* Allocate a buffer if needed. */
      if (f->_IO_write_base == NULL)
	{
	  _IO_doallocbuf (f);
	  _IO_setg (f, f->_IO_buf_base, f->_IO_buf_base, f->_IO_buf_base);
	}
      if (__glibc_unlikely (_IO_in_backup (f)))
	{
	  size_t nbackup = f->_IO_read_end - f->_IO_read_ptr;
	  _IO_free_backup_area (f);
	  f->_IO_read_base -= MIN (nbackup,
				   f->_IO_read_base - f->_IO_buf_base);
	  f->_IO_read_ptr = f->_IO_read_base;
	}

      if (f->_IO_read_ptr == f->_IO_buf_end)
	f->_IO_read_end = f->_IO_read_ptr = f->_IO_buf_base;
      f->_IO_write_ptr = f->_IO_read_ptr;
      f->_IO_write_base = f->_IO_write_ptr;
      f->_IO_write_end = f->_IO_buf_end;
      f->_IO_read_base = f->_IO_read_ptr = f->_IO_read_end;

      f->_flags |= _IO_CURRENTLY_PUTTING;
      if (f->_mode <= 0 && f->_flags & (_IO_LINE_BUF | _IO_UNBUFFERED))
	f->_IO_write_end = f->_IO_write_ptr;
    }
  if (ch == EOF)
    return _IO_do_write (f, f->_IO_write_base, f->_IO_write_ptr - f->_IO_write_base);
    ...
}
```

这里就有两个地方需要绕过了：

1. f->_flags & _IO_NO_WRITES（`#define _IO_NO_WRITES 8`）
2. f->_flags & _IO_CURRENTLY_PUTTING) == 0 || f->_IO_write_base == NULL（`#define _IO_CURRENTLY_PUTTING 0x800`）

这两个都不能满足，不能进 if 线，其实就是 flags 的 0xfbad0000 的最低半字节不能是 8 或者 f，还有倒数第三个半字节得是 8 或者 f

第二个 if 不能进是因为进入之后会动到 FILE 结构体的几个缓冲区指针，直接让利用链失效了

```c
int
_IO_new_do_write (FILE *fp, const char *data, size_t to_do)
{
  return (to_do == 0
	  || (size_t) new_do_write (fp, data, to_do) == to_do) ? 0 : EOF;
}
```

这里的 to_do 也不用特地绕过

```c
static size_t
new_do_write (FILE *fp, const char *data, size_t to_do)
{
  size_t count;
  if (fp->_flags & _IO_IS_APPENDING)
    fp->_offset = _IO_pos_BAD;
  else if (fp->_IO_read_end != fp->_IO_write_base)
    {
      off64_t new_pos
	= _IO_SYSSEEK (fp, fp->_IO_write_base - fp->_IO_read_end, 1);
      if (new_pos == _IO_pos_BAD)
	return 0;
      fp->_offset = new_pos;
    }
  count = _IO_SYSWRITE (fp, data, to_do);
    ...
}
```

这里也是两个点：

1. fp->_flags & _IO_IS_APPENDING（`#define _IO_IS_APPENDING 0x1000`）
2. fp->_IO_read_end != fp->_IO_write_base

但是因为这里第二点是很难不满足的，因为要想要这两个值相等会影响到_IO_write_base 的取值

所以直接进 if 线，即满足：

fp->_flags & _IO_IS_APPENDING（`#define _IO_IS_APPENDING 0x1000`）

## printf
跟 puts 一样的条件

## 总结
总结下来就是`_flags`要满足：`0xfbad1800`，

然后`_mode=-1``_fileno=1`（结果要输出到标准输出，不过后面这两点 stdout 都满足）

在改 `_IO_write_base` 这方面，可以从 flags 一块写下去，把 stdout 的`_IO_read_ptr` `_IO_read_end` `_IO_read_base`直接也覆盖，一直到`_IO_write_base`的尾字节改为`\x00`，然后就能泄漏出来 libc 的地址了

别忘了本身 stdout 的缓冲区得是 0，不然就没法正常输出

![](/images/441f09a183f8458b9a141b4297e70a8c.png)

但是我有一个很大的疑惑，stdout 结构体不是在 libc 里面吗？要改到 stdout 的话不需要泄漏 libc 吗...

问了 AI 是说利用部分覆盖得到 IO_FILE 结构体的指针，应该跟堆栈风水有关系

# 任意写
条件：

+ `_IO_write_end`大于`_IO_write_ptr`（`_IO_write_ptr` 是目标地址）
+ 输出函数的内容可控

poc：

```c
#include <stdio.h>
#include <unistd.h>

int main(){
    setvbuf(stdout, 0, 2, 0);
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stderr, 0, 2, 0);

    printf("I give you soemthing nice! Here: %p\n", &read);
    unsigned long addr;
    char buf[0x100];
    read(0, buf, 0x100);
    scanf("%lu", &addr);
    printf("%lx\n", addr);
    read(0, (void *)addr, 0x100);
    printf(buf);
}
```

## puts
```c
size_t
_IO_new_file_xsputn (FILE *f, const void *data, size_t n)
{
    const char *s = (const char *) data;
    size_t to_do = n;
    int must_flush = 0;
    size_t count = 0;

    if (n <= 0)
        return 0;
    /* This is an optimized implementation.
     If the amount to be written straddles a block boundary
     (or the filebuf is unbuffered), use sys_write directly. */

    /* First figure out how much space is available in the buffer. */
    if ((f->_flags & _IO_LINE_BUF) && (f->_flags & _IO_CURRENTLY_PUTTING))
    {
        count = f->_IO_buf_end - f->_IO_write_ptr;
        if (count >= n)
        {
            const char *p;
            for (p = s + n; p > s; )
            {
                if (*--p == '\n')
                {
                    count = p - s + 1;
                    must_flush = 1;
                    break;
                }
            }
        }
    }
    else if (f->_IO_write_end > f->_IO_write_ptr)
        count = f->_IO_write_end - f->_IO_write_ptr; /* Space available. */

    /* Then fill the buffer. */
    if (count > 0)
    {
        if (count > to_do)
            count = to_do;
        f->_IO_write_ptr = __mempcpy (f->_IO_write_ptr, s, count);
```

## printf
跟 puts 条件一样

