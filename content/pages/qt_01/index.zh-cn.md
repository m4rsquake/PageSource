---
title: "Windows 系统上 Qt QProcess setWorkingDirectory 不起作用"
description: "Windows 系统上 Qt QProcess setWorkingDirectory 不起作用"
weight: 2
draft: false
tags: ["C++", "Qt", "QProcess", "教程"]
---

### 前言

最近在使用 Qt 编写一个项目时，发现 `QProcess` 再设定好工作目录后，仍然找不到文件，启动失败。经过上网查阅与阅读官方文档后，对此有了一个较为清晰的认知。

先说总结：

- Windows 平台下，QProcess 类在查找外部程序时，始终在父进程的当前工作目录中查找，而不是在 `setWorkingDirectory` 所设置的目录中查找。只有在启动外部程序时，父进程才会从设置的工作目录查找。
- 该特性与 Unix 系统下不同。
- 仔细看官方文档。

### 运行环境

1. Windows 11
2. Qt Creator 16.0.0Qt 
3. Qt 6.8.2 (MSVC 2022, x86_64) 

### 准备外部程序

为了进行测试，我准备了一份 helloworld.exe 程序。该程序是一个控制台程序，启动时在控制台打印 helloworld! 字符串。并在启动目录输出 helloworld.txt 文件（用于测试工作目录是否生效）。

我将该程序放在了 `D:` 目录下，确保无误：

{{< figure
    src="qt_01/image1.png"
    alt="确保程序存在"
    >}}

### 错误用法

代码如下（为了直接显示运行结果，我使用了 `startDetached()` ，使用 `start()` 启动程序程序不会直接显示）：

```cpp
QObject *parent = nullptr;
QProcess *process = new QProcess(parent);

//wrong usage
QString program = "helloworld.exe";
QStringList arguments;
arguments << "run";
process->setWorkingDirectory("D:/"); // 设置工作目录
process->startDetached(program, arguments);
```

- 此处 `arguments` 参数是不必要的，目的是为了展示在使用时可能的需求。

运行该代码，Terminal 没有任何反应，原因是 QProcess 在当前项目目录找不到 helloworld.exe，所以启动外部程序失败了。

### 正确用法

```cpp
QObject *parent = nullptr;
QProcess *process = new QProcess(parent);

//corrcet usage
QString program = "D:/helloworld.exe"; //指定程序路径，使得 QProcess 能够找到该外部程序
QStringList arguments;
arguments << "run";
process->setWorkingDirectory("D:/"); // 设置工作目录
process->startDetached(program, arguments);
```

运行该代码，Terminal 成功输出了 “helloworld!”，且 helloworld.txt 被生成到了 `D:` 。

{{< figure
    src="qt_01/image2.png"
    alt="程序文件输出"
    >}}

### 测试 WorkingDirectory 参数作用

去除设定工作目录所在行：

```cpp
QObject *parent = nullptr;
QProcess *process = new QProcess(parent);

//corrcet usage
QString program = "D:/helloworld.exe"; //指定程序路径，使得 QProcess 能够找到该外部程序
QStringList arguments;
arguments << "run";
// process->setWorkingDirectory("D:/"); // 设置工作目录
process->startDetached(program, arguments);
```

运行后，Terminal 同样成功输出了 “helloworld!”，但 helloworld.txt 在当前项目运行目录生成。

### 结论

QProcess 在 Windows 系统上运行外部程序时，若采用相对路径，其起始目录只能是父进程当前的当前工作目录，附上官方文档的解释：

{{< figure
    src="qt_01/image3.png"
    alt="官方文档"
    >}}

这说明，启动程序时，WorkingDirectory 参数并不能起到查找程序的作用，它只在 QProcess 找到应用程序后即将启动时，再切换至 WorkingDirectory 完成启动。

该特性是否只属于 Windows 平台？ 根据官方文档所述，是的。

以上就是本文全部内容，有问题还请指出，欢迎讨论。
