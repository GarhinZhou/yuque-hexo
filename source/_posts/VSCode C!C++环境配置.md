---
title: VSCode C/C++环境配置
date: '2025-03-17 15:26:00'
updated: '2025-03-22 11:34:59'
---
整了台笔记本专门用来打比赛，这下得重新配置一下环境了

## 下载编译器
[MinGW下载链接](https://nuwen.net/mingw.html)

![](/images/172acd2119aa141e1a96765fa4b6dcdd.png)

下载完成后进行安装，自己选择安装路径，安装的路径需要记住，马上就要用到。注意路径中不能出现中文！

## 配置环境变量
![](/images/109ccb17f6395648561b2f704a1e14f9.png)

![](/images/c17ce3e801827ca4027a523857bd7491.png)

![](/images/312e5825a7d74eec9715f3aa0192a0ea.png)

![](/images/ab5fabdbb1e4d8c0ee46149869044404.png)

然后我们再来看一看刚刚的操作有没有成功，按Win+R，输入cmd

（注意是 cmd，powershell 是不行的！）

在控制台中输入g++ --version，有版本号出现就可以了

## 安装 VSCode 拓展
搜索C/C++ 安装第一个插件

![](/images/7e80891c7ea5e4d7c030db67261c6658.png)

再搜索安装Code Runner

![](/images/ed3b57843f9c8516a8a4b28d4e05fdf8.png)

## ！配置调试环境
这是配置过程中的重中之重！

首先在一个你希望的位置建一个文件夹，随意起名就可以（注意不可以用中文！），以后的C/C++代码文件都要放在这个文件夹里才可以正常调试。

然后进入VSCode,点击Open Folder或者点击左上角File -> Open Folder，然后打开刚刚建的文件夹，选择信任父级文件夹

点击这个图标新建一个文件夹，命名为`.vscode`（注意必须是这个名字！）

![](/images/17adb89afee4c3564107cf6d1247c8f2.png)

创建完成后再点击这个图标新建四个文件，文件名分别是![](/images/772d7d2b91e69b4812a63109ed92d654.png)

```bash
//c_cpp_properties.json
//launch.json
//settings.json
//tasks.json
```

接下来复制粘贴这四个文件的内容 

首先是`c_cpp_properties.json`

```json
{
  "configurations": [
    {
      "name": "Win64",
      "includePath": ["${workspaceFolder}/**"],
      "defines": ["_DEBUG", "UNICODE", "_UNICODE"],
      "windowsSdkVersion": "10.0.18362.0",
      "compilerPath": "C:/MinGW/bin/g++.exe",
      "cStandard": "c17",
      "cppStandard": "c++17",
      "intelliSenseMode": "gcc-x64"
    }
  ],
  "version": 4
}
```

注意compilerPath这一项要把路径改成刚才g++的安装路径：找到刚刚的安装文件夹->MinGW->bin->g++,exe ,然后复制或者手动把g++.exe的路径敲上去，格式要跟上面代码段一样

然后是`launch.json`

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "(gdb) Launch", 
      "type": "cppdbg", 
      "request": "launch", 
      "program": "${fileDirname}\\${fileBasenameNoExtension}.exe", 
      "args": [], 
      "stopAtEntry": false,
      "cwd": "${workspaceRoot}",
      "environment": [],
      "externalConsole": true, 
      "MIMode": "gdb",
      "miDebuggerPath": "C:\\MinGW\\bin\\gdb.exe",
      "preLaunchTask": "g++",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ]
    }
  ]
}
```

注意miDebuggerPath这一项也要把路径改成刚才g++的安装路径：找到刚刚的安装文件夹->MinGW->bin->gdb,exe ,然后复制或者手动把gdb.exe的路径敲上去，格式要跟上面代码段一样

 接下来是`settings.json`

```json
{
  "files.associations": {
    "*.py": "python",
    "iostream": "cpp",
    "*.tcc": "cpp",
    "string": "cpp",
    "unordered_map": "cpp",
    "vector": "cpp",
    "ostream": "cpp",
    "new": "cpp",
    "typeinfo": "cpp",
    "deque": "cpp",
    "initializer_list": "cpp",
    "iosfwd": "cpp",
    "fstream": "cpp",
    "sstream": "cpp",
    "map": "c",
    "stdio.h": "c",
    "algorithm": "cpp",
    "atomic": "cpp",
    "bit": "cpp",
    "cctype": "cpp",
    "clocale": "cpp",
    "cmath": "cpp",
    "compare": "cpp",
    "concepts": "cpp",
    "cstddef": "cpp",
    "cstdint": "cpp",
    "cstdio": "cpp",
    "cstdlib": "cpp",
    "cstring": "cpp",
    "ctime": "cpp",
    "cwchar": "cpp",
    "exception": "cpp",
    "ios": "cpp",
    "istream": "cpp",
    "iterator": "cpp",
    "limits": "cpp",
    "memory": "cpp",
    "random": "cpp",
    "set": "cpp",
    "stack": "cpp",
    "stdexcept": "cpp",
    "streambuf": "cpp",
    "system_error": "cpp",
    "tuple": "cpp",
    "type_traits": "cpp",
    "utility": "cpp",
    "xfacet": "cpp",
    "xiosbase": "cpp",
    "xlocale": "cpp",
    "xlocinfo": "cpp",
    "xlocnum": "cpp",
    "xmemory": "cpp",
    "xstddef": "cpp",
    "xstring": "cpp",
    "xtr1common": "cpp",
    "xtree": "cpp",
    "xutility": "cpp",
    "stdlib.h": "c",
    "string.h": "c"
  },
  "editor.suggest.snippetsPreventQuickSuggestions": false,
  "aiXcoder.showTrayIcon": true
}
```

 最后是`tasks.json`

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "g++",
      "command": "g++",
      "args": [
        "-g",
        "${file}",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}.exe"
      ],
      "problemMatcher": {
        "owner": "cpp",
        "fileLocation": ["relative", "${workspaceRoot}"],
        "pattern": {
          "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
          "file": 1,
          "line": 2,
          "column": 3,
          "severity": 4,
          "message": 5
        }
      },
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```

 保存这四个文件就配置完成了！



