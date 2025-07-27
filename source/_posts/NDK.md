---
title: NDK
date: '2025-07-23 11:21:43'
updated: '2025-07-23 16:13:33'
---
<font style="color:rgba(0, 0, 0, 0.9);">N</font><font style="color:rgba(0, 0, 0, 0.9);">DK（Native Development Kit）是HarmonyOS SDK提供的Native API、相应编译脚本和编译工具链的集合，方便开发者使用C或C++语言实现应用的关键功能。NDK只覆盖了HarmonyOS一些基础的底层能力，如C运行时基础库libc、图形库、窗口系统、多媒体、压缩库、面向ArkTS/JS与C跨语言的Node-API等，并没有提供ArkTS/JS API的完整能力</font>

## <font style="color:rgba(0, 0, 0, 0.9);">NDK常用模块</font>
<font style="color:rgba(0, 0, 0, 0.9);">下表介绍了NDK的常用模块</font>

| **<font style="color:rgba(0, 0, 0, 0.9);">模块</font>** | **<font style="color:rgba(0, 0, 0, 0.9);">模块简介</font>** |
| --- | --- |
| <font style="color:rgba(0, 0, 0, 0.9);">标准C库</font> | <font style="color:rgba(0, 0, 0, 0.9);">以musl为基础提供的标准C库接口。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">标准C++库</font> | <font style="color:rgba(0, 0, 0, 0.9);">C++运行时库libc++_shared。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">日志</font> | <font style="color:rgba(0, 0, 0, 0.9);">打印日志到系统的HiLog接口。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">Node-API</font> | <font style="color:rgba(0, 0, 0, 0.9);">当需要实现ArkTS/JS和C/C++之间的交互时，可以使用Node-API。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">FFRT</font> | <font style="color:rgba(0, 0, 0, 0.9);">基于任务的并发编程框架。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">libuv</font> | <font style="color:rgba(0, 0, 0, 0.9);">三方异步IO库。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">zlib</font> | <font style="color:rgba(0, 0, 0, 0.9);">zlib库，提供基本的数据压缩、解压接口。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">Rawfile</font> | <font style="color:rgba(0, 0, 0, 0.9);">应用资源访问接口，可以读取应用中打包的各种资源。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">XComponent</font> | <font style="color:rgba(0, 0, 0, 0.9);">ArkUI XComponent组件提供surface与触屏事件等接口，方便开发者开发高性能图形应用。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">Drawing</font> | <font style="color:rgba(0, 0, 0, 0.9);">系统提供的2D图形库，可以在surface进行绘制。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">OpenGL</font> | <font style="color:rgba(0, 0, 0, 0.9);">系统提供的OpenGL 3D图形接口。</font> |
| <font style="color:rgba(0, 0, 0, 0.9);">OpenSL ES</font> | <font style="color:rgba(0, 0, 0, 0.9);">用于2D、3D音频加速的接口库。</font> |


<font style="color:rgb(0, 0, 0);">  
</font>

