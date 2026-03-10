---
title: stdin任意写
date: '2025-12-20 10:53:18'
updated: '2025-12-27 09:50:50'
---
# scanf
函数参数：

```c
scanf(const char *format, ...);
```

执行链：

```plain
scanf
  __nldbl__IO_vfscanf
    __vfscanf_internal
      _IO_file_xsgetn
        __underflow
          _IO_UNDERFLOW -> _IO_new_file_underflow
              _IO_SYSREAD
```

调用到内部的时候的长度 n 会从解析的格式化字符串里面获取，比如 %20s 就是 20 个字节长度

_IO_file_xsgetn：

```c
size_t
_IO_file_xsgetn (FILE *fp, void *data, size_t n)
{
    size_t want, have;
    ssize_t count;
    char *s = data;

    want = n;

    if (fp->_IO_buf_base == NULL)
    {
        /* Maybe we already have a push back pointer.  */
        if (fp->_IO_save_base != NULL)
        {
            free (fp->_IO_save_base);
            fp->_flags &= ~_IO_IN_BACKUP;
        }
        _IO_doallocbuf (fp);
    }

    while (want > 0)
    {
        have = fp->_IO_read_end - fp->_IO_read_ptr;
        if (want <= have)
        {
            memcpy (s, fp->_IO_read_ptr, want);
            fp->_IO_read_ptr += want;
            want = 0;
        }
        else
        {
            if (have > 0)
            {
                s = __mempcpy (s, fp->_IO_read_ptr, have);
                want -= have;
                fp->_IO_read_ptr += have;
            }

            /* Check for backup and repeat */
            if (_IO_in_backup (fp))
            {
                _IO_switch_to_main_get_area (fp);
                continue;
            }

            /* If we now want less than a buffer, underflow and repeat
	     the copy.  Otherwise, _IO_SYSREAD directly to
	     the user buffer. */
            if (fp->_IO_buf_base
          && want < (size_t) (fp->_IO_buf_end - fp->_IO_buf_base))
            {
                if (__underflow (fp) == EOF)
                    ...
```

要执行到`__underflow`的判断：

+ _IO_buf_base 不为 0
+ _IO_read_end - _IO_read_ptr 等于 0
+ _IO_buf_end - _IO_buf_base 大于读取长度 n

接着看`__underflow`：

```c
int
__underflow (FILE *fp)
{
  if (_IO_vtable_offset (fp) == 0 && _IO_fwide (fp, -1) != -1)
    return EOF;

  if (fp->_mode == 0)
    _IO_fwide (fp, -1);
  if (_IO_in_put_mode (fp))
    if (_IO_switch_to_get_mode (fp) == EOF)
      return EOF;
  if (fp->_IO_read_ptr < fp->_IO_read_end)
    return *(unsigned char *) fp->_IO_read_ptr;
  if (_IO_in_backup (fp))
    {
      _IO_switch_to_main_get_area (fp);
      if (fp->_IO_read_ptr < fp->_IO_read_end)
	return *(unsigned char *) fp->_IO_read_ptr;
    }
  if (_IO_have_markers (fp))
    {
      if (save_for_backup (fp, fp->_IO_read_end))
	return EOF;
    }
  else if (_IO_have_backup (fp))
    _IO_free_backup_area (fp);
  return _IO_UNDERFLOW (fp);
}
libc_hidden_def (__underflow)
```

下面这段源代码真找不到...直接 copy 过来了...

```c
#define _IO_UNDERFLOW(FP) JUMP0 (__underflow, FP)
--------------------------------------------------------------------------
const struct _IO_jump_t _IO_file_jumps =
{
    ...
    JUMP_INIT(underflow, _IO_file_underflow),
    ...
}
--------------------------------------------------------------------------
# define _IO_new_file_underflow _IO_file_underflow
```

然后看`_IO_new_file_underflow`：

