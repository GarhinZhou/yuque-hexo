---
title: JDK 配置
date: '2025-02-05 09:14:46'
updated: '2025-03-09 00:17:05'
---
[JDK官网](https://adoptium.net/zh-CN/temurin/releases/)

官网下下来 zip 文件，直接解压不用安装

## Windows 环境
为了让 Java 在命令行中可用，需要配置环境变量

### 设置 `JAVA_HOME` 变量
1. 右键点击 "此电脑" 或 "计算机"，然后选择 属性。
2. 点击左侧的 高级系统设置，然后在弹出的窗口中点击 环境变量。
3. 在 系统变量 部分，点击 新建，添加以下内容：
    - 变量名：`JAVA_HOME`
    - 变量值：JDK 安装目录的路径（例如：`C:\Program Files\Java\jdk-xx`）。
4. 点击 确定。

### 配置 `Path` 环境变量
1. 在环境变量窗口中，找到 Path 变量，选择它并点击 编辑。
2. 在编辑窗口中，点击 新建，然后添加 JDK 的 `bin` 目录路径（例如：`C:\Program Files\Java\jdk-xx\bin`）。
3. 点击 确定 保存更改。

