---
title: FILE结构
date: '2025-05-11 13:24:54'
updated: '2025-05-24 00:43:42'
---
## _IO_FILE_plus
FILE 在 Linux 系统的标准 IO 库中是用于描述文件的结构，称为文件流

FILE 结构在程序执行 fopen 等函数时会进行创建，并分配在堆中。我们常定义一个指向 FILE 结构的指针来接收这个返回值

`_IO_FILE`结构外包裹着另一种结构`_IO_FILE_plus`，其中包含了一个重要的指针 ：vtable ，指向了一系列函数指针

```c
struct _IO_FILE_plus
{
    _IO_FILE    file; //要注意！！这里存的不是指针，是一整个结构体！！！
    IO_jump_t   *vtable;
}
```

vtable 是 `IO_jump_t` 类型的指针，`IO_jump_t`中保存了一些函数指针，在后面我们会看到在一系列标准 IO 函数中会调用这些函数指针

```c
struct _IO_FILE {
	int _flags;       /* High-order word is _IO_MAGIC; rest is flags. */
	#define _IO_file_flags _flags

	/* The following pointers correspond to the C++ streambuf protocol. */
	/* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
	char* _IO_read_ptr;   /* Current read pointer */
	char* _IO_read_end;   /* End of get area. */
	char* _IO_read_base;  /* Start of putback+get area. */
	char* _IO_write_base; /* Start of put area. */
	char* _IO_write_ptr;  /* Current put pointer. */
	char* _IO_write_end;  /* End of put area. */
	char* _IO_buf_base;   /* Start of reserve area. */
	char* _IO_buf_end;    /* End of reserve area. */
	/* The following fields are used to support backing up and undo. */
	char *_IO_save_base; /* Pointer to start of non-current get area. */
	char *_IO_backup_base;  /* Pointer to first valid character of backup area */
	char *_IO_save_end; /* Pointer to end of non-current get area. */

	struct _IO_marker *_markers;

	struct _IO_FILE *_chain;

	int _fileno;
	#if 0
	int _blksize;
	#else
	int _flags2;
	#endif
	_IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

	#define __HAVE_COLUMN /* temporary */
	/* 1+column number of pbase(); 0 is unknown. */
	unsigned short _cur_column;
	signed char _vtable_offset;
	char _shortbuf[1];

	/*  char* _save_gptr;  char* _save_egptr; */

	_IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE //如果_IO_USE_OLD_IO_FILE有定义才会有_IO_FILE_complete
};

struct _IO_FILE_complete
{
	struct _IO_FILE _file;
#endif
	#if defined _G_IO_IO_FILE_VERSION && _G_IO_IO_FILE_VERSION == 0x20001
	_IO_off64_t _offset;
	# if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
	/* Wide character stream stuff.  */
	struct _IO_codecvt *_codecvt;
	struct _IO_wide_data *_wide_data;
	struct _IO_FILE *_freeres_list;
	void *_freeres_buf;
	# else
	void *__pad1;
	void *__pad2;
	void *__pad3;
	void *__pad4;

	size_t __pad5;
	int _mode;
	/* Make sure we don't get into trouble again.  */
	char _unused2[15 * sizeof (int) - 4 * sizeof (void *) - sizeof (size_t)];
	#endif
};
```

`_IO_FILE`在满足`_IO_USE_OLD_IO_FILE`的情况下才会转变完善为`_IO_FILE_complete`

进程中的 FILE 结构会通过_chain 域彼此连接形成一个链表，链表头部用全局变量`_IO_list_all` 表示，通过这个值我们可以遍历所有的 FILE 结构

在标准 I/O 库中，每个程序启动时有三个文件流是自动打开的：stdin、stdout、stderr。因此在初始状态下，`_IO_list_all`指向了一个有这些文件流构成的链表

我们可以在 libc.so 中找到 stdin\stdout\stderr 等符号，这些符号是指向 FILE 结构的指针，真正结构的符号是：

```c
_IO_2_1_stderr_
_IO_2_1_stdout_
_IO_2_1_stdin_
```

一个进程中的所有FILE结构会通过`_chain`来连接成一个单向链表，并通过_IO_list_all来记录链表头部。而这个变量在进程一开始是直接指向_IO_2_1_stderr_的

## vtable
vtable 是一个IO_jump_t 类型的结构体指针，而这个IO_jump_t 结构体是这样的：

是一个虚表，存了一系列 IO 函数的指针

