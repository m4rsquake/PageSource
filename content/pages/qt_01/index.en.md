---
title: "Qt QProcess setWorkingDirectory Not Working on Windows System"
description: "Qt QProcess setWorkingDirectory Not Working on Windows System"
weight: 2
draft: false
tags: ["C++", "Qt", "QProcess", "Tutorial"]
---

### Introduction
Recently, while working on a project using Qt, I noticed that even after setting the working directory for QProcess, it still couldn’t find the file, resulting in a failed startup. After researching online and reading the official documentation, I gained a clearer understanding of this issue.

Here’s the summary:

- On Windows, when the QProcess class searches for an external program, it always looks in the current working directory of the parent process, not the directory set by `setWorkingDirectory()`. Only when the external program is launched does the parent process switch to the directory specified by `setWorkingDirectory()`.
- This behavior differs from Unix systems.
- Pay close attention to the official documentation.
### Environment
1. Windows 11
2. Qt Creator 16.0.0
3. Qt 6.8.2 (MSVC 2022, x86_64)
### Preparing the External Program
For testing purposes, I prepared a helloworld.exe program. This is a console application that prints "helloworld!" to the console upon startup and generates a helloworld.txt file in the startup directory (to verify if the working directory takes effect).

I placed this program in the `D:` directory to ensure everything was set up correctly:

{{< figure
    src="qt_01/image1.png"
    alt="Ensure file existence."
    >}}

### Incorrect Usage
Here’s the code (I used `startDetached()` to directly display the output; using `start()` wouldn’t show the program’s output immediately):

```cpp
Copy
QObject *parent = nullptr;
QProcess *process = new QProcess(parent);

// Wrong usage
QString program = "helloworld.exe";
QStringList arguments;
arguments << "run";
process->setWorkingDirectory("D:/"); // Set working directory
process->startDetached(program, arguments);
```

The arguments parameter here is unnecessary; it’s included to demonstrate potential use cases.
When running this code, the Terminal showed no response. This happened because `QProcess` couldn’t find helloworld.exe in the current project directory, causing the external program to fail to launch.

### Correct Usage

```cpp
QObject *parent = nullptr;
QProcess *process = new QProcess(parent);

// Correct usage
QString program = "D:/helloworld.exe"; // Specify the program path so QProcess can locate the external program
QStringList arguments;
arguments << "run";
process->setWorkingDirectory("D:/"); // Set working directory
process->startDetached(program, arguments);
```

Running this code, the Terminal successfully output "helloworld!", and helloworld.txt was generated in the `D:` directory.

{{< figure
    src="qt_01/image2.png"
    alt="Process file output"
    >}}

### Testing the Effect of the `WorkingDirectory` Parameter
Now, let’s remove the line that sets the working directory:

```cpp
QObject *parent = nullptr;
QProcess *process = new QProcess(parent);

// Correct usage
QString program = "D:/helloworld.exe"; // Specify the program path so QProcess can locate the external program
QStringList arguments;
arguments << "run";
// process->setWorkingDirectory("D:/"); // Set working directory (commented out)
process->startDetached(program, arguments);
```

After running this, the Terminal still successfully output "helloworld!", but helloworld.txt was generated in the current project’s running directory.

### Conclusion
When QProcess runs an external program on Windows using a relative path, it can only start from the current working directory of the parent process. Here’s the explanation from the official documentation:

{{< figure
    src="qt_01/image4.png"
    alt="Official documentation"
    >}}

This indicates that the WorkingDirectory parameter does not affect the search for the program during startup. It only comes into play after QProcess locates the application, switching to the specified WorkingDirectory just before launching it.

Is this behavior unique to Windows? According to the official documentation, yes.

That concludes this article. If you spot any issues, please point them out. I welcome any discussion!