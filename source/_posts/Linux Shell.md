---
title: Linux Shell
date: '2024-12-09 23:42:14'
updated: '2025-03-09 00:17:05'
---
## <font style="color:rgb(51, 51, 51);">0x01 重定向和管道</font>
<font style="color:#000000;">重定向命令列表如下：</font>

| <font style="color:#000000;">命令</font> | <font style="color:#000000;">说明</font> |
| --- | --- |
| <font style="color:#000000;">command > file</font> | <font style="color:#000000;">将输出重定向到 file。</font> |
| <font style="color:#000000;">command < file</font> | <font style="color:#000000;">将输入重定向到 file。</font> |
| <font style="color:#000000;">command >> file</font> | <font style="color:#000000;">将输出以追加的方式重定向到 file。</font> |
| <font style="color:#000000;">n > file</font> | <font style="color:#000000;">将文件描述符为 n 的文件重定向到 file。</font> |
| <font style="color:#000000;">n >> file</font> | <font style="color:#000000;">将文件描述符为 n 的文件以追加的方式重定向到 file。</font> |
| <font style="color:#000000;">n >& m</font> | <font style="color:#000000;">将输出文件 m 和 n 合并。</font> |
| <font style="color:#000000;">n <& m</font> | <font style="color:#000000;">将输入文件 m 和 n 合并。</font> |
| <font style="color:#000000;"><< tag</font> | <font style="color:#000000;">将开始标记 tag 和结束标记 tag 之间的内容作为输入。</font> |


> _<font style="color:#000000;background-color:rgb(243, 247, 240);">需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。</font>_
>

管道

以`|`建立管道，将`|`左侧的指令的标准输出接到右侧指令的标准输入

## 0x02 变量
用等号赋值，但两边不能有空格

获取变量值要在变量名前面加上$，`$变量名`只是`${变量名}`的简写，如果上下文有歧义时要使用后者

局部变量必须明确以local声明，否则即使在代码块中也是全局可见的

### <font style="background-color:rgba(255, 255, 255, 0);">环境变量</font>
<font style="background-color:rgba(255, 255, 255, 0);">环境变量是全局变量的一种，动态flag与环境变量相关</font>

<font style="background-color:rgba(255, 255, 255, 0);">相关指令：</font>`<font style="background-color:rgba(255, 255, 255, 0);">export</font>`<font style="background-color:rgba(255, 255, 255, 0);">新增、修改或删除环境变量；</font>`<font style="background-color:rgba(255, 255, 255, 0);">env</font>`<font style="background-color:rgba(255, 255, 255, 0);">设置环境变量；</font>`<font style="background-color:rgba(255, 255, 255, 0);">unset</font>`<font style="background-color:rgba(255, 255, 255, 0);">删除环境变量（函数）</font>

#### 常见环境变量
PATH 外部命令的搜索路径

LOGNAME 当前用户的登录名

HOSTNAME 主机名称

## 0x03 参数
$参数 表示传递的参数

| 引用参数 | 描述 |
| --- | --- |
| 0,1,2... | 参数0相当于是调用本程序的指令，后续分别是第1、2...个参数 |
| * | 用一个字符串把所有参数显示 |
| @ | 从参数1开始按顺序替换 |
| # | 参数数量（不包括参数0） |
| $ | 当前进程ID |
| ！ | 后台运行的最后一个进程的ID |
| ？ | 上一个指令的退出状态（返回值） |
| - | 显示shell的当前选项，和set命令功能一样 |


> `${#变量名}`返回变量值字符串中的字符个数
>

## 特殊文件
`/dev/null` 类似一个只写文件，所有写入它的内容都不可读取

`/dev/zero` 可以产生一个null流（二进制的0流，不是ascii类型），主要用来创建一个指定长度，并且初始化为空的文件

`/dev/tty` 打开时UNIX/Linux会自动将它重定向到当前所处的终端