```c
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
};
```

在libc中定义的vtable有`_IO_file_jumps`, `_IO_str_jumps`, `_IO_cookie_jumps`等

<font style="background-color:#FBDE28;">FILE头和 vtable的偏移在64位下一般是 0xd8 大小</font>

如果从内存层面看（把_IO_FILE_plus 里面包含的 _IO_FILE 结构体展开来看），整个_IO_FILE_plus结构体内部偏移如下：

```c
0x0   _flags
0x8   _IO_read_ptr
0x10  _IO_read_end
0x18  _IO_read_base
0x20  _IO_write_base
0x28  _IO_write_ptr
0x30  _IO_write_end
0x38  _IO_buf_base
0x40  _IO_buf_end
0x48  _IO_save_base
0x50  _IO_backup_base
0x58  _IO_save_end
0x60  _markers
0x68  _chain
0x70  _fileno
0x74  _flags2
0x78  _old_offset
0x80  _cur_column
0x82  _vtable_offset
0x83  _shortbuf
0x88  _lock
0x90  _offset
0x98  _codecvt
0xa0  _wide_data
0xa8  _freeres_list
0xb0  _freeres_buf
0xb8  __pad5
0xc0  _mode
0xc4  _unused2 
-----往上是_IO_FILE 结构体-----往下是_IO_FILE_plus结构体-----
0xd8  vtable
```

动调用 pwndbg 看就是这样：

![](/images/e0823805dfd31c911eb898225329d665.png)

还可以用

```c
p/x *(struct _IO_jump_t *)_IO_list_all->vtable
```

看 vtable 的内容

![](/images/238eafe467bc3f985a78e08467632648.png)

甚至可以`p/x *(struct _IO_FILE_plus *)_IO_list_all->file->_chain->_chain` 一路 chain 下去看（，之前很少用 pwndbg 的结构体相关的东西，没想到这么好用

感觉对结构体的理解能更深刻一点，理解之后这几幅图就很形象了：

![](/images/a3d2edeac6965c5fa60c5cb4c27e2aac.png)

其实跟 _IO_FILE_plus 相同的还有一个_IO_FILE_complete_plus 结构体

![](/images/5a5caeea5f65281319a3193598e0d8ae.png)

![](/images/640ffc397895a1ab69bd2518321befa9.png)

## IO 函数调用链
### fread/fwrite
都是标准 IO 库函数

+ fread 的代码位于 / libio/iofread.c 中，函数名为_IO_fread，但真正的功能实现在子函数_IO_sgetn 中，在_IO_sgetn 函数中会调用_IO_XSGETN，而_IO_XSGETN 是_IO_FILE_plus.vtable 中的函数指针，在调用这个函数时会首先取出 vtable 中的指针然后再进行调用
+ fwrite 的代码位于 / libio/iofwrite.c 中，函数名为_IO_fwrite。 在_IO_fwrite 中主要是调用_IO_XSPUTN 来实现写入的功能，在_IO_XSPUTN 对应的默认函数_IO_new_file_xsputn 中会调用同样位于 vtable 中的_IO_OVERFLOW，_IO_OVERFLOW 默认对应的函数是_IO_new_file_overflow，在_IO_new_file_overflow 内部最终会调用系统接口 write 函数



fread 函数原型：

```c
size_t fread (void *__restrict __ptr, size_t __size,size_t __n, FILE *__restrict __stream);
```

又有宏定义`#define fread(p, m, n, s) _IO_fread (p, m, n, s)`

于是我们追踪_IO_fread函数：

```c
size_t
_IO_fread (void *buf, size_t size, size_t count, FILE *fp)
{
  size_t bytes_requested = size * count;
  size_t bytes_read;
  CHECK_FILE (fp, 0);
  if (bytes_requested == 0)
    return 0;
  _IO_acquire_lock (fp);
  bytes_read = _IO_sgetn (fp, (char *) buf, bytes_requested);
  _IO_release_lock (fp);
  return bytes_requested == bytes_read ? count : bytes_read / size;
}
```

_IO_sgetn函数（在genops.c中）：

```c
size_t
_IO_sgetn (FILE *fp, void *data, size_t n)
{
  /* FIXME handle putback buffer here! */
  return _IO_XSGETN (fp, data, n);
}
```

_IO_XSGETN就是vtable中的函数指针之一，默认指向_IO_file_xsgetn

### fopen
#### 创建 FILE 结构过程
首先在 fopen 对应的函数__fopen_internal 内部会调用 malloc 函数，分配 FILE 结构的空间。因此我们可以获知 FILE 结构是存储在堆上的

但是需要注意的是标准流位于 libc.so 的数据段，而使用 fopen 创建的文件流是分配在堆内存上的

```c
*new_f = (struct locked_FILE *) malloc (sizeof (struct locked_FILE));
```

之后会为创建的 FILE 初始化 vtable，并调用_IO_file_init 进一步初始化操作

```c
_IO_JUMPS (&new_f->fp) = &_IO_file_jumps;
_IO_file_init (&new_f->fp);
```

在_IO_file_init 函数的初始化操作中，会调用_IO_link_in 把新分配的 FILE 链入_IO_list_all 为起始的 FILE 链表中

```c
void _IO_link_in (fp)
     struct _IO_FILE_plus *fp; //定义一个_IO_FILE_plus结构体指针
{
    if ((fp->file._flags & _IO_LINKED) == 0)
    {
      fp->file._flags |= _IO_LINKED;
      fp->file._chain = (_IO_FILE *) _IO_list_all;
      _IO_list_all = fp;
      ++_IO_list_all_stamp;
    }
}
```

之后__fopen_internal 函数会调用_IO_file_fopen 函数打开目标文件，_IO_file_fopen 会根据用户传入的打开模式进行打开操作，总之最后会调用到系统接口 open 函数

```c
if (_IO_file_fopen ((_IO_FILE *) new_f, filename, mode, is32) != NULL)
    return __fopen_maybe_mmap (&new_f->fp.file);
```

总结一下 fopen 的操作

1. 使用 malloc 分配 FILE 结构
2. 设置 FILE 结构的 vtable
3. 初始化分配的 FILE 结构
4. 将初始化的 FILE 结构链入 FILE 结构链表中
5. 调用系统调用打开文件

### fclose
关闭一个文件流，使用 fclose 就可以把缓冲区内最后剩余的数据输出到磁盘文件中，并释放文件指针和有关的缓冲区

fclose 首先会调用_IO_unlink_it 将指定的 FILE 从_chain 链表中脱链

```c
if (fp->_IO_file_flags & _IO_IS_FILEBUF)
    _IO_un_link ((struct _IO_FILE_plus *) fp);
```

之后会调用_IO_file_close_it 函数，_IO_file_close_it 会调用系统接口 close 关闭文件

```c
if (fp->_IO_file_flags & _IO_IS_FILEBUF)
    status = _IO_file_close_it (fp);
```

最后调用 vtable 中的_IO_FINISH，其对应的是_IO_file_finish 函数，其中会调用 free 函数释放之前分配的 FILE 结构

```c
_IO_FINISH (fp);
```

### printf/puts
printf 和 puts 是常用的输出函数，在 printf 的参数是以'\n'结束的纯字符串时，printf 会被优化为 puts 函数并去除换行符

puts 在源码中实现的函数是_IO_puts，这个函数的操作与 fwrite 的流程大致相同，函数内部同样会调用 vtable 中的_IO_sputn，结果会执行_IO_new_file_xsputn，最后会调用到系统接口 write 函数

printf 的调用栈回溯如下，同样是通过_IO_file_xsputn 实现

```c
vfprintf
	_IO_file_xsputn
		_IO_file_overflow
			funlockfile
				_IO_file_write
					write
```

### <font style="background-color:rgba(255, 255, 255, 0);">常见函数对应指针总结</font>
<font style="background-color:rgba(255, 255, 255, 0);">printf/puts ->_IO_XSPUTN->_IO_OVERFLOW</font>

<font style="background-color:rgba(255, 255, 255, 0);">scanf/gets -> _IO_XSGETN</font>

<font style="background-color:rgba(255, 255, 255, 0);">fwrite -> _IO_XSPUTN->_IO_OVERFLOW</font>

<font style="background-color:rgba(255, 255, 255, 0);">fread -> _IO_XSGETN</font>

<font style="background-color:rgba(255, 255, 255, 0);">fclose -> _IO_FINISH</font>

<font style="background-color:rgba(255, 255, 255, 0);">exit -> _IO_flush_all_lockp ->_IO_OVERFLOW</font>

<font style="background-color:rgba(255, 255, 255, 0);">顺带一提，当我们用printf输出一个以换行符结尾的纯字符串的时候，printf会被优化成puts函数并去除换行符</font>

