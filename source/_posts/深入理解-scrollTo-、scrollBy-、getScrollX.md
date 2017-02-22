title: 深入理解 scrollTo()、scrollBy()、getScrollX()
date: 2015-12-24 09:35:59
categories: Android开发
tags: android
---

## 一、废话先说
   我们在开发Android自定义控件时，尤其是做一些滑动效果时，往往会使用 scrollTo()、scrollBy()、getScrollX() 这几个方法。对初学者来说不太好理解这几个方法，这篇博文就来彻底弄清这几个API的用法。

## 二、测试界面
 我们测试的界面中有三个Linearlayout如下图：黄色框所在的区域为屏幕显示区域
 
![](http://img.blog.csdn.net/20150414083752720)

运行时如下图：
![这里写图片描述](http://img.blog.csdn.net/20150414084317419)
单击按钮会执行相应的方法，并弹当前getScrollX()、getScrollY()的值 

<!--more-->

## 三、详细讲解 

### 1、scrollTo()
   View中的源码如下：
```java
    /**
     * Set the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the x position to scroll to
     * @param y the y position to scroll to
     */
    public void scrollTo(int x, int y) {
        if (mScrollX != x || mScrollY != y) {
            int oldX = mScrollX;
            int oldY = mScrollY;
            mScrollX = x;
            mScrollY = y;
            invalidateParentCaches();
            onScrollChanged(mScrollX, mScrollY, oldX, oldY);
            if (!awakenScrollBars()) {
                postInvalidateOnAnimation();
            }
        }
    }
```
- scrollTo用来设置你的View要滚动的坐标 
- mScrollX、mScrollY 表示当前View在水平和垂直方向上分别滑动了多少
- scrollTo执行后会调用onScrollChanged()方法

我们执行  **scrollTo(100,100)** 过程如下：
![这里写图片描述](http://img.blog.csdn.net/20150414091643430)
执行结果如下：**我们的屏幕向左上方滑动了**
![这里写图片描述](http://img.blog.csdn.net/20150414091815236)

再执行  **scrollTo(-100,-100)** 过程如下：
![这里写图片描述](http://img.blog.csdn.net/20150414094119017)
执行结果如下：**我们的屏幕向右下方滑动了**
![这里写图片描述](http://img.blog.csdn.net/20150414093915469)

是不是很好理解呢，总结下：
- **x>0表示视图(View或ViewGroup)的内容从右向左滑动;反之，从左向右滑动**
- **y>0表示视图(View或ViewGroup)的内容从下向上滑动;反之，从上向下滑动**

### 2、scrollBy()
 View中的源码如下：
```java
    /**
     * Move the scrolled position of your view. This will cause a call to
     * {@link #onScrollChanged(int, int, int, int)} and the view will be
     * invalidated.
     * @param x the amount of pixels to scroll by horizontally
     * @param y the amount of pixels to scroll by vertically
     */
    public void scrollBy(int x, int y) {
        scrollTo(mScrollX + x, mScrollY + y);
    }
```
只是简单调用了 srcollTo(),在原有 mScrollX 、mScrollY 的基础上增量滚动 x、y

我们从刚才scrollTo(-100,-100)基础上scrollBy(50,50)，那么就相当于 scrollTo(50,50),很简单吧。
![这里写图片描述](http://img.blog.csdn.net/20150414095633182)

### 3、getScrollX()； getScrollY()
 View中的源码如下：
```java
/**
     * Return the scrolled left position of this view. This is the left edge of
     * the displayed part of your view. You do not need to draw any pixels
     * farther left, since those are outside of the frame of your view on
     * screen.
     *
     * @return The left edge of the displayed part of your view, in pixels.
     */
    public final int getScrollX() {
        return mScrollX;
    }

    /**
     * Return the scrolled top position of this view. This is the top edge of
     * the displayed part of your view. You do not need to draw any pixels above
     * it, since those are outside of the frame of your view on screen.
     *
     * @return The top edge of the displayed part of your view, in pixels.
     */
    public final int getScrollY() {
        return mScrollY;
    }
```
getScrollX()、getScrollY()返回的就是scrollTo(),scrollBy()中的不断变化的偏移量，我的前面的 Toast也
能体现出来。
  