## 特殊字符、运算符
| ~ | 主目录 |
| --- | --- |
| ` | 命令替换 |
| # | shell脚本的注释 |
| $ | 变量表达式符号 |
| & | 后台作业，置于命令末端让命令在后台执行 |
| * | 字符串通配符 |
| ( | 启动子shell |
| ) | 停止子shell |
| \ | 转义下一个字符 |
| | | 管道 |
| [ | 开始字符集通配符号 |
| ] | 结束字符串通配符号 |
| { | 开始命令块 |
| } | 结束命令块 |
| ; | shell命令分隔符 |
| ' | 强引用 |
| " | 弱引用（其中变量将被替换为对应值） |
| < | 输入重定向 |
| > | 输出重定向 |
| / | 路径名目录分隔符 |
| ? | 单个任意字符（相当于代表单个字符的通配符） |
| ! | 管道行逻辑NOT |


### 字符串运算符
#### 替换运算符
| 变量运算符 | 替换 |
| --- | --- |
| ${变量名:-默认值} | 变量存在且非空则返回变量值，否则返回默认值 |
| ${变量名:=默认值} | 变量存在且非空则返回变量值，否则设定为默认值 |
| ${变量名:?信息} | 变量存在且非空则返回变量值，否则打印信息并退出脚本 |
| ${变量名:+信息} | 变量存在且非空则返回信息，否则返回null |


> 如果省略冒号则将"存在且非空"条件换为"存在"，即只判断变量是否存在。
>

#### 模式匹配运算符
| 变量运算符 | 替换 |
| --- | --- |
| ${变量名#匹配值} | 如果匹配变量值的开头处则删除匹配的最短部分，返回剩余部分 |
| ${变量名##匹配值} | 如果匹配变量值的开头处则删除匹配的最长部分，返回剩余部分 |
| ${变量名%匹配值} | 如果匹配变量值的结尾处则删除匹配的最短部分，返回剩下部分 |
| ${变量名%%匹配值} | 如果匹配变量值的结尾处则删除匹配的最长部分，返回剩下部分 |
| ${变量名/被替换值/替换值}<br/>${变量名//被替换值/替换值} | 1）只有第一个被替换值会被替换<br/>2）变量当中所有被替换值都会被替换 |


## 常用指令
`mv` 移动或重命名文件或目录

`mkdir` 创建一个或多个新的目录

`head` 显示一个文件或者多个文件的前几行或前几个字节

`read` 从标准输入中读取一行

### `grep` 指令
提供了在文本中检索特定字符串的方法

<font style="color:rgb(51, 51, 51);">grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为</font><font style="color:rgb(51, 51, 51);"> </font>**<font style="color:rgb(51, 51, 51);background-color:rgb(236, 234, 230);">-</font>**<font style="color:rgb(51, 51, 51);">，则 grep 指令会从标准输入设备读取数据。</font>

#### <font style="color:rgb(51, 51, 51);">语法</font>
```bash
grep [options] pattern [files]
或
grep [-abcEFGhHilLnqrsvVwxy][-A<显示行数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help][范本样式][文件或目录...]
```

+ <font style="color:rgb(51, 51, 51);">pattern - 表示要查找的字符串或正则表达式。</font>
+ <font style="color:rgb(51, 51, 51);">files - 表示要查找的文件名，可以同时查找多个文件，如果省略 files 参数，则默认从标准输入中读取数据。</font>

**<font style="color:rgb(51, 51, 51);">常用选项：</font>**<font style="color:rgb(51, 51, 51);">：</font>

+ `<font style="color:rgb(51, 51, 51);">-i</font>`<font style="color:rgb(51, 51, 51);">：忽略大小写进行匹配。</font>
+ `<font style="color:rgb(51, 51, 51);">-v</font>`<font style="color:rgb(51, 51, 51);">：反向查找，只打印不匹配的行。</font>
+ `<font style="color:rgb(51, 51, 51);">-n</font>`<font style="color:rgb(51, 51, 51);">：显示匹配行的行号。</font>
+ `<font style="color:rgb(51, 51, 51);">-r</font>`<font style="color:rgb(51, 51, 51);">：递归查找子目录中的文件。</font>
+ `<font style="color:rgb(51, 51, 51);">-l</font>`<font style="color:rgb(51, 51, 51);">：只打印匹配的文件名。</font>
+ `<font style="color:rgb(51, 51, 51);">-c</font>`<font style="color:rgb(51, 51, 51);">：只打印匹配的行数。</font>

**<font style="color:rgb(51, 51, 51);">更多参数说明</font>**<font style="color:rgb(51, 51, 51);">：</font>

+ **<font style="color:rgb(51, 51, 51);">-a 或 --text</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 不要忽略二进制的数据。</font>
+ **<font style="color:rgb(51, 51, 51);">-A<显示行数> 或 --after-context=<显示行数></font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 除了显示符合范本样式的那一列之外，并显示该行之后的内容。</font>
+ **<font style="color:rgb(51, 51, 51);">-b 或 --byte-offset</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 在显示符合样式的那一行之前，标示出该行第一个字符的编号。</font>
+ **<font style="color:rgb(51, 51, 51);">-B<显示行数> 或 --before-context=<显示行数></font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 除了显示符合样式的那一行之外，并显示该行之前的内容。</font>
+ **<font style="color:rgb(51, 51, 51);">-c 或 --count</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 计算符合样式的列数。</font>
+ **<font style="color:rgb(51, 51, 51);">-C<显示行数> 或 --context=<显示行数>或-<显示行数></font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 除了显示符合样式的那一行之外，并显示该行之前后的内容。</font>
+ **<font style="color:rgb(51, 51, 51);">-d <动作> 或 --directories=<动作></font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。</font>
+ **<font style="color:rgb(51, 51, 51);">-e<范本样式> 或 --regexp=<范本样式></font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 指定字符串做为查找文件内容的样式。</font>
+ **<font style="color:rgb(51, 51, 51);">-E 或 --extended-regexp</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 将样式为延伸的正则表达式来使用。</font>
+ **<font style="color:rgb(51, 51, 51);">-f<规则文件> 或 --file=<规则文件></font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。</font>
+ **<font style="color:rgb(51, 51, 51);">-F 或 --fixed-regexp</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 将样式视为固定字符串的列表。</font>
+ **<font style="color:rgb(51, 51, 51);">-G 或 --basic-regexp</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 将样式视为普通的表示法来使用。</font>
+ **<font style="color:rgb(51, 51, 51);">-h 或 --no-filename</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 在显示符合样式的那一行之前，不标示该行所属的文件名称。</font>
+ **<font style="color:rgb(51, 51, 51);">-H 或 --with-filename</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 在显示符合样式的那一行之前，表示该行所属的文件名称。</font>
+ **<font style="color:rgb(51, 51, 51);">-i 或 --ignore-case</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 忽略字符大小写的差别。</font>
+ **<font style="color:rgb(51, 51, 51);">-l 或 --file-with-matches</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 列出文件内容符合指定的样式的文件名称。</font>
+ **<font style="color:rgb(51, 51, 51);">-L 或 --files-without-match</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 列出文件内容不符合指定的样式的文件名称。</font>
+ **<font style="color:rgb(51, 51, 51);">-n 或 --line-number</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 在显示符合样式的那一行之前，标示出该行的列数编号。</font>
+ **<font style="color:rgb(51, 51, 51);">-o 或 --only-matching</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 只显示匹配PATTERN 部分。</font>
+ **<font style="color:rgb(51, 51, 51);">-q 或 --quiet或--silent</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 不显示任何信息。</font>
+ **<font style="color:rgb(51, 51, 51);">-r 或 --recursive</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 此参数的效果和指定"-d recurse"参数相同。</font>
+ **<font style="color:rgb(51, 51, 51);">-s 或 --no-messages</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 不显示错误信息。</font>
+ **<font style="color:rgb(51, 51, 51);">-v 或 --invert-match</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 显示不包含匹配文本的所有行。</font>
+ **<font style="color:rgb(51, 51, 51);">-V 或 --version</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 显示版本信息。</font>
+ **<font style="color:rgb(51, 51, 51);">-w 或 --word-regexp</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 只显示全字符合的列。</font>
+ **<font style="color:rgb(51, 51, 51);">-x --line-regexp</font>**<font style="color:rgb(51, 51, 51);"> </font><font style="color:rgb(51, 51, 51);">: 只显示全列符合的列。</font>
+ **<font style="color:rgb(51, 51, 51);">-y </font>**<font style="color:rgb(51, 51, 51);">: 此参数的效果和指定"-i"参数相同。</font>

#### <font style="color:rgb(51, 51, 51);">实例</font>
<font style="color:rgb(51, 51, 51);">1、在文件 file.txt 中查找字符串 "hello"，并打印匹配的行：</font>

`grep hello file.txt`

<font style="color:rgb(51, 51, 51);">2、在文件夹 dir 中递归查找所有文件中匹配正则表达式 "pattern" 的行，并打印匹配行所在的文件名和行号：</font>

`grep -r -n pattern dir/`

<font style="color:rgb(51, 51, 51);">3、在标准输入中查找字符串 "world"，并只打印匹配的行数：</font>

`echo "hello world" | grep -c world`

<font style="color:rgb(51, 51, 51);">4、在当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行。此时，可以使用如下命令：</font>

`grep test *file`

<font style="color:rgb(51, 51, 51);">结果如下所示：</font>

```plain
$ grep test test* #查找前缀有“test”的文件包含“test”字符串的文件  
testfile1:This a Linux testfile! #列出testfile1 文件中包含test字符的行  
testfile_2:This is a linux testfile! #列出testfile_2 文件中包含test字符的行  
testfile_2:Linux test #列出testfile_2 文件中包含test字符的行
```

<font style="color:rgb(51, 51, 51);">5、以递归的方式查找符合条件的文件。例如，查找指定目录/etc/acpi 及其子目录（如果存在子目录的话）下所有文件中包含字符串"update"的文件，并打印出该字符串所在行的内容，使用的命令为：</font>

`grep -r update /etc/acpi`

<font style="color:rgb(51, 51, 51);">输出结果如下：</font>

```plain
$ grep -r update /etc/acpi #以递归的方式查找“etc/acpi”  
#下包含“update”的文件  
/etc/acpi/ac.d/85-anacron.sh:# (Things like the slocate updatedb cause a lot of IO.)  
Rather than  
/etc/acpi/resume.d/85-anacron.sh:# (Things like the slocate updatedb cause a lot of  
IO.) Rather than  
/etc/acpi/events/thinkpad-cmos:action=/usr/sbin/thinkpad-keys--update
```

<font style="color:rgb(51, 51, 51);">6、反向查找。前面各个例子是查找并打印出符合条件的行，通过"-v"参数可以打印出不符合条件行的内容。</font>

<font style="color:rgb(51, 51, 51);">查找文件名中包含 test 的文件中不包含test 的行，此时，使用的命令为：</font>

`grep -v test *test*`

<font style="color:rgb(51, 51, 51);">结果如下所示：</font>

```plain
$ grep-v test* #查找文件名中包含test 的文件中不包含test 的行  
testfile1:helLinux!  
testfile1:Linis a free Unix-type operating system.  
testfile1:Lin  
testfile_1:HELLO LINUX!  
testfile_1:LINUX IS A FREE UNIX-TYPE OPTERATING SYSTEM.  
testfile_1:THIS IS A LINUX TESTFILE!  
testfile_2:HELLO LINUX!  
testfile_2:Linux is a free unix-type opterating system.
```

