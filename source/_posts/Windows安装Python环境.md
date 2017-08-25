---
title: Windows安装Python环境
date: 2016-12-07 20:34:38
categories:
- 技术
- Python
tags:
- Python
---

Python是跨平台的，它可以运行在Windows、Mac和各种Linux/Unix系统上。在Windows上写Python程序，放到Linux上也是能够运行的。

## 2.x版本还是3.x版本

目前，Python有两个版本，一个是2.x版，一个是3.x版，这两个版本是不兼容的，因为现在Python正在朝着3.x版本进化，在进化过程中，大量的针对2.x版本的代码要修改后才能运行，所以，目前有许多第三方库还暂时无法在3.x上使用。

## 在Windows上安装Python

首先，从Python的[官方网站python.org][1]下载最新的2.7版本。

然后，运行下载的MSI安装包，在选择安装组件的一步时，勾上所有的组件：

install-python-windows

特别要注意选上pip和Add python.exe to Path，然后一路点“Next”即可完成安装。

默认会安装到C:\Python27目录下，然后打开命令提示符窗口，敲入python后，会出现两种情况：

情况一：
![此处输入图片的描述][2]
看到上面的画面，就说明Python安装成功！

你看到提示符>>>就表示我们已经在Python交互式环境中了，可以输入任何Python代码，回车后会立刻得到执行结果。现在，输入exit()并回车，就可以退出Python交互式环境（直接关掉命令行窗口也可以！）。

情况二：得到一个错误：


```
‘python’不是内部或外部命令，也不是可运行的程序或批处理文件。
```

这是因为Windows会根据一个Path的环境变量设定的路径去查找python.exe，如果没找到，就会报错。如果在安装时漏掉了勾选Add python.exe to Path，那就要手动把python.exe所在的路径C:\Python27添加到Path中。


  [1]: https://www.python.org/downloads/
  [2]: http://ogts8rw5s.bkt.clouddn.com/python_1.jpg