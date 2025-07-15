---
title: DevEco使用
date: '2025-07-14 23:47:03'
updated: '2025-07-15 15:18:28'
---
<font style="background-color:rgba(255, 255, 255, 0);">代码重构（refacter）：</font>

<font style="background-color:rgba(255, 255, 255, 0);">控件的属性可以重构成一个 extend 函数，从而提高可读性、实现复用</font>

<font style="background-color:rgba(255, 255, 255, 0);">具体操作：选中控件的属性，右键选择 refacter 里面的 extract method</font>

<font style="background-color:rgba(255, 255, 255, 0);">还有别的代码可以通过 refacter 重构</font>

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">代码跳转：</font>**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgba(255, 255, 255, 0);">Ctrl +</font>****<font style="color:rgba(0, 0, 0, 0.9);background-color:rgba(255, 255, 255, 0);"> </font>****<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">鼠标点击</font>**

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">代码格式化：</font>**<font style="color:rgba(0, 0, 0, 0.9);background-color:rgba(255, 255, 255, 0);">Ctrl + Alt + L</font>**

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">代码快速注释：</font>**<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">Ctrl + /</font>**

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">代码结构树：</font>**<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">Alt + 7 / Ctrl + F12</font>**

![](/images/cf0a5be1f03ff1234974c3811ad52051.png)

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">函数注释生成：</font>**<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">/** + Enter</font>**

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">代码查找：</font>

1. <font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">双击 </font>**<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">shift</font>**<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);"> 调出项目级查找</font>
2. **<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">Ctrl + F</font>**<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);"> 调出文件级查找</font>

<font style="color:rgba(0, 0, 0, 0.85);background-color:rgba(255, 255, 255, 0);">Optimize Imports功能：可以快速清除未使用的import，并根据设置的规则对import进行合并或排序</font>



代码自动补全

快速覆写父类

快速生成构造器：

构造器用于初始化类的对象

在类中使用快捷键**Alt+Insert**，或单击鼠标右键选择Generate...，在弹窗中选择Constructor，选择一个或多个需要生成构造函数的参数，点击OK。若选择Select None，则生成不带参数的构造器

![](/images/f1b49bfe2b3920d0a8e852f795506b72.gif)

快速生成get/set方法：

使用 get 和 set 方法可以为类提供更好的封装性，允许对数据进行验证、访问控制、延迟初始化等多种操作，同时为未来的代码维护和扩展提供更多的灵活性

![](/images/4903ee36369bcee8322bd5ac3d93d0fa.gif)f

快速生成声明信息到Index文件：

编辑器支持将HSP和HAR模块中变量、方法、接口、类等需要对外暴露的信息，通过Generate...>Declarations功能，批量在Index.ets文件中进行声明，便于其他模块调用。

在HSP或HAR模块内的文件编辑界面，单击右键选择Generate...>Declarations，或者使用快捷键Alt+Insert，在菜单中选择Declarations，按住快捷键Ctrl并选择需要声明的变量名、方法名、接口名、类名等，即可在模块的Index.ets文件中批量生成相应的声明信息。

> HSP（Harmony Shared Package）和HAR（Harmony Archive）是HarmonyOS中用于代码和资源共享的两种不同类型的模块
>
> HSP - 动态共享包
>
> HSP主要是用于实现代码和资源的共享，但它不支持独立发布。这意味着HSP必须跟随其宿主应用的APP包一起发布，并且与宿主应用同进程，具有相同的包名和生命周期。HSP的优势在于它可以提高代码和资源的可重用性及可维护性。例如，如果多个HAP（Harmony Ability Package）需要共享相同的代码或资源，可以将这些公共部分放入一个HSP中，这样可以有效控制应用包的大小，避免重复打包
>
> HAR - 静态共享包
>
> 相比之下，HAR是一种静态共享包，主要用于在应用内部共享代码、C++库、资源和配置文件，或者在应用发布后供其他应用使用。HAR可以通过Static Library创建，它支持应用内的共享，也可以作为二方库发布到OHPM私仓，供公司内部其他应用使用。HAR的主要用途是在多个模块或工程之间共享ArkUI组件或其他资源
>

## 预览
