# subprocess
Python 标准库里的 `subprocess` 模块，用它来**启动和控制外部进程/系统命令**。

你可以把它理解成：

> 让 Python 去调用终端命令，或者启动别的程序。

## 最常见的用途

比如你想在 Python 里执行：

ls  
pwd  
git status  
python other.py

就会用到 `subprocess`。

## 一个最简单的例子

import subprocess  
  
result = subprocess.run(["ls", "-l"])

这行代码会执行终端命令：

ls -l