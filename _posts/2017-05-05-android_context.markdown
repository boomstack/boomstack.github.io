---

layout: post 

title: "Android获取上下文几个方法的区别" 

date: 2017-05-05 17:05:56 +0800 

categories: Android

---

先看下继承关系，Activity/Service/Application都是继承自Context的，获取上下文实际获取的是各子类的上下文实例，可能是Activity，也可能是Application等，具体使用哪一个，需要根据当前控件选择，不能随意使用。
![这里写图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/qetrefgsd23423.png?raw=true)

### 1.getContext
 这是View的一个方法，获取视图上下文，view一般是依托于Activity，所以这个方法返回的是当前Activity实例，一般在Activity中可以使用YourActivityName.this代替。
 
### 2.getApplicationContext
 这个是获取整个app生命周期的上下文，一般用于application中，获取app相关的基础信息
### 3.getBaseContext
是ContextWrapper中的方法，基本不用，Google也不推荐使用，是一种委托，在一个context获取另一个context。
### 4.getApplication
这个是获取当前进程的Application实例，可以去操作自己写的Application中的方法。
 