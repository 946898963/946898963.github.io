---
title: Android布局优化-merge
date: 2017-08-25 09:22:39
categories:
- 技术
- Android
tags:
- Android
---


merge的作用减少布局的层级，主要有以下两个应用场景：

1，当Activity的根布局是FrameLayout的时候，可以用merge替代FrameLayout布局，减少布局层级。

2，当一个布局要include进去另一个布局的时候，如果该布局和父布局是相同的布局，那么可以将该布局的根布局设置为merge。


merge的效果是FrameLayout的效果，但是如果将一个根布局是merge的布局include进去另一个布局的时候，merge将被忽略，这个布局的控件将完全被添加到父布局里面，控件展示效果将完全由父布局进行决定。比如，一个merge布局被include进一个FrameLayut，则控件按照FrameLayout的效果进行展示；如果被include进一个LinearLayout布局，则控件按照线性布局的效果进行展示。

使用merge需要注意的事项：
> * merge只能当作根布局使用
> * merge不是view，也不是viewgroup，他相当于声明了一些视图等待被添加
> * 因为merge不是view，所以merge标签设置的属性都是无效的
> * 因为merge标签并不是View，所以在通过LayoutInflate.inflate方法渲染的时候， 第二个参数必须指定一个父容器，且第三个参数必须为true，也是必须为merge下的视图指定一个父亲节点。


自定义view的时候，自定义View的布局文件，根节点如果设置成merge的话，也可以减少布局的层级，如果不在代码中进行属性设置的话，merge布局将按照帧布局的效果进行展示。

自定义view的时候，自定义View如果继承自LinearLayout，建议让自定义View的布局文件根节点设置成merge，这样能少一层结点。不过需要在布局代码中设置布局的属性，例如使用 setOrientation(VERTICAL)设置为垂直效果。如果不在代码中进行设置属性的话merge布局将按照帧布局的效果进行展示。例子参考：[ Android 布局优化之include与merge][7]