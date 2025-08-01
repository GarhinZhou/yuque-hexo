---
title: C语言文学
date: '2024-12-09 21:08:37'
updated: '2025-03-09 00:17:05'
---
## 0x01 操作字符串的函数
| 函数 | 目的 |
| :--- | :--- |
| **strcpy(s1, s2);** | 复制字符串 s2 到字符串 s1。 |
| **strcat(s1, s2);**  | 连接字符串 s2 到字符串 s1 的末尾。 |
| **strlen(s1);** | 返回字符串 s1 的长度。 |
| **strcmp(s1, s2);**  | 如果 s1 和 s2 是相同的，则返回 0；如果 s1<s2 则返回小于 0；如果 s1>s2 则返回大于 0。 |
| **strncmp(s1, s2, n);** | 用于比较两个字符串的前 n 个字符。 |
| **strchr(s1, ch);**  | 返回一个指针，指向字符串 s1 中字符 ch 的第一次出现的位置。 |
| **strstr(s1, s2);**  | 返回一个指针，指向字符串 s1 中字符串 s2 的第一次出现的位置。 |


`atoi(s)` 是 C 标准库中的一个函数，用于将字符串 s 转换为一个整数。

## 0x02 typedef
`typedef unsigned char BYTE;`定义了一个类型名字（类似C#当中的类）

定义数据类型：只需要在定义时加上struct，  
实际上就是结构体，而在定义结构体的时候不需要加上struct

```plain
typedef struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
} Book;
```



## 0x03 C语言的输入和输出
C 语言把所有的设备都当作文件。（跟linux很像）

所以设备（比如显示器）被处理的方式与文件相同。

以下三个文件会在程序执行时自动打开，以便访问键盘和屏幕。

1、标准输入

2、标准输出

3、标准错误



## 0x04 预处理器
| 指令 | 描述 |
| :--- | :--- |
| #define | 定义宏 |
| #include | 包含一个源代码文件 |
| #undef | 取消已定义的宏 |
| #ifdef | 如果宏已经定义，则返回真 |
| #ifndef | 如果宏没有定义，则返回真 |
| #if | 如果给定条件为真，则编译下面代码 |
| #else | #if 的替代方案 |
| #elif | 如果前面的 #if 给定条件不为真，当前条件为真，则编译下面代码 |
| #endif | 结束一个 #if……#else 条件编译块 |
| #error | 当遇到标准错误时，输出错误消息 |
| #pragma | 使用标准化方法，向编译器发布特殊的命令到编译器中 |


### 预处理器运算符
#### **宏延续运算符（\）**
一行写不下的宏定义可以用宏延续运算符（\）

例如：

```plain
#define  message_for(a, b)  \
    printf(#a " and " #b ": We love you!\n")
```



#### **字符串常量化运算符（#）**
在宏定义中，当需要把一个宏的参数转换为字符串常量时，则使用字符串常量化运算符（#）。

在宏中使用的该运算符有一个特定的参数或参数列表。

```plain
#include <stdio.h>
 
#define  message_for(a, b)  \
    printf(#a " and " #b ": We love you!\n")
 
int main(void)
{
   message_for(Carole, Debra);
   return 0;
}
```

类似函数定义当中的参数传递



#### **标记粘贴运算符（##）**
宏定义内的标记粘贴运算符（##）会合并两个参数。它允许在宏定义中两个独立的标记被合并为一个标记。例如：

##### **实例**
#include <stdio.h>  #define tokenpaster(n) printf ("token" #n " = %d", token##n)  int main(void) {   int token34 = 40;      tokenpaster(34);   return 0; }

当上面的代码被编译和执行时，它会产生下列结果：

```plain
token34 = 40
```

这是怎么发生的，因为这个实例会从编译器产生下列的实际输出：

```plain
printf ("token34 = %d", token34);
```

这个实例演示了 token##n 会连接到 token34 中，在这里，我们使用了**字符串常量化运算符（#）****和****标记粘贴运算符（##）**。

#### **defined() 运算符**
<font style="color:rgb(51, 51, 51);">预处理器 </font>**<font style="color:rgb(51, 51, 51);">defined</font>**<font style="color:rgb(51, 51, 51);"> 运算符是用在常量表达式中的，用来确定一个标识符是否已经使用 #define 定义过。如果指定的标识符已定义，则值为真（非零）。如果指定的标识符未定义，则值为假（零）。下面的实例演示了 defined() 运算符的用法：</font>

##### 实例
```plain
#include <stdio.h> 
#if !defined (MESSAGE)
    #define MESSAGE "You wish!"
#endif  
int main(void) 
{   
    printf("Here is the message: %s\n", MESSAGE);
    return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```plain
Here is the message: You wish!
```

### 参数化的宏
CPP 一个强大的功能是可以使用参数化的宏来模拟函数。例如，下面的代码是计算一个数的平方：

```plain
int square(int x) {
   return x * x;
}
```

我们可以使用宏重写上面的代码，如下：

```plain
#define square(x) ((x) * (x))
```

在使用带有参数的宏之前，必须使用 **#define** 指令定义。参数列表是括在圆括号内，且必须紧跟在宏名称的后边。宏名称和左圆括号之间不允许有空格。例如：

#### 实例
```c
#include <stdio.h>
#define MAX(x,y) ((x) > (y) ? (x) : (y))
int main(void)
{   
    printf("Max between 20 and 10 is %d\n", MAX(10, 20));
    return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```plain
Max between 20 and 10 is 20
```



## 0x05 头文件
### 引用头文件的语法
使用预处理指令 **#include** 可以引用用户和系统头文件。它的形式有以下两种：

```plain
#include <file>
```

这种形式用于引用系统头文件。它在系统目录的标准列表中搜索名为 file 的文件。在编译源代码时，您可以通过 -I 选项把目录前置在该列表前。

```plain
#include "file"
```

这种形式用于引用用户头文件。它在包含当前文件的目录中搜索名为 file 的文件。在编译源代码时，您可以通过 -I 选项把目录前置在该列表前。

### 只引用一次头文件
如果一个头文件被引用两次，编译器会处理两次头文件的内容，这将产生错误。为了防止这种情况，标准的做法是把文件的整个内容放在条件编译语句中，如下：

```c
#ifndef HEADER_FILE
#define HEADER_FILE

the entire header file

#endif
```

这种结构就是通常所说的包装器 **#ifndef**。当再次引用头文件时，条件为假，因为 HEADER_FILE 已定义。此时，预处理器会跳过文件的整个内容，编译器会忽略它。

### 有条件引用
有时需要从多个不同的头文件中选择一个引用到程序中。例如，需要指定在不同的操作系统上使用的配置参数。您可以通过一系列条件来实现这点，如下：

```c
#if SYSTEM_1
   # include "system_1.h"
#elif SYSTEM_2
   # include "system_2.h"
#elif SYSTEM_3
   ...
#endif
```

但是如果头文件比较多的时候，这么做是很不妥当的，预处理器使用宏来定义头文件的名称。这就是所谓的**有条件引用**。它不是用头文件的名称作为 **#include** 的直接参数，您只需要使用宏名称代替即可：

```c
 #define SYSTEM_H "system_1.h"
 ...
 #include SYSTEM_H
```

SYSTEM_H 会扩展，预处理器会查找 system_1.h，就像 **#include** 最初编写的那样。SYSTEM_H 可通过 -D 选项被您的 Makefile 定义。



## 0x06 强制类型转换
### 强制类型转换的语法
```c
(type) expression
```

+ **type**：目标类型，表示你希望将表达式 `expression` 转换成什么类型。
+ **expression**：要转换的值或变量。

补充：浮点数强制转换成整数会变成负数

例子：

```c
double f=0.001;
printf("%d",f);
```



## 0x07 错误处理
C 语言不提供对错误处理的直接支持，但是作为一种系统编程语言，它以返回值的形式允许您访问底层数据。在发生错误时，大多数的 C 或 UNIX 函数调用返回 1 或 NULL，同时会设置一个错误代码 **errno**，该错误代码是全局变量，表示在函数调用期间发生了错误。您可以在 errno.h 头文件中找到各种各样的错误代码。

所以，C 程序员可以通过检查返回值，然后根据返回值决定采取哪种适当的动作。开发人员应该在程序初始化时，把 errno 设置为 0，这是一种良好的编程习惯。0 值表示程序中没有错误。

### errno、perror() 和 strerror()
C 语言提供了 **perror()** 和 **strerror()** 函数来显示与 **errno** 相关的文本消息。

+ **perror()** 函数显示您传给它的字符串，后跟一个冒号、一个空格和当前 errno 值的文本表示形式。
+ **strerror()** 函数，返回一个指针，指针指向当前 errno 值的文本表示形式。

### 程序退出状态
通常情况下，程序成功执行完一个操作正常退出的时候会带有值 EXIT_SUCCESS。在这里，EXIT_SUCCESS 是宏，它被定义为 0。

如果程序中存在一种错误情况，当您退出程序时，会带有状态值 EXIT_FAILURE，被定义为 -1。



## 0x08 可变参数
C 语言为这种情况提供了一个解决方案，它允许您定义一个函数，能根据具体的需求接受可变数量的参数。

声明方式为：

```plain
int func_name(int arg1, ...);
```

其中，省略号 **...** 表示可变参数列表。

下面的实例演示了这种函数的使用：

```c
int func(int, ...) //这里是重点
{   
    ...
}

int main()
{   
    func(2, 2, 3);
    func(3, 2, 3, 4);
}
```

请注意，函数 **func()** 最后一个参数写成省略号，即三个点号（**...**），省略号之前的那个参数是 **int**，代表了要传递的可变参数的总数。为了使用这个功能，您需要使用 **stdarg.h** 头文件，该文件提供了实现可变参数功能的函数和宏。具体步骤如下：

+ 定义一个函数，最后一个参数为省略号，省略号前面可以设置自定义参数。
+ 在函数定义中创建一个 **va_list** 类型变量，该类型是在 stdarg.h 头文件中定义的。
+ 使用 **int** 参数和 **va_start()** 宏来初始化 **va_list** 变量为一个参数列表。宏 **va_start()** 是在 stdarg.h 头文件中定义的。
+ 使用 **va_arg()** 宏和 **va_list** 变量来访问参数列表中的每个项。
+ 使用宏 **va_end()** 来清理赋予 **va_list** 变量的内存。

常用的宏有：

+ `va_start(ap, last_arg)`：初始化可变参数列表。`ap` 是一个 `va_list` 类型的变量，`last_arg` 是最后一个固定参数的名称（也就是可变参数列表之前的参数）。该宏将 `ap` 指向可变参数列表中的第一个参数。
+ `va_arg(ap, type)`：获取可变参数列表中的下一个参数。`ap` 是一个 `va_list` 类型的变量，`type` 是下一个参数的类型。该宏返回类型为 `type` 的值，并将 `ap` 指向下一个参数。
+ `va_end(ap)`：结束可变参数列表的访问。`ap` 是一个 `va_list` 类型的变量。该宏将 `ap` 置为 `NULL`。

## 0x09 指针
& 取地址符

* 取值符

### void *指针
`malloc()`函数返回的是void *类型指针，

void *指针强制转换：`int *pt=(int *)malloc(n*sizeof(int))`



> 注意：`int *p[10]`不等于`int (*p)[10]`，
>
> 前者是指向有十个元素的数组的指针，后者是有十个整数指针的数组
>

### 二级指针
指向指针的指针，可以当作二维数组使用是因为它们在内存中的表示和访问方式相似。具体来说：

1. 内存布局：二维数组在内存中是连续存储的，而二级指针指向的是指针数组，每个指针指向一维数组的起始地址。
2. 访问方式：通过二级指针访问二维数组的元素时，可以使用类似于数组的下标访问方式。例如，`array[i][j]` 可以通过二级指针 ptr 访问为`ptr[i][j]`。

## 0x0A 内存管理
`malloc()`函数和`free()`函数在头文件<stdlib.h>中定义

`mmap()`、`munmap()`函数分别用于内存映射和解映射

`memset()`函数将向传入地址指向的内存写入数据

