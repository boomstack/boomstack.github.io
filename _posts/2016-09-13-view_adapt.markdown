---

layout: post 

title: "Android屏幕适配的基本原理" 

date: 2016-09-13 15:23:41 +0800 

categories: Android

---

本篇博客只记述图片的适配，尺寸的适配是在dimens文件中加的，与本篇无关。

为什么要做图片的适配？一张图片有它的分辨率，比如32x32，就是32个像素x32个像素，不同手机的分辨率不同，而我们想让这些图片在不同分辨率的Android设备上显示它原有的像素，这就是原因。

先来看几个变量：
（1）分辨率（resolution）：常说的手机1080 x 1920的值就是分辨率。

（2）像素（px）： pixel，一个点，最基本的单元，一般是正方形的.

（3）像素密度（dpi）：dots per inch，在一个英寸中有多少个像素点，在Android中 ppi=dpi，ppi其实和dpi是不一样的，但是在Android中一样，他们的区别我也没细看，有兴趣可以看看这个博客：https://99designs.com/blog/tips/ppi-vs-dpi-whats-the-difference/   比如我的手机是1080 x 1920的，5英寸（屏幕对角）的手机，所以它的dpi=根号下（1080^2+1920^2）再除以5，约等于440dpi。

（4）密度无关像素（dp）： density-independent pixel，它是一种缩放过的“像素”，跟px关系如下：dp/160=px/dpi, 适配过程中就依靠dp来实现图片的缩放。

（5）像素比（Pixel ratio）：这个没有简写，一般人们也不会提这个概念，Pixel ratio=dpi/160, 我不知道为什么很多人把它叫做“density”，这个很显然是个比例啊，官方并没有把这个比例叫做“density”，详见：https://www.google.com/design/spec/layout/units-measurements.html#units-measurements-designing-layouts-for-dp （我挂着VPN，不知道有没有被墙）

（6）密度（density）：如果你看了别人的博客把density定义为dpi/160, 如果你怕晕，还是不要继续看了。。density=resolution/(width in inch * height in inch)，即每平方英寸的像素数。


适配的时候有很多的drawable文件夹，他们与dpi对应关系如下：
drawable-ldpi (dpi=120, Pixel ratio=0.75)

drawable-mdpi (dpi=160, Pixel ratio=1)

drawable-hdpi (dpi=240, Pixel ratio=1.5)

drawable-xhdpi (dpi=320, Pixel ratio=2)   （平时设计师所谓的“2倍图”就是指的pixel ratio=2）

drawable-xxhdpi (dpi=480, Pixel ratio=3)


看一个实际的例子：
我现在有一个32 x 32的图片要放在项目中，现在主流的手机dpi一般在300~500之间，设计师做的都是“2倍图”，放在xhdpi文件夹下，可能过一段时间后就会出新的“3倍图”放在xxhdpi了，这个是要符合大多数设备，那么，我在定义布局的时候，layout_width和layout_height要写成 16dp（或16dip），因为dp=(px*160)/320,这样，dpi>320的手机会把图片拉伸一些，但是图片清晰度影响较小，dpi<320的手机会缩小图片，性能上也影响不大。比如，在xxhdpi的手机上，图片被放大了3/2=1.5倍，这个计算是这样的：dp被写死，16dp，接着套用公式求px=(dp*dpi)/160=48 , 48/32=1.5