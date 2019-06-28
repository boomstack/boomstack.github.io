---

layout: post 

title: "使用rxjava实现点击防抖动" 

date: 2017-06-16 18:41:13 +0800 

categories: Android

---

开发中经常遇到这种点击按钮会响应两次的情况，原因就是点击一次没反应又点击了一次，包括微信的发现页、朋友圈点击都是这样，连续点两次就弹出两次页面，这不是什么大问题，但是对于点击之后立马要处理逻辑的事件就可能有问题。还是解决一下
### 传统解决方法
获取系统时间，第一次可以点击，后续要加上时间间隔判断，大于设定的时间间隔再执行点击，很简洁。
```java
public abstract class OnMultiClickListener implements View.OnClickListener {

    public static final int MIN_CLICK_DELAY_TIME = 1000;
    private long lastClickTime = 0;

    @Override
    public void onClick(View v) {
        long currentTime = Calendar.getInstance().getTimeInMillis();
        if (currentTime - lastClickTime > MIN_CLICK_DELAY_TIME) {
            lastClickTime = currentTime;
            onMultiClick(v);
        }
    }

    public abstract void onMultiClick(View v);
}
```
### rxjava方式
最近项目都是在用rxjava，还不是很会，多写写demo吧。直接写了个util类，以后还会扩展别的。这里用订阅的方式，包装了一下点击事件（rxjava2.x把create废弃了，这里用的1.x）
```java
package com.boomstack.rxlearn;

import android.support.annotation.NonNull;
import android.view.View;

import rx.Observable;
import rx.Subscriber;

/**
 * Created by wangkangfei on 17/4/21.
 */

public class RxUtils {

    public static Observable<Void> clickView(@NonNull View view) {
        checkNoNull(view);
        return Observable.create(new ViewClickOnSubscribe(view));
    }

    /**
     * 查空
     */
    private static <T> void checkNoNull(T value) {
        if (value == null) {
            throw new NullPointerException("generic value here is null");
        }
    }

    private static class ViewClickOnSubscribe implements Observable.OnSubscribe<Void> {
        private View view;

        public ViewClickOnSubscribe(View view) {
            this.view = view;
        }

        @Override
        public void call(final Subscriber<? super Void> subscriber) {
            View.OnClickListener onClickListener = new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    //订阅没取消
                    if (!subscriber.isUnsubscribed()) {
                        //发送消息
                        subscriber.onNext(null);
                    }
                }
            };
            view.setOnClickListener(onClickListener);
        }
    }
}

```
具体使用：
```java
RxUtils.clickView(btnClick/*your view*/)
                .throttleFirst(1000, TimeUnit.MILLISECONDS)
                .subscribe(new Action1<Void>() {
                    @Override
                    public void call(Void aVoid) {
                        Toast.makeText(getActivity(), "rx click triggered", Toast.LENGTH_SHORT).show();
                    }
                });
```
主要是使用的rxjava的throttleFirst操作符，它只会放出每个时间段内的第一个对象：
![这里写图片描述](http://img.blog.csdn.net/20170421144317322?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZXRoYW5ob2xh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
所以中间的点击事件是被忽略掉的，也就实现了防抖动。