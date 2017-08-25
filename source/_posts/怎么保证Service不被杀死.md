---
title: 怎么保证Service不被杀死
date: 2016-06-01 17:09:21

categories:
- 技术
- Android
tags:
- Android
---


避免服务被回收，即使得服务常驻内存，方法很多，如下

> * 将后台服务的内存优化的非常小，避免被回收的几率；
> * 通过商务手段将应用加入各大平台的白名单中；
> * 通过 startForeground将进程设置为前台进程，做前台服务，优先级和前台应用一个级别，除非在系统内存非常缺，否则此进程不会被 kill；
> *  腾讯黑科技，QQ的一个像素常驻法：在应用退到后台后，另起一个只有 1 像素的页面停留在桌面上，让自己保持前台状态，保护自己不被后台清理工具杀死；
> * 双进程Service：让2个进程互相保护，其中一个Service被清理后，另外没被清理的进程可以立即重启进程；
> * 参考：[Android Service 服务不被杀死的妙招][1]；
    参考：[Service服务详解以及如何使service服务不被杀死][3]
> *  开启比如守护进程；
参考：[通过JNI实现守护进程，使Service服务不被杀死 ][2]


[1]: http://www.jb51.net/article/74560.htm
[2]: http://blog.csdn.net/f2006116/article/details/50908855
[3]:http://www.tuicool.com/articles/iu22QnF