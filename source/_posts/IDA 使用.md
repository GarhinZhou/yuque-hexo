---
title: IDA 使用
date: '2024-12-15 13:55:58'
updated: '2025-04-07 21:33:51'
---
[来自先知社区](https://xz.aliyun.com/t/4205?time__1311=n4%2Bxni0QG%3DDQtAKDtD%2F8W4BImxYwNrEqYvPQx)

## 设置
### <font style="color:#000000;">自动添加反汇编注释</font>
<font style="color:#000000;">这个功能对于萌新来说非常友好，在刚刚初学汇编的时候， 难免遇到几个不常用的蛇皮汇编指令，就得自己一个个去查，很麻烦，开启了自动注释的功能后，IDA就可以直接告诉你汇编指令的意思</font>

<font style="color:#000000;">同样是在菜单栏中设置：option-->general</font>

![](/images/a8ed37360c2584317ec5dbbb28e48bb0.png)

<font style="color:#000000;">效果如下：</font>

![](/images/d010c93e9e5a350c050c20e6984496c0.png)

## <font style="color:#000000;">快捷键</font>
<font style="color:#000000;">a：将数据转换为字符串</font>

<font style="color:#000000;">f5：一键</font>[<font style="color:#000000;">反汇编</font>](https://marketing.csdn.net/p/3127db09a98e0723b83b2914d9256174?pId=2782&utm_source=glcblog&spm=1001.2101.3001.7020)

<font style="color:#000000;">esc：回退键，能够倒回上一部操作的视图（只有在反汇编窗口才是这个作用，如果是在其他窗口按下esc，会关闭该窗口）</font>

<font style="color:#000000;">shift+f12：可以打开string窗口，一键找出所有的字符串，右击setup，还能对窗口的属性进行设置</font>

<font style="color:#000000;">ctrl+s：选择某个数据段，直接进行跳转</font>

<font style="color:#000000;">ctrl+鼠标滚轮：能够调节流程视图的大小</font>

<font style="color:#000000;">x(xref)：对着某个函数、变量按该快捷键，可以查看它的交叉引用</font>

<font style="color:#000000;">g：直接跳转到某个地址</font>

<font style="color:#000000;">n：更改变量的名称</font>

<font style="color:#000000;">y：更改变量的类型</font>

<font style="color:#000000;">/ ：在</font>[<font style="color:#000000;">反编译</font>](https://so.csdn.net/so/search?q=%E5%8F%8D%E7%BC%96%E8%AF%91&spm=1001.2101.3001.7020)<font style="color:#000000;">后伪代码的界面中写下注释</font>

<font style="color:#000000;">\：在反编译后伪代码的界面中隐藏/显示变量和函数的类型描述，有时候变量特别多的时候隐藏掉类型描述看起来会轻松很多</font>

<font style="color:#000000;">；：在反汇编后的界面中写下注释</font>

<font style="color:#000000;">ctrl+shift+w：拍摄IDA快照</font>

<font style="color:#000000;">u：undefine，取消定义函数、代码、数据的定义</font>

## <font style="color:#000000;">创建数组</font>
<font style="color:#000000;">在操作IDA的时候，经常会遇到需要创建数组的情况，尤其是为了能方便我们看字符串的时候，创建数组显得非常必要，以下我随便找了个数据来创建数组</font>

<font style="color:#000000;">首先点击选中你想要转换成数组的一块区域：</font>

![](/images/65291cfe5c73a398b581c53f6fd49f67.png)

<font style="color:#000000;">接着在菜单栏中选择：edit-->array，就会弹出如下的选项框</font>

![](/images/bdb65c6bcf15cd0016e74629f0a12c22.png)

<font style="color:#000000;">下面来解释一下各个参数的意思：</font>

`<font style="color:#000000;">Array element size</font>`<font style="color:#000000;"> </font><font style="color:#000000;">这个值表示各数组元素的大小（这里是1个字节），是根据你选中的数据值的大小所决定的</font>

`<font style="color:#000000;">Maximum possible size</font>`<font style="color:#000000;"> </font><font style="color:#000000;">这个值是由自动计算得出的，他表示数组中的元素的可能的最大值</font>

`<font style="color:#000000;">Array size</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示数组元素的数量，一般都根据你选定的自动产生默认值</font>

`<font style="color:#000000;">Items on a line</font>`<font style="color:#000000;"> </font><font style="color:#000000;">这个表示指定每个反汇编行显示的元素数量，它可以减少显示数组所需的空间</font>

`<font style="color:#000000;">Element print width</font>`<font style="color:#000000;"> </font><font style="color:#000000;">这个值用于格式化，当一行显示多个项目时，他控制列宽</font>

`<font style="color:#000000;">Use “dup” construct</font>`<font style="color:#000000;"> </font><font style="color:#000000;">：使用重复结构，这个选项可以使得相同的数据值合并起来，用一个重复说明符组合成一项</font>

`<font style="color:#000000;">Signed elements</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示将数据显示为有符号数还是无符号数</font>

`<font style="color:#000000;">Display indexes</font>`<font style="color:#000000;"> </font><font style="color:#000000;">显示索引，使得数组索引以常规的形式显示，如果选了这个选项，还会启动右边的Indexes选项栏，用于选择索引的显示格式</font>

`<font style="color:#000000;">Create as array</font>`<font style="color:#000000;"> </font><font style="color:#000000;">创建为数组，这个一般默认选上的</font>

<font style="color:#000000;">创建好了以后，就变成了这样：</font>

![](/images/239747de2fdc05dc274cc47a31b94095.png)<font style="color:rgb(51, 51, 51);">  
</font><font style="color:#000000;">)</font>

<font style="color:#000000;">可以看到这些数据已经被当成一个数组折叠到了一起，其中</font>`<font style="color:#000000;">2 dup(0FFh)</font>`<font style="color:#000000;">这样的，表示有两个重复的数据0xff</font>

## <font style="color:#000000;">流程图</font>
### <font style="color:#000000;">折叠流程图中的分支</font>
<font style="color:#000000;">在流程视图中，分支过多的时候，可以在窗口标题处右击选择group nodes，就能把当前块折叠起来</font>

![](/images/1b97ab6cc55f1cecc48d3bd0e9a25c48.png)

<font style="color:#000000;">效果如下:</font>

![](/images/920952c0e5e410d568680e03926aea9e.png)

<font style="color:#000000;">分支块是可以自己命名的，方便自己逆向理解</font>

### <font style="color:#000000;">函数调用图</font>
<font style="color:#000000;">菜单栏中：view-->graphs-->Function calls(快捷键Ctrl+F12)</font>

![](/images/bf753200ab38a014bf823c3d4143d2d1.png)

<font style="color:#000000;">这个图能很清楚地看到函数之间是如何相互调用的</font>

### <font style="color:#000000;">函数流程图</font>
<font style="color:#000000;">菜单栏中：view-->graphs-->flowt chart(快捷键F12)</font>

![](/images/0043f1ab3c3c725e1995bb7fb5d8acde.png)

<font style="color:#000000;">这个其实跟IDA自带的反汇编流程视图差不多，他可以导出来作为单独的一张图</font>

## <font style="color:#000000;">创建结构体：</font>
### <font style="color:#000000;">手工创建结构体</font>
<font style="color:#000000;">创建结构体是在IDA的structures窗口中进行的，这个操作在堆漏洞的pwn题中经常使用</font>

![](/images/3fedf645d49eab9cb3a7180ee357ccce.png)

<font style="color:#000000;">可以看到，这里已经存在了四个结构体，程序本身存在的，可以右击选择hide/unhide,来看具体的结构体的内容</font>

![](/images/2e08bcacea6bd07f1df596a090c07fc1.png)

<font style="color:#000000;">创建结构体的快捷键是：insert</font>

![](/images/8d1272207bf0e1c52c75ecc23bd22353.png)

<font style="color:#000000;">在弹出的窗口中，可以编辑结构体的名字</font>

<font style="color:#000000;">这底下有三个复选框，第一个表示显示在当前结构体之前（就会排列在第一位，否则排列在你鼠标选定的位置），第二个表示是否在窗口中显示新的结构体，第三个表示是否创建联合体。</font>

<font style="color:#000000;">需要注意的是，结构体的大小是它所包含的字段大小的总和，而联合体的大小则等于其中最大字段的大小</font>

<font style="color:#000000;">在单击ok以后，就定好了一个空的结构体：</font>

![](/images/f536293e277cd64a31b702f36a41a1b3.png)

<font style="color:#000000;">将鼠标放在 ends这一行，单击快捷键D即可添加结构体成员，成员的命名默认是以field_x表示的，x代表了该成员在结构体中的偏移</font>

![](/images/102ec8de4f44d5857fd19a2108241d57.png)

<font style="color:#000000;">同时，可以把鼠标放在结构体成员所在的行，按D，就可以切换不同的字节大小</font>

<font style="color:#000000;">默认情况下可供选择的就只有db，dw，dd（1，2，4字节大小）</font>

<font style="color:#000000;">如果想添加型的类型，可以在option-->setup data types(快捷键Alt+D)，进行设置</font>

![](/images/49fc0c5de1db6a6578e023ea585565ca.png)

<font style="color:#000000;">如图，勾选了第五个和第九个的话，就会出现dq和xmmword了（代表了8字节和16字节）</font>

![](/images/f90371a0977eaa56150e96083dc3d7e4.png)

<font style="color:#000000;">如果要添加数组成员则可以对着成员所在的那一行，右击选择array</font>

![](/images/c3a711314ebc507ee104bba601bf0a9f.png)

<font style="color:#000000;">如图，要创建的是16个元素的4字节数组</font>

**<font style="color:#000000;">如果要删除结构体，那么对着结构体按下delete键即可删除</font>**

**<font style="color:#000000;">如果要删除成员，则对着成员按下u（undefine）但是需要注意的是，这里只是删除了成员的名字，而没有删除它所分配的空间</font>**

<font style="color:#000000;">如图，我们删除了中间的field_10的数组成员：</font>

![](/images/91da8b737619b355c405217aac8fb8ac.png)

<font style="color:#000000;">会变成这样：</font>

![](/images/e359ceb0d3974429d59bc4fc73d47d93.png)

<font style="color:#000000;">数组所分配的20个字节的空间并没有被删除，这时如果要删除掉这些空间，就需要在原来数组成员所在的第一行中按下Ctrl+S，删除空间（Edit-->shrink struct types）</font>

<font style="color:#000000;">就可以真正的删除掉成员</font>

**<font style="color:#000000;">给结构体的成员重命名可以用快捷键N</font>**

<font style="color:#000000;">我们在IDA中创建好了结构体以后，就是去应用它了</font>

<font style="color:#000000;">如图，这是一个典型的堆的题目</font>

![](/images/af38543c2b8d37b76f36e8b3b01ed470.png)

<font style="color:#000000;">可以看到v1是一个新建的chunk的地址指针，而后的操作都是往chunk不同的偏移位置写入内容，为了方便我们逆向观察，可以将其变成一个结构体，通过</font>`<font style="color:#000000;">v1</font>`<font style="color:#000000;"> </font>`<font style="color:#000000;">v1+4</font>`<font style="color:#000000;"> </font>`<font style="color:#000000;">v1+0x48</font>`<font style="color:#000000;"> </font><font style="color:#000000;">这样的偏移，创建好结构体后，将</font>`<font style="color:#000000;">char *v1</font>`<font style="color:#000000;">的类型改成</font>`<font style="color:#000000;">mail *v1</font>`<font style="color:#000000;">,（快捷键Y可以更改函数、变量的类型和参数）这个mail是我们创建的结构体的名称，效果如下：</font>

![](/images/c2a681224c58ca609b655ab01e0417ad.png)

### <font style="color:#000000;">导入C语言声明的结构体</font>
<font style="color:#000000;">实际上，IDA有提供一个更方便的创建结构体的方法，就是直接写代码导入</font>

<font style="color:#000000;">在View-->Open Subviews-->Local Types中可以看到本地已有的结构体，在该窗口中右击insert</font>

<font style="color:#000000;">可以添加新的结构体：</font>

![](/images/092deab38c56770a2b981f54034c5b0b.png)

<font style="color:#000000;">这样就导入了新的结构体：</font>

![](/images/47cd001e1925b1e9c7f508b74f9c5f6a.png)

<font style="color:#000000;">但同时我们发现structure视图里面，并没有这个结构体，我们需要对着my_structure右击，选择 synchronize to idb</font>

<font style="color:#000000;">这样structure视图就有了,如图</font>

![](/images/38e58289ccd7dc1f5fc47d24f93630fa.png)

<font style="color:#000000;">这里你会发现，多出来两个db的undefined的成员，这是因为ida默认是会把结构体统一4字节对齐的，满足结构体的大小为0x28</font>

## <font style="color:#000000;">IDA动态调试elf：</font>
<font style="color:#000000;">这里我以一个在Ubuntu虚拟机中的elf为例子，进行调试</font>

<font style="color:#000000;">首先把ida目录中的dbgsrv文件夹中的linux_server64拷贝到Ubuntu的elf的文件夹下，这个elf是64位的所有用的是linux_server64，如果你调试的是32位的程序，你就需要拷贝linux_server</font>

<font style="color:#000000;">记得给他们权限，然后在终端运行，这个程序的作用就像是连接ida和虚拟机中elf的桥梁</font>

![](/images/4e4968d22370c64c22eda896d624d961.png)

<font style="color:#000000;">然后再到ida中进行配置：</font>

<font style="color:#000000;">在菜单栏中选择：debugger-->process options</font>

![](/images/ce6254d50e21ccf861cae23cc5c42f40.png)

<font style="color:#000000;">注意，application和input file 都是填写在虚拟机中的elf的路径，记得要加文件名</font>

<font style="color:#000000;">而directory 填写elf所在目录，不用加文件名</font>

<font style="color:#000000;">hostname是虚拟机的ip地址，port是默认的连接端口</font>

<font style="color:#000000;">parameter和password一般都不用填</font>

<font style="color:#000000;">设置好了以后点击ok</font>

<font style="color:#000000;">接着可以直接在反汇编视图中下断点，只要点击左边的小蓝点即可</font>

![](/images/ae4c915b8b779b087e27b4070f038c7f.png)

<font style="color:#000000;">这时按下快捷键F9，可以直接开始调试</font>

<font style="color:#000000;">按下快捷键F4，则直接运行到断点处停下</font>

![](/images/ac8e9966d5f012da35430ba44d748e3a.png)

<font style="color:#000000;">这个就是基本的各个功能区的介绍，上面是我比较喜欢的常用布局，和ida默认的不太一样，想要自定义添加一些视图的话，可以在debugger-->quick debug view中添加</font>

<font style="color:#000000;">另外可以在Windows-->save desktop来保持当前的视图布局，以后就可以直接加载使用</font>

<font style="color:#000000;">下面介绍一些常用的快捷键</font>

`<font style="color:#000000;">F7</font>`<font style="color:#000000;"> </font><font style="color:#000000;">单步步入，遇到函数，将进入函数代码内部  
</font>`<font style="color:#000000;">F8</font>`<font style="color:#000000;"> </font><font style="color:#000000;">单步步过，执行下一条指令，不进入函数代码内部  
</font>`<font style="color:#000000;">F4</font>`<font style="color:#000000;"> </font><font style="color:#000000;">运行到光标处（断点处）  
</font>`<font style="color:#000000;">F9</font>`<font style="color:#000000;"> </font><font style="color:#000000;">继续运行  
</font>`<font style="color:#000000;">CTRL+F2</font>`<font style="color:#000000;"> </font><font style="color:#000000;">终止一个正在运行的调试进程  
</font>`<font style="color:#000000;">CTRL+F7</font>`<font style="color:#000000;"> </font><font style="color:#000000;">运行至返回,直到遇到RETN（或断点）时才停止.</font>

<font style="color:#000000;">知道了这些快捷键后，调试起来就比较容易了，ida调试有个比较方便的地方在于能直接看到函数的真实地址，下断点也非常直观易操作</font>

## <font style="color:#000000;">IDA-python</font>
<font style="color:#000000;">在IDA的最下面有个不起眼的Output Window的界面，其实是一个终端界面，这里有python终端和IDC终端</font>

![](/images/b00b574774d3ceea6eabf859ed501c33.png)

<font style="color:#000000;">这里的python是2.7的版本，虽然老了点，但已经足够我们用了，在IDA的运用中，我们经常需要计算地址，计算偏移，就可以直接在这个终端界面进行操作，非常方便</font>

---

<font style="color:#000000;">当然上面说的只是很简单的python用法，真正的IDA-python的用法是这样的：</font>

<font style="color:#000000;">这里以简单的一道逆向题来做个例子</font>

![](/images/475b0d322bf3d0ff56deea8c7b48f220.png)

<font style="color:#000000;">这个程序很简单，一开始来个for循环，把judge函数的内容全部异或0xc，这样就导致了程序一运行就会直接破坏掉judge函数</font>

![](/images/53a72559501b3e4759dea25e69eb3f43.png)

<font style="color:#000000;">从而使得没法进行后面的flag判断</font>

<font style="color:#000000;">这里我们就需要写一个脚本来先把被破坏的内容还原，这里IDA提供了两种写脚本操作的方法，一种就是IDC脚本，一种就是python脚本</font>

<font style="color:#000000;">这里只简单的介绍IDA-python</font>

<font style="color:#000000;">而IDA-python通过三个python模块将python代码注入IDA中：</font>

<font style="color:#000000;">idaapi模块负责访问核心IDA API</font>

<font style="color:#000000;">idc模块负责提供IDA中的所有函数功能</font>

<font style="color:#000000;">idautils模块负责提供大量实用函数，其中许多函数可以生成各种数据库相关对象的python列表</font>

<font style="color:#000000;">所有的IDApython脚本会自动导入idc和idautils模块，而idaapi模块得自己去导入</font>

<font style="color:#000000;">这里贴上IDApython的</font>[<font style="color:#000000;">官方函数文档</font>](https://www.hex-rays.com/products/ida/support/idapython_docs/)<font style="color:#000000;">，这里包含了所有函数，值得一看</font>

<font style="color:#000000;">针对以上的题目，我们只需要做一个脚本，指定judg函数的0-181范围的字节异或0xc，即可恢复</font>

```plain
judge=0x600B00
for i in range(182):
    addr=0x600B00+i
    byte=get_bytes(addr,1)#获取指定地址的指定字节数
    byte=ord(byte)^0xC
    patch_byte(addr,byte)#打patch修改字节
```

<font style="color:#000000;">在菜单栏中file-->script file，加载python脚本</font>

<font style="color:#000000;">接着在judge函数中undefined掉原来的函数，在重新生成函数（快捷键p），就可以重新f5了  
</font><font style="color:#000000;">脚本中出现的函数都是已经封装在idc模块中的，具体可查官方文档</font>

![](/images/260d425f9e78f877251c44ee807ad787.png)

<font style="color:#000000;">这只是一个简单的IDApython的使用例子，实际上这个功能非常强大，能弄出非常骚的操作</font>

## <font style="color:#000000;">打PATCH</font>
<font style="color:#000000;">打patch，其实就是给程序打补丁，本质上是修改程序的数据，指令等，这在CTF中的AWD赛制中经常用到，发现程序漏洞后马上就要用这个功能给程序打好patch，防止其他队伍攻击我们的gamebox</font>

<font style="color:#000000;">这里，我是用一个叫keypatch的插件进行操作的，IDA自带的patch功能不太好用</font>

### <font style="color:#000000;">安装keypatch</font>
<font style="color:#000000;">这个很简单，教程在</font>[<font style="color:#000000;">github</font>](https://github.com/keystone-engine/keypatch)<font style="color:#000000;">就有</font>

<font style="color:#000000;">下载Keypatch.py复制到插件目录</font>

**<font style="color:#000000;">IDA 7.0\plugins\Keypatch.py</font>**

<font style="color:#000000;">下载安装keystone python模块，64位系统只需要安装这一个就行</font>

[**<font style="color:#000000;">https://github.com/keystone-engine/keystone/releases/download/0.9.1/keystone-0.9.1-python-win64.msi</font>**](https://github.com/keystone-engine/keystone/releases/download/0.9.1/keystone-0.9.1-python-win64.msi)

<font style="color:#000000;">安装好后，你就会发现这里有个keypatch的选项</font>

![](/images/9cbe2601b2bc9ad33ebec3dc6bc255dd.png)

### <font style="color:#000000;">修改程序指令</font>
<font style="color:#000000;">如果我们要修改程序本身的指令，怎么做呢</font>

![](/images/3ef113def25ba56b73d8004f8ac05a43.png)

<font style="color:#000000;">如图，我们要修改63h这个值</font>

<font style="color:#000000;">将鼠标指向改行，按快捷键Ctrl+Alt+K</font>

![](/images/e1d79ad8ffb0da64ffbd72d58e5e97fb.png)

<font style="color:#000000;">直接输入汇编语句即可修改，打好patch后效果如图：</font>

![](/images/1690535570994db7f8a030439bdc6325.png)

<font style="color:#000000;">这里会生成注释告诉你，这里打过patch，非常人性化</font>

<font style="color:#000000;">接着还要在菜单栏进行设置才能真正使得patch生效</font>

![](/images/bbe6ec72149ad1000eca85b808843310.png)

<font style="color:#000000;">这样一来，原来的程序就已经被修改了</font>

### <font style="color:#000000;">撤销patch</font>
<font style="color:#000000;">如果不小心打错了patch，就可以在这里进行撤销上一次patch的操作了</font>

![](/images/5b77e320619788a1f9dc1d17188aa669.png)

<font style="color:#000000;">但是如果打了很多次patch，不好分清该撤销哪一次的patch，那么可以在菜单栏中打开patched bytes界面</font>

![](/images/1a412515c0804a43f0d5ed4f8b348afb.png)

<font style="color:#000000;">看到所有的patch，要撤销哪一个就右击选择 revert</font>

![](/images/4c143cf4849fa46cfe7e6e1dea57bbf9.png)

## <font style="color:#000000;">IDA导出数据文件</font>
<font style="color:#000000;">在菜单栏中，这里有个选项可以生成各种不同的输出文件</font>

![](/images/c74e019610c73c009869347d9a622d6a.png)

<font style="color:#000000;">这里简单的介绍前两个文件，后面的大家可以自己去生成测试一下用途，我这里就不详细介绍了</font>

<font style="color:#000000;">.map文件描述二进制文件的总体结构，包括与构成改二进制文件的节有关的信息，以及每个节中符号的位置。</font>

<font style="color:#000000;">.asm文件，也就是汇编了，直接能导出ida中反汇编的结果，这个非常实用，有的时候在逆向中经常遇到大量数据加解密的情况，如果在从IDA中一个个慢慢复制可就太没效率了，直接导出生成asm，在里面复制数据快很多</font>

# <font style="color:#000000;">IDA常见命名意义</font>
<font style="color:#000000;">IDA经常会自动生成假名字。他们用于表示子函数，程序地址和数据。根据不同的类型和值假名字有不同前缀</font>

<font style="color:#000000;">sub</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">指令和子函数起点  
</font>__<font style="color:#000000;">locret</font>_<font style="color:#000000;"> </font><font style="color:#000000;">返回指令  
</font><font style="color:#000000;">loc</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">指令  
</font>__<font style="color:#000000;">off</font>_<font style="color:#000000;"> </font><font style="color:#000000;">数据，包含偏移量  
</font><font style="color:#000000;">seg</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">数据，包含段地址值  
</font>__<font style="color:#000000;">asc</font>_<font style="color:#000000;"> </font><font style="color:#000000;">数据，ASCII字符串  
</font><font style="color:#000000;">byte</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">数据，字节（或字节数组）  
</font>__<font style="color:#000000;">word</font>_<font style="color:#000000;"> </font><font style="color:#000000;">数据，16位数据（或字数组）  
</font><font style="color:#000000;">dword</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">数据，32位数据（或双字数组）  
</font>__<font style="color:#000000;">qword</font>_<font style="color:#000000;"> </font><font style="color:#000000;">数据，64位数据（或4字数组）  
</font><font style="color:#000000;">flt</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">浮点数据，32位（或浮点数组）  
</font>__<font style="color:#000000;">dbl</font>_<font style="color:#000000;"> </font><font style="color:#000000;">浮点数，64位（或双精度数组）  
</font><font style="color:#000000;">tbyte</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">浮点数，80位（或扩展精度浮点数）  
</font>__<font style="color:#000000;">stru</font>_<font style="color:#000000;"> </font><font style="color:#000000;">结构体(或结构体数组)  
</font><font style="color:#000000;">algn</font>_<font style="color:#000000;"> </font>__<font style="color:#000000;">对齐指示  
</font>__<font style="color:#000000;">unk</font>_<font style="color:#000000;"> </font><font style="color:#000000;">未处理字节</font>

<font style="color:#000000;">IDA中有常见的说明符号，如db、dw、dd分别代表了1个字节、2个字节、4个字节</font>

# <font style="color:#000000;">IDA反编译报错</font>
<font style="color:#000000;">目前来说， 我遇到的反编译报错的情况，一般是两种</font>

+ <font style="color:#000000;">一是由于程序存在动态加密，导致程序的某些代码段被修改，从而反编译出错，这种情况，就需要去使用IDA-python解密一波，再进行F5反汇编</font>
+ <font style="color:#000000;">二是由于某些玄学问题，直接提示了某个地方出错，一般来说，就按照IDA的提示，去进行修改</font>

<font style="color:#000000;">比如，出现如下报错：</font>

![](/images/9675a136d6a7f75020c2f284ced08b32.png)

<font style="color:#000000;">那我们就去找413238这个地址的地方，提示是说sp指针的值没有被找到，说明是这里出错了，那么就去修改sp的值，修改方法如下：</font>

![](/images/ee2d90a7e18636611a04b7558ddb2a8f.png)

<font style="color:#000000;">也可以使用快捷键 Alt+K</font>

<font style="color:#000000;">有的时候，遇到的这种报错</font>

![](/images/b01f11d696d433abe310faeb2a23cf18.png)

<font style="color:#000000;">就尝试着把报错的地址的汇编语句改一哈，改成nop，就可以解决问题</font>

<font style="color:#000000;">目前来说，我遇到报错的情况不多，一般都可以通过以上方法解决</font>

# <font style="color:#000000;">配置IDA</font>
<font style="color:#000000;">在ida的根目录的cfg文件夹是专门用来存储配置文件的</font>

<font style="color:#000000;">ida的主配置文件为ida.cfg，另外的还有idagui.cfg，idatui.cfg这两个配置文件对应IDA的GUI配置和文本模式的版本</font>

**<font style="color:#000000;">一、ida.cfg</font>**

<font style="color:#000000;">该文件包含了option-->general中的所有选项的配置，可以通过选项中的描述在配置文件总找到相应的选项</font>

<font style="color:#000000;">这里举几个例子：</font>

`<font style="color:#000000;">SHOW_AUTOCOMMENTS</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示是否自动生成汇编指令的注释</font>

`<font style="color:#000000;">GRAPH_SHOW_LINEPREFIXES</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示是否在流程控制视图中显示地址</font>

`<font style="color:#000000;">VPAGESIZE</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示内存调整参数，当处理非常大的输入文件时，IDA可能报告内存不足而无法创建新数据库，在这种情况下增大该参数，重新打开输入文件即可解决问题</font>

`<font style="color:#000000;">OPCODE_BYTES</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示要显示的操作码字节数的默认值</font>

`<font style="color:#000000;">INDENTATION</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示指令缩进的距离</font>

`<font style="color:#000000;">NameChars</font>`<font style="color:#000000;"> </font><font style="color:#000000;">表示IDA支持的变量命令使用的字符集，默认是数字+字母还有几个特殊符号，如果需要添加就改变该参数</font>

**<font style="color:#000000;">二、idagui.cfg</font>**

<font style="color:#000000;">这个文件主要配置默认的GUI行为，键盘的快捷键等，这个很少需要修改，不做过多介绍。感兴趣的可以自己打开该文件观察，并不难懂，改改快捷键还是很容易的</font>

**<font style="color:#000000;">三、idatui.cfg</font>**

<font style="color:#000000;">这个似乎更加不常用，不多说了</font>

<font style="color:#000000;">需要注意的是，以上三个文件是默认配置，也就是说，每次打开创建新的ida数据库的时候，都会以这三个配置文件的设置进行创建，之前临时在菜单栏的设置就会消失，要永久设置ida的配置，就改这三个文件</font>

<font style="color:#000000;">但，凡是都有例外，在option-->font和option-->colors这两个选项是全局选项，修改一次就永久生效的，不用在以上三个配置文件中改</font>

---

