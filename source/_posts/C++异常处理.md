---
title: C++异常处理
date: '2025-09-16 15:11:07'
updated: '2025-09-20 17:23:50'
---
主要就是利用 try 块当中的代码出现错误时会直接截断函数执行然后进行栈展开（立刻沿着函数调用链回溯查找 catch 块，所以后面的 canary 检查没有执行），从而绕过原本的安全检查机制比如 canary

关键在于异常处理时会截断执行，从而执行异常处理函数的`leave;ret`，实现栈迁移

## 调用 throw 函数的上一级函数中有对应 catch 块
```python
void main(){
    try{
        vuln();
    }
    catch(const char *s){
        printf(s)
    }
}

void vuln(){
    char s[10];
    int len;
    len = read(0,s,100);
    if (len > 10){
        throw "Buffer overflow.";
    }
}
```

这种利用方式只有在报错 throw 函数的上一级函数（调用函数）中有 try 和 catch 块时可以实现，因为如果再沿着调用链往回溯就不会用到这个函数的 rbp

在调用 throw 的上一级函数当中有<font style="background-color:#FBDE28;">对应错误</font>的 catch 块，即错误会被第一时间抓取

```python
poc1 = padding + p64(addr-0x8) #实现栈迁移
```

## 没有对应 catch 块
```python
void main(){
    vuln();
    printf(s);
}

void vuln(){
    char s[10];
    int len;
    len = read(0,s,100);
    if (len > 10){
        throw "Buffer overflow.";
    }
}

void backdoor(){
    try{
        printf("nononono!");
    }
    catch(const char *s){
        system("/bin/sh");
    }
}
```

在上一级函数没有对应 catch 块的时候可以利用别的函数的对应的 catch 块，将该函数 try 块地址覆盖函数返回地址从而伪造上一级函数有对应 catch 块，实现调用到 catch 块，还有当中的 `leave;ret`

```python
poc2 = padding + p64(addr-0x8) + p64(try_addr) #将try块地址覆盖返回地址
```

用 try 块区间的地址覆盖返回地址就可以伪造出当前函数的栈帧为含有对应 catch 块的函数栈帧了

