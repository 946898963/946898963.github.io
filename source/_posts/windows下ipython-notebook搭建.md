---
title: windows下ipython notebook搭建
date: 2016-12-14 20:22:20
categories:
- 技术
- Python
tags:
- Python
---

IPython notebook目前已经成为用Python做教学、计算、科研的一个重要工具。本文介绍了Windows环境下，IPython notebook的环境的搭建。

1，首先安装Python环境，可以参考[Windows安装Python环境][1]
2，下载安装Setuptools
下载ez_setup.py到某个目录(如:F:\bigdata\python\python2.7\Scripts)，[下载地址][2]
安装：进入F:\bigdata\python\python2.7\Scripts，运行：
```
python ez_setup.py
```
在这个过程中，系统会连接网络下载所需要的安装包并完成安装，这样就可以使用easy_install了。
3，下载安装pip
下载get-pip.py到某个目录(如:F:\bigdata\python\python2.7\Scripts)，[下载地址][3]
安装：进入F:\bigdata\python\python2.7\Scripts，运行：
```
python get-pip.py
```
在这个过程中，系统会连接网络下载所需要的安装包并完成安装，这样就可以使用easy_install了。
4，下载安装IPthon
运行：
```
easy_install Ipython
```
系统就会去网上寻找ipython包，进行下载及安装，并且还把pyreadline也安装了。
如果用easy_install安装失败，这时候可以尝试用pip来安装，运行：
```
pip install ipython
```
本人在安装过程中用easy_install安装失败了，但是用pip安装成功了。
ipython.exe被安装在F:\bigdata\python\python2.7\Scripts下面，因为前面添加过环境变量的路径支持，所以可以直接输入：
```
ipython
```
5,使用Notebook的话，还需要下载一些其他的东西。
1)下载安装pyzmq，pip对pyzmq支持不太好，装不上。尝试使用easy_install
```
easy_install pyzmq
```
2)下载安装jinja2
```
easy_install jinja2
```
3)下载安装tornado
```
easy_install tornado
```
注意如果安装失败的话，可以尝试用pip安装，我就是用easy_install安装失败，但用pip安装成功了。
好了，使用下面命令就可以把Notebook连起来：
```
ipython notebook
```
如果执行该指令出现如下错误：
[ImportError: No module named notebook.notebookapp][4]
可以通过执行以下命令来解决：
```
pip install jupyter
```
6，尝试科学计算的画图工具matplotlib
因为下载的python没有自带numpy 和 matplotlib
1)下载安装nose
```
easy_install.exe nose
```
2)由于easy_install.exe 与 pip都不能安装numpy，所以从这个[下载地址][5]可以得到
numpy-MKL-1.8.2.win32-py2.7.whl
要注意，因为我们使用的是python2.7，所以一点也要选py2.7的numpy。直接安装，它会依照Windows注册表里面登记的pythonInstall来确定安装路径。
3)最后安装matplotlib，但由于easy_install.exe 与 pip都不能安装matplotlib，所以同上[下载地址][6]可得
matplotlib-1.3.1.win32-py2.7.whl
4)同上[下载地址][7]可得
scipy‑0.14.0.win32‑py2.7.exe
好了，这就大功告成了。


参考链接：
[ windows下面安装跟使用Python，Ipython notebook（详细步骤）][8]
[ windows下ipython notebook搭建][9]


  [1]: http://946898963.github.io/2016/12/07/Windows%E5%AE%89%E8%A3%85Python%E7%8E%AF%E5%A2%83/
  [2]: https://bootstrap.pypa.io/
  [3]: https://bootstrap.pypa.io/
  [4]: http://stackoverflow.com/questions/31401890/importerror-no-module-named-notebook-notebookapp
  [5]: http://www.lfd.uci.edu/~gohlke/pythonlibs/
  [6]: http://www.lfd.uci.edu/~gohlke/pythonlibs/
  [7]: http://www.lfd.uci.edu/~gohlke/pythonlibs/
  [8]: http://www.360doc.com/content/14/0902/11/16740871_406476389.shtml
  [9]: http://blog.csdn.net/u012332571/article/details/38563897