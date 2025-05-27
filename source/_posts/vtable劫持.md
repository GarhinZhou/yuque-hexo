---
title: vtable劫持
date: '2025-05-12 17:50:23'
updated: '2025-05-21 15:54:35'
---
Linux 中的一些常见的 IO 操作函数都需要经过 FILE 结构进行处理

尤其是_IO_FILE_plus 结构中存在 vtable，一些函数会取出 vtable 中的指针进行调用

因此伪造 vtable 劫持程序流程的中心思想就是针对_IO_FILE_plus 的 vtable 动手脚，通过把 vtable 指向我们控制的内存，并在其中布置函数指针来实现

vtable 劫持分为两种：

+ 一种是直接改写 vtable 中的函数指针，通过任意地址写就可以实现
+ 另一种是覆盖 vtable 的指针指向我们控制的内存，然后在其中布置函数指针



首先需要知道_IO_FILE_plus 位于哪里，

对于 fopen 的情况下是位于堆内存，对于 stdin\stdout\stderr 是位于 libc.so 中

根据 vtable 在_IO_FILE_plus （也就是 FILE 指针）的偏移可以得到 vtable 的地址

### 直接改写 vtable 中的函数指针
需要搞清楚要劫持的 IO 函数会调用 vtable 中的哪个函数

+ 在 libc2.23 版本往后，位于 libc 数据段的 vtable 是不可以进行写入的。不过，通过在可控的内存中伪造 vtable 的方法依然可以实现利用
+ 在 libc2.23 之前，vtable 是可以写入并且不存在其他检测的

步骤：

1. 先分配一块内存来存放伪造的 vtable
2. 然后修改_IO_FILE_plus 的 vtable 指针指向这块内存
3. 在要劫持的函数对应 vtable 位置写入 system 函数地址
4. 把 "sh" 字符串写入_IO_FILE_plus 头部

因为 vtable 中的指针我们放置的是 system 函数的地址，因此需要传递参数 "/bin/sh" 或 "sh"

因为 vtable 中的函数调用时会把对应的_IO_FILE_plus 指针作为第一个参数传递，因此这里我们把 "sh" 写入_IO_FILE_plus 头部。之后对 fwrite 的调用就会经过我们伪造的 vtable 执行 system("sh")

> 如果程序中不存在 fopen 等函数创建的_IO_FILE 时，也可以选择 stdin\stdout\stderr 等位于 libc.so 中的_IO_FILE，这些流在 printf\scanf 等函数中就会被使用到
>

可以用`p &_IO_2_1_stdin(stdout/stderr)_`来找标准流 FILE 结构体的地址

```c
p/x *(struct _IO_FILE_plus *)_IO_list_all
```

![](/images/e0823805dfd31c911eb898225329d665.png)

后面顺着 chain 又看多了两下，原来这个指令只是展示了`_IO_list_all` 结构体，

没有接着展示其他`_IO_FILE_plus` 结构体，仔细看了一眼，

才发现原来所有`_IO_FILE_plus` 结构体共用一个 vtable



2.24中新增了对vtable指针的检测，检查该地址是否合法：

```c
IO_validate_vtable (const struct _IO_jump_t *vtable)
{
  /* Fast path: The vtable pointer is within the __libc_IO_vtables
     section.  */
  uintptr_t section_length = __stop___libc_IO_vtables - __start___libc_IO_vtables;
  uintptr_t ptr = (uintptr_t) vtable;
  uintptr_t offset = ptr - (uintptr_t) __start___libc_IO_vtables;
  if (__glibc_unlikely (offset >= section_length))
    /* The vtable pointer is not in the expected section.  Use the
       slow path, which will terminate the process if necessary.  */
    _IO_vtable_check ();
  return vtable;
}
```

`IO_validate_vtable` 函数负责检查 vtable 的合法性，会判断 vtable 的地址是不是在一个合法的区间，如果 vtable 的地址不合法，程序将会异常终止

```c
void attribute_hidden
_IO_vtable_check (void)
{
#ifdef SHARED
  /* Honor the compatibility flag.  */
  void (*flag) (void) = atomic_load_relaxed (&IO_accept_foreign_vtables);
#ifdef PTR_DEMANGLE
  PTR_DEMANGLE (flag);
#endif
  if (flag == &_IO_vtable_check)
    return;

  /* In case this libc copy is in a non-default namespace, we always
     need to accept foreign vtables because there is always a
     possibility that FILE * objects are passed across the linking
     boundary.  */
  {
    Dl_info di;
    struct link_map *l;
    if (!rtld_active ()
        || (_dl_addr (_IO_vtable_check, &di, &l, NULL) != 0
            && l->l_ns != LM_ID_BASE))
      return;
  }

#else /* !SHARED */
  /* We cannot perform vtable validation in the static dlopen case
     because FILE * handles might be passed back and forth across the
     boundary.  Therefore, we disable checking in this case.  */
  if (__dlopen != NULL)
    return;
#endif

  __libc_fatal ("Fatal error: glibc detected an invalid stdio handle\n");
}
```

首先检查vtable是否在libc的数据段上，如果不在，则检查其是否在ld等其他模块的合法位置，若否则报错。然而这个检查跳过了`_IO_str_jumps`和`IO_wstr_jumps`这两个与原本vtable结构相同的虚表，则我们可以通过劫持这两个虚表，再修改vtable指针即能绕过检查

## 利用链
+ _IO_str_jumps -> _IO_str_finish
+ _IO_str_jumps -> _IO_str_overflow

### _IO_str_finish
```c
void _IO_str_finish (FILE *fp, int dummy)
{
  if (fp->_IO_buf_base && !(fp->_flags & 1))
    ((void (*)(void))fp + 0xE8 ) (fp->_IO_buf_base); // call qword ptr [fp+E8h]
  fp->_IO_buf_base = NULL;
  _IO_default_finish (fp, 0);
}
```

可以看到这个函数以`fp->_IO_buf_base`为参数执行了`fp+0xE8`处的函数。

需要满足:

```c
fp->_IO_buf_base != 0
fp->_flags为偶数
```

这条链是exit来触发的，所以还需要满足_IO_flush_all_lockp的检查：

```c
fp->_IO_write_ptr > fp-> _IO_write_base
fp-> _mode <= 0
```

所以要构造：

```c
fp->_flag = 0
fp->_IO_write_base = 0
fp->_IO_write_ptr = 1
fp->_IO_buf_base = str_binsh_addr
fp->_mode = 0
fp+0xE8 = system_addr
```

然后将目标文件流的vtable指向_IO_str_jumps-0x8来调用 _IO_str_finish（因为原本要调用的是 _IO_str_overflow，减去0x8即可指向 _IO_str_finish）

### _IO_str_overflow
这个函数比较复杂，不分析了，直接套用其他师傅的结论:

以`2 * (fp->_IO_buf_end - fp->_IO_buf_base) + 100`为参数调用fp+0xE0处的函数。绕过条件需要满足：

```c
fp->_flags & 8 == 0, (fp-> _flags & 0xC00) == 0x400, fp-> _flags & 1 = 0
fp->_IO_write_ptr - fp->_IO_write_base > fp->_IO_buf_end - fp->_IO_buf_base
```

所以我们需要构造

```c
_flags = 0
_IO_write_base = 0
_IO_write_ptr = (binsh_in_libc_addr -100) / 2 +1
_IO_buf_base = 0
_IO_buf_end = (binsh_in_libc_addr -100) / 2 

_mode = -1
fp+0xE0 = system_addr
vtable = _IO_str_jumps - 0x18
```

### 2.28
2.28版本之后上面两个利用链的函数指针被改为free，无法劫持其为system或ogg去实行攻击。2.35的代码为例对比观察一下就能发现问题了：

```c
void
_IO_str_finish (FILE *fp, int dummy)
{
  if (fp->_IO_buf_base && !(fp->_flags & _IO_USER_BUF))
    free (fp->_IO_buf_base);
  fp->_IO_buf_base = NULL;

  _IO_default_finish (fp, 0);
}
```

从此往后的利用需要用到setcontext（2.29-2.31）和house of apple（2.31-2.39）

