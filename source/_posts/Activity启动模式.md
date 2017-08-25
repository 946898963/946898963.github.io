---
title: Activity启动模式
date: 2017-08-25 09:42:09
categories:
- 技术
- Android
tags:
- Android
---

##Standard

标准启动模式，一个活动设置为标准模式，每次启动Activity的时候，都会在任务栈中创建该Activity的实例。
设置为标准启动模式的Activity位于启动这个Activity的Activity所在的任务栈中。如果启动Activity的组件没有任务栈，启动该Activity的时候需要设置FlAG_Activity_NEW_TASK标识，这样就会给Activity创建新的任务栈。

例：在Service中启动Activity，利用ApplicationContext启动Activity。

##SingleTop

启动SingleTop模式的Activity，如果Activity位于栈顶，不创建新的Activity，只会调用onNewIntent方法；如果不是位于栈顶，则创建新的Activity。

应用场景：接收到消息展示的界面。

##SingleTask

启动SingleTask模式的Activity,如果存在该Activity所需要的任务栈，同时任务栈中存在该Activity，则清空任务栈中该Activity之上的Activity，Activity的onNewIntent方法会被调用，如果不存在该Activity，就创建新的Activity；如果不存在Activity所需要的任务栈，则创建新的任务栈，同时创建新的Activity。

可以在AndroidManifest中通过taskAffinity指定活动所需要的任务栈，默认活动所需要的任务栈是当前应用程序的包名。taskAffinity只有和SingleTask或者allowTaskReparenting配合使用的时候才有意义。

设置为SingleTask模式的Activity其所在的任务栈为taskAffinity所在的任务栈。

任务栈分为前台任务栈和后台任务栈，后台任务栈中的活动都处于Pause状态。如果启动的SingleTask模式的活动已经在一个后台的任务栈中，那么该后台任务栈会一起被切换到前台，同时后台任务栈中该Activity之上的活动都会被清除。

![此处输入图片的描述][1]

应用场景：结束应用，将主Activity设置为SingleTask模式，在onNewItent方法中finish()，退出应用的时候，直接跳转到主Activity。

taskAffinity与allowTaskReparenting结合使用：allowTaskReparenting属性，它的主要作用是activity的迁移，即从一个task迁移到另一个task，这个迁移跟activity的taskAffinity有关。当allowTaskReparenting的值为“true”时，则表示Activity能从启动的Task移动到有着affinity的Task（当这个Task进入到前台时），当allowTaskReparenting的值为“false”，表示它必须呆在启动时呆在的那个Task里。如果这个特性没有被设定，元素(当然也可以作用在每次activity元素上)上的allowTaskReparenting属性的值会应用到Activity上。默认值为“false”。这样说可能还比较难理解，我们举个例子，比如现在有两个应用A和B，A启动了B的一个ActivityC，然后按Home键回到桌面，再单击B应用时，如果此时，allowTaskReparenting的值为“true”，那么这个时候并不会启动B的主Activity，而是直接显示已被应用A启动的ActivityC，我们也可以认为ActivityC从A的任务栈转移到了B的任务栈中。
![此处输入图片的描述][2]

##SingleInstance

增强版的SingleTask,除了会具有SingleTask模式的特点外，启动该模式的Activity的时候，会为Activity创建新的任务栈。

参考：
《Android 第一行代码》
《Android 群英传》
《Android 开发艺术探索》

[Activity启动模式与任务栈(Task)全面深入记录（下）][3]


  [1]: http://ogts8rw5s.bkt.clouddn.com/taskStack.png
  [2]: http://ogts8rw5s.bkt.clouddn.com/allowTaskReparating.png
  [3]: http://blog.csdn.net/javazejian/article/details/52072131