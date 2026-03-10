---
title: exit_hook劫持
date: '2025-09-25 15:32:01'
updated: '2025-12-19 14:58:52'
---
来自：[https://www.cnblogs.com/bhxdn/p/14222558.html](https://www.cnblogs.com/bhxdn/p/14222558.html)

![](/images/d9fb334cb8b27de7d791ad08788fd5f1.png)

这里可以看到是执行了__run_exit_handlers。进入这个函数，看看它调用了哪些函数。

![](/images/9ebf2f85c4375c50999f60de4a486c82.png)

这里显示其中调用的一个函数，是_dl_fini。这里为了方便看，我们直接看_dl_fini的关键源码。

```cpp
#ifdef SHARED
  int do_audit = 0;
 again:
#endif
  for (Lmid_t ns = GL(dl_nns) - 1; ns >= 0; --ns)
    {
      /* Protect against concurrent loads and unloads.  */
      __rtld_lock_lock_recursive (GL(dl_load_lock));

      unsigned int nloaded = GL(dl_ns)[ns]._ns_nloaded;
      /* No need to do anything for empty namespaces or those used for
     auditing DSOs.  */
if (nloaded == 0
#ifdef SHARED
      || GL(dl_ns)[ns]._ns_loaded->l_auditing != do_audit
#endif
      )
    __rtld_lock_unlock_recursive (GL(dl_load_lock));
```

看8行和18行，发现是调用了 __rtld_lock_lock_recursive 和 __rtld_lock_unlock_recursive 。

这里我们看一下这两个函数在哪。

![](/images/f95103c897a760f120583ac369ab8110.png)

![](/images/a72d9c02f421cad4920c38329749e44f.png)

这两个函数在_rtld_global结构体里面。只要我们将其中一个指向one_gadgets，在CTF中，就可以拿到shell了。

![](/images/4863b218a5eaebcd0d94417e341f1d40.png)

接下来就是计算偏移了，发现在64位libc-2.23中，这两个指针在结构体中的偏移分别是3848和3856。为了方便，我们可以直接记住这两个指针和libc的基地址之间的距离。 

在libc-2.23中

```cpp
exit_hook = libc_base+0x5f0040+3848
exit_hook = libc_base+0x5f0040+3856
```

在libc-2.27中

```cpp
exit_hook = libc_base+0x619060+3840
exit_hook = libc_base+0x619060+3848
```

这样一来，只要知道libc版本和任意地址的写，我们可以直接写这个指针，执行exit后就可以拿到shell了。（其实不用非要执行exit，就程序正常返回也可以执行到这里）



但是在 glibc2.34 之后__rtld_lock_lock_recursive 和 __rtld_lock_unlock_recursive就不复存在了，所以这个利用手法适用于 2.34 及之前的 libc 版本

![](/images/10047d338ebcffdf6ea7bfa4e61d0889.png)

![](/images/2e704afd6a43f74150b496dbc9f45585.png)

一次任意写：

修改`__rtld_lock_lock_recursive` 或者 `__rtld_lock_unlock_recursive` 为 one_gadget

两次任意写：

一次修改`dl_load_lock`控制 rdi

第二次修改`__rtld_lock_lock_recursive` 或者 `__rtld_lock_unlock_recursive` 为 system