```c
int
_IO_new_file_underflow (FILE *fp)
{
  ssize_t count;

  /* C99 requires EOF to be "sticky".  */
  if (fp->_flags & _IO_EOF_SEEN)
    return EOF;

  if (fp->_flags & _IO_NO_READS)
    {
      fp->_flags |= _IO_ERR_SEEN;
      __set_errno (EBADF);
      return EOF;
    }
  if (fp->_IO_read_ptr < fp->_IO_read_end)
    return *(unsigned char *) fp->_IO_read_ptr;

  if (fp->_IO_buf_base == NULL)
    {
      /* Maybe we already have a push back pointer.  */
      if (fp->_IO_save_base != NULL)
	{
	  free (fp->_IO_save_base);
	  fp->_flags &= ~_IO_IN_BACKUP;
	}
      _IO_doallocbuf (fp);
    }

  /* FIXME This can/should be moved to genops ?? */
  if (fp->_flags & (_IO_LINE_BUF|_IO_UNBUFFERED))
    {
      /* We used to flush all line-buffered stream.  This really isn't
	 required by any standard.  My recollection is that
	 traditional Unix systems did this for stdout.  stderr better
	 not be line buffered.  So we do just that here
	 explicitly.  --drepper */
      _IO_acquire_lock (stdout);

      if ((stdout->_flags & (_IO_LINKED | _IO_NO_WRITES | _IO_LINE_BUF))
	  == (_IO_LINKED | _IO_LINE_BUF))
	_IO_OVERFLOW (stdout, EOF);

      _IO_release_lock (stdout);
    }

  _IO_switch_to_get_mode (fp);

  /* This is very tricky. We have to adjust those
     pointers before we call _IO_SYSREAD () since
     we may longjump () out while waiting for
     input. Those pointers may be screwed up. H.J. */
  fp->_IO_read_base = fp->_IO_read_ptr = fp->_IO_buf_base;
  fp->_IO_read_end = fp->_IO_buf_base;
  fp->_IO_write_base = fp->_IO_write_ptr = fp->_IO_write_end
    = fp->_IO_buf_base;

  count = _IO_SYSREAD (fp, fp->_IO_buf_base, fp->_IO_buf_end - fp->_IO_buf_base);
```

这里要调用到`_IO_SYSREAD`需要满足的条件：

1. _flags & _IO_EOF_SEEN 为空 0x10
2. _flags & _IO_NO_READS 为空 0x4
3. _IO_read_ptr 等于 _IO_read_end
4. _IO_buf_base 不为空
5. _flags & (_IO_LINE_BUF|_IO_UNBUFFERED)为空 0x200 0x2

最终的利用点：

```c
_IO_SYSREAD(fp, fp->_IO_buf_base, fp->_IO_buf_end-fp->_IO_buf_base);
```

总结一下这些条件：

1. `_IO_EOF_SEEN`、`_IO_NO_READS`、`_IO_LINE_BUF`、`_IO_UNBUFFERED` 位为 0
2. `_IO_read_ptr` 等于 `_IO_read_end`

> 正常来说 stdin 对于上面这两个条件不需要修改也能实现的，即使不满足`_IO_UNBUFFERED`也可以实现
>

3. `_IO_buf_base` 为目标写入地址，`_IO_buf_end` - `_IO_buf_base` 大于读取长度 n

# fread
```c
fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
```

fread 的条件跟 scanf 一样，因为到后面都是调用到了`_IO_file_xsgetn`

```plain
fread
  __fread_chk
    _IO_sgetn
      _IO_XSGETN
        _IO_file_xsgetn
          __underflow
            _IO_UNDERFLOW -> _IO_new_file_underflow
                _IO_SYSREAD
```

# fgets
```c
fgets(char *str, int n, FILE *stream);
```

好家伙 fgets 这边不太一样：

```plain
fgets
  __fgets_chk
    _IO_getline
      _IO_getline_info
        __uflow
          _IO_UFLOW -> _IO_default_uflow
              _IO_UNDERFLOW -> _IO_new_file_underflow
                _IO_SYSREAD
```

fgets 有点不一样，fread 和 scanf 在`_IO_read_ptr` 默认情况下不等于 `_IO_read_end`的时候也能执行到 SYSREAD，但是 fgets 必须得相等才能执行到 SYSREAD

> 总之到时候多动调看看，不通应该就是这些条件没满足的问题
>

