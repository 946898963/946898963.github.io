---
title: MeasureSpec
date: 2016-12-02 14:31:49
categories:
- 技术
- Android
- 绘图机制
tags:
- Android
- 绘图机制
---


MeasureSpec，参与了View的measure过程，而且在很大程度上决定了View的尺寸规格。在测量过程中系统通过View的LayoutParams跟父容器所施加的规则（父View的MeasureSpec）转换成相应的MeasureSpec，然后根据这个MeasureSpec来测量出View的宽和高。这个宽和高是测量的宽和高，并不一定等于View的最终的宽和高。

## MeasureSpec

MeasureSpec是一个32位的int值，高2位是SpecMode，代表测量模式，低30位是SpecSize，代表在某种模式下的大小。

```
        private static final int MODE_SHIFT = 30;
        private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

        public static final int UNSPECIFIED = 0 << MODE_SHIFT;
        public static final int EXACTLY     = 1 << MODE_SHIFT;

        
        public static final int AT_MOST     = 2 << MODE_SHIFT;

        public static int makeMeasureSpec(int size, int mode) {
            if (sUseBrokenMakeMeasureSpec) {
                return size + mode;
            } else {
                return (size & ~MODE_MASK) | (mode & MODE_MASK);
            }
        }
        
        public static int makeSafeMeasureSpec(int size, int mode) {
            if (sUseZeroUnspecifiedMeasureSpec && mode == UNSPECIFIED) {
                return 0;
            }
            return makeMeasureSpec(size, mode);
        }

        public static int getMode(int measureSpec) {
            return (measureSpec & MODE_MASK);
        }

        public static int getSize(int measureSpec) {
            return (measureSpec & ~MODE_MASK);
        }
```

MeasureSpec将SpecMode跟SpecSize封装成一个int值，同时提供了封装跟解封装的方法。SpecMode跟SpecSize都是int值，一对SpecMode跟SpecSize可以被封装成一个MeasureSpec，一个MeasureSpec也可以解封装成SpecMode跟SpecSize。

SpecMode有三种，具体如下：
> * UNSPECIFIED   父容器对view的大小没有限制，view可以任意大，用的比较少；
> * AT_MOST       父容器确定了一个值SpecSize，view的大小，不会超过这个值，对应LayoutParams中的wrap_content；
> * EXACTLY       父容器确定了一个精确值SpecSize，这个值就是view的大小，对应LayoutParams中的match_parent跟具体数值两种模式;

## 影响MeasureSpec的因素

系统在测量的过程中通过MeasureSpec确定View大小，正常情况下使用View的父容器确定的MeasureSpec确定View的大小，但是我们仍然可以给View设置LayoutParams。在View测量的时候，系统会将LayoutParams在父容器的约束下转换成对应的MeasureSpec，然后再根据这个MeasureSpec确定View的测量的宽和高。
MeasureSpec不是由LayoutParams唯一确定的，而是由父容器的MeasureSpec跟LayoutParams一起决定的。

**对于顶级View(DecorView)，MeasureSpec的大小是由窗口的尺寸跟其自身的LayoutParams决定的；**
**对于普通的View，MeasureSpec是由父容器的MeasureSpec跟自身的LayoutParams一起决定的；**

确定了MeasureSpec之后，就可以在onMeasure中确定View的测量的宽和高了。

### 源码分析

分析ViewRoot中的源码，观察performTraversals()方法可以发现如下代码：
```
childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);  
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height); 
```
这段代码对应了最外层根View的MeasureSpec的创建过程。这里调用了getRootMeasureSpec()方法去获取widthMeasureSpec和heightMeasureSpec的值，方法中传入的参数，其中lp.width和lp.height在创建ViewGroup实例的时候就被赋值了。

getRootMeasureSpec方法的实现如下：
```
private int getRootMeasureSpec(int windowSize, int rootDimension) {  
    int measureSpec;  
    switch (rootDimension) {  
    case ViewGroup.LayoutParams.MATCH_PARENT:  
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);  
        break;  
    case ViewGroup.LayoutParams.WRAP_CONTENT:  
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);  
        break;  
    default:  
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);  
        break;  
    }  
    return measureSpec;  
}  
```

这里使用了MeasureSpec.makeMeasureSpec()方法来组装一个MeasureSpec，根据LayoutParams中的宽高参数来划分，组装的规则如下：
> * LayoutParams.MATCH_PARENT：精确模式，大小就是窗口的大小；
> * LayoutParams.WRAP_CONTENT：最大模式，大小不确定，但是不能超过窗口的大小；
> * 固定大小：精确模式，大小为LayoutParams参数中指定的大小；

对于普通的View，也就是我们布局中的子View，其Measure过程是由父View传递过来的，ViewGroup中有一个measureChildWithMargins方法，代码如下：
```
 protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```
在这个方法中，会对子View进行Measure，在进行measure之前，会调用getChildMeasureSpec方法，确定子View的MeasureSpec。getChildMeasureSpec实现代码如下：
```
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }

```
getChildMeasureSpec方法的作用就是根据父view的MeasureSpec结合自身的LayoutParams来决定子view的MeasureSpec。参数padding是指父容器中已经占据的空间的大小，因此子元素的大小为父元素的尺寸，减去padding，具体代码如下：
```
int specSize = MeasureSpec.getSize(spec);
int size = Math.max(0, specSize - padding);
```
getChildMeasureSpec的工作原理可以参考一下表格：
![此处输入图片的描述][1]

针对不同的父容器跟LayoutParams，子View可以有多种MeasureSpec。当View采用固定的宽和高的时候，不管父容器的MeasureSpec是什么，View的MeasureSpec都是精确模式，并且大小遵循LayoutParams的大小；当View的宽和高是match_parent的时候，如果父容器的模式是精确模式，那么View也是精确模式，并且其大小是父View剩余空间的大小，如果父View是最大模式，那么子View也是最大模式，并且其大小不会超过父View剩余空间的大小。当View的宽和高是wrap_content的时候，不管父容器的模式是精确还是最大模式View的模式总是最大模式，并且其大小不会超过父View剩余空间的大小。

只要提供了父view的MeasureSpec跟View的LayoutParams，就可以得到子View的MeasureSpec，然后就可以进一步的得到View的宽和高了。

[1]: http://ogts8rw5s.bkt.clouddn.com/measurespec_1.png