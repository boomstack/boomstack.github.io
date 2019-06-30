---

layout: post 

title: "为什么不建议写死dp" 

date: 2019-04-17 20:10:55 +0800 

categories: Android

---

基础：Android中很熟悉的一个概念：dp (density-independent pixels)，一个dp代表多少实际像素，与设备dpi相关，与px（像素）换算关系：dp/160 = px/dpi。
由换算关系得到px = （dp * dpi）/160, 看上去意思是相同dp情况下，dpi越大，px值越大，即设备像素密度越高，1个dp代表的实际像素也越多。同理，dpi越小，相同dp时，px值也越小，这不正好适合做屏幕适配吗？其实大部分情况下是可以的，为什么我又说不建议写死dp呢？
根本原因我理解是：**手机分辨率与设备dpi关系不统一**
什么意思呢？下面举个例子来看。

```xml
    <Button
        android:id="@+id/button9"
        android:layout_width="360dp"
        android:layout_height="50dp"
        android:layout_marginTop="8dp"
        android:layout_marginBottom="8dp"
        android:text="Button"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/imageView"
        app:layout_constraintVertical_bias="0.741" />
```
简单画了一个Button，宽高都写死，这里只分析宽度值，高度同理。点击按钮会触发以下逻辑：
```java
    private int getDensityDpi() {
        DisplayMetrics dm = new DisplayMetrics();
        Display display = getWindowManager().getDefaultDisplay();
        display.getMetrics(dm);
        Log.i("holaa", "dpi: " + dm.densityDpi);
        Log.i("holaa", "button with(pixels): " + btn.getWidth());
        return dm.densityDpi;
    }
```
即获取设备密度并打印这个Button实际的px值，btn.getWidth()获取的是px值，这一点注释说的很清楚：

```java
    /**
     * Return the width of your view.
     *
     * @return The width of your view, in pixels.
     */
    @ViewDebug.ExportedProperty(category = "layout")
    public final int getWidth() {
        return mRight - mLeft;
    }
```
下边看几台分辨率都是1920x1080手机的执行这段代码情况，屏幕尺寸分别是
1. nexus5: 4.95英寸
2. nexus5X：5.2
3. 红米note3: 5.5英寸
执行结果：  

```shell
#nexus5
I/holaa   ( 2418): dpi: 480
I/holaa   ( 2418): button with(pixels): 1080

#nexus5X
05-31 15:47:22.000  1655  1655 I holaa   : dpi: 420
05-31 15:47:22.000  1655  1655 I holaa   : button with(pixels): 945

#note3
I/holaa   ( 7577): dpi: 480
I/holaa   ( 7577): button with(pixels): 1080
```  

截图：  
1.nexus5:  

![在这里插入图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/20192394023io234o23o3.png?raw=true)  

2.nexus5X:  

![在这里插入图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/2019321947092378.png?raw=true)  

3.红米note3：  

![在这里插入图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/201932423kjlk324232e.png?raw=true)  

根据现象思考几个问题：

### button宽度（px值）是怎么计算出来的？
答案是根据dp和px换算关系
先看nexus5（由于红米note3分辨率和dpi与nexus5一致，不再分析）：
px（btn）= (dp * dpi) /160 = (360 * 480) /160 = 1080px 正好等于屏幕宽度（1080px）
再看nexus5X：
px（btn）= (dp * dpi) /160 = (360 * 420) /160 = 945px 发现与nexus5占用的px值不等了，同时界面上出现较大的差异。

换一种计算思路，nexus5和nexus5X屏幕宽度都是1080px，那么对应换算成dp等于多少？
nexus5：
dp（screen）= (px * 160) / dpi = (1080 * 160) / 480 = 360dp
nexus5X:
dp（screen）= (px * 160) / dpi = (1080 * 160) / 420 = 411dp
那么如果我写死Button宽度是360dp就发现问题了，由于**两部手机宽度代表的dp值不一致，而Button的宽度是一样的，这导致占用屏幕的比例也就不一样**，视觉上当然差别就出来了。

### dpi是谁定义的？
我理解是手机厂商rom定义的。
**网上很多答案都是根据勾股定理，由分辨率和对角线尺寸计算的每英寸像素数，其实计算出来的是ppi，并不是dpi，二者没有任何关系**。ppi是每英寸像素数(pixels per inch)，与设备强相关，而**dpi是每英寸点数(dots per inch)，但是并没说一个点就是一个像素**啊。拿以上三部手机来看：  

**1.nexus5：ppi = (根号下(1920 * 1920 + 1080 * 1080)) / 4.95 = 445ppi; 通过API获取到的dpi = 480  
2.nexus5X：ppi = (根号下(1920 * 1920 + 1080 * 1080)) / 5.2 = 424ppi; 通过API获取到的dpi = 420  
3.note3：ppi = (根号下(1920 * 1920 + 1080 * 1080)) / 5.5 = 401ppi; 通过API获取到的dpi = 480**  

可以看出ppi和dpi没有什么关系，我们平时开发都是用的dpi，**这个值不是计算出来的，而是厂商定义的**，当然厂商们应该会有一套规范吧，但是实际情况是千差万别，碎片化就是这么来的。如果所有厂商规定分辨率与dpi一一对应，而且分辨率规格固定，比如1920x1070是不合法的，其实没有那么多适配问题，就会像iOS一样适配非常简单，但是现实是屏幕千差万别，dpi千差万别。
### 回答问题
由于手机**分辨率与dpi没有直接关系**，一般情况下分辨率越高dpi越大，但没有数学关系在里边，**屏幕宽高的dp是不一定的，写死dp会产生控件相对屏幕比例失衡，进而导致不同型号手机的布局错乱**。
### 如何应对？
其实就是如何**布局适配**问题，**目前最好的方式就是用ConstraintLayout，没有之一**。大杀器就是ConstraintLayout支持百分比布局，以前官方推出过percentlayout，不过很快就废弃了，而ConstraintLayout是支持百分比布局的。百分比布局完美解决适配问题，一切皆可百分比。有了百分比，控件宽高可成比例，控件占用父布局可成比例，间隙可成比例。另外ConstraintLayout综合了LinearLayout、RelativeLayout、FrameLayout诸多优点，是目前最牛掰的布局方式。

以下是官方的解释：
The best way to create a responsive layout for different screen sizes is to use ConstraintLayout as the base layout in your UI. ConstraintLayout allows you to specify the position and size for each view according to spatial relationships with other views in the layout. This way, all the views can move and stretch together as the screen size changes.


但是ConstraintLayout使用起来相对其他布局较为繁琐，容易写bug。