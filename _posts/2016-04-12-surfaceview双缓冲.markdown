---

layout: post
title:  "Android SurfaceView双缓冲绘图"
date:   2016-04-12 10:55:25 +0800
categories: Android

---

这篇文章与下一篇关于DialogFragment与Activity通信的博客共同组成了一个Demo，即使用SurfaceView实现自由手绘，功能包括颜色选择、画笔粗细、撤销重做、橡皮擦。源码托管在github，欢迎follow and fork！
源码地址：https://github.com/boomstack/MySurfaceView  

双缓冲其实解决的问题是不加缓冲时的闪烁、卡顿问题，不加缓冲时，每次刷新都要逐个绘制图形，一大堆图形都要短时间内重新绘制当然效率低下，而且对于surfaceview还会有闪烁问题。双缓冲其实就是在内存中建立一个虚拟画布，说是虚拟，但它是真正的画布，只不过不立马在视图中显示，因此不参与屏幕刷新，要等到所有图形绘制完毕后一次性显示出来，只刷新一次。

mCanvas（用于显示的画布）、tmpCanvas、cacheBitmap关系可以用下图表示：
![这里写图片描述](http://img.blog.csdn.net/20160314175316094)
字很难看，见谅。。。

关于双缓冲的例子我感觉这个写的挺好的，http://blog.csdn.net/lee576/article/details/7870160  
我就不再写demo了，在MySurfaceView中已经实现了在Surfaceview中使用双缓冲，clone吧~