---

layout: post 

title: "Android Service与Activity双向通信的两种方式" 

date: 2017-05-30 20:05:59 +0800 

categories: Android

---

本博客只讲述同一个进程中activity和Service的通信，进程间通信可以使用AIDL，后续博客更新.
## 关于Service的认识
service是一种组件，不是单独的线程或者进程，它属于UI线程，只不过当activity被销毁后还可以继续执行，然后在恰当的时刻被系统回收掉，弥补了activity不好管理线程的缺点，若想让Service长久运行，可以使用前台Service，网易云音乐、墨迹天气的通知栏应该都是前台Service，这个不在本博客描述范围内。
## 方法一：加接口
实际项目写多了慢慢发现了接口的好处，它可以屏蔽两个模块的操作细节，只把功能名称、所携带的参数暴露出来，实现一个接口就是说明当前模块已经实现了某一个功能。那么，如果Service可以提供一个功能，比如说更新进度的功能，把这个功能抽象成一个接口，当activity实现了这个接口的时候，Service就可以向activity传输数据了，即完成了通信。
## 方法二：广播机制
广播也是解耦和的好方法，Service可以在执行任务过程中发送广播，activity接受到广播后就可以根据接受到的数据进行后续操作。
### demo：
代码逻辑比较简单，Service中去执行递增操作，把每次递增的操作反馈给activity  
(1)service
在Service中定义了一个接口，并提供了相关的监听方法，开启一个线程执行耗时操作（Service在UI线程运行，耗时操作必须开线程），然后通过handler向主线程发消息，然后调用接口，把数据传给activity。
```java
package com.boomstack.preparehigh.service;

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.Handler;
import android.os.HandlerThread;
import android.os.IBinder;
import android.os.Message;
import android.support.v4.content.LocalBroadcastManager;

import com.boomstack.preparehigh.HolaPrint;

public class MyService extends Service {
    private OnServiceProgressListener listener;
    private Handler handler;

    public MyService() {
    }

    @Override
    public void onCreate() {
        //使用mainlooper 确保在UI线程执行
        handler = new Handler(getMainLooper()) {
            @Override
            public void handleMessage(Message msg) {
                if (listener != null) {
                    listener.onProgressChanged(msg.what);
                }
            }
        };
    }

    public interface OnServiceProgressListener {
        void onProgressChanged(int progress);
    }

    public void setOnServiceProgressChangedListener(OnServiceProgressListener listener) {
        this.listener = listener;
    }

    /**
     * 递增操作，耗时
     */
    public void increaseNumber() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 1000; i++) {
                    try {
                        handler.sendEmptyMessage(i);
                        HolaPrint.pr("耗时操作： " + i);

                        //广播机制通信
                        Intent intent = new Intent("com.boomstack.preparehigh.service");
                        intent.putExtra("extra_data", String.valueOf(i));
                        LocalBroadcastManager.getInstance(getApplicationContext()).sendBroadcast(intent);
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return new MyBinder();
    }

    public class MyBinder extends Binder {
        public MyService getService() {
            return MyService.this;
        }
    }


}

```
(2)activity
```java
package com.boomstack.preparehigh.service;

import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.ServiceConnection;
import android.os.IBinder;
import android.support.v4.content.LocalBroadcastManager;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;

import com.boomstack.preparehigh.HolaPrint;
import com.boomstack.preparehigh.R;

public class ServiceActivity extends AppCompatActivity implements MyService.OnServiceProgressListener {

    private MyService myService = null;
    private MyReceiver myReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_service);
        myReceiver = new MyReceiver();
        IntentFilter filter = new IntentFilter("com.boomstack.preparehigh.service");
        LocalBroadcastManager.getInstance(getApplicationContext()).registerReceiver(myReceiver, filter);
    }

    public void onBindService(View view) {
        ServiceConnection connection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                myService = ((MyService.MyBinder) service).getService();
                myService.setOnServiceProgressChangedListener(ServiceActivity.this);
            }

            @Override
            public void onServiceDisconnected(ComponentName name) {
                HolaPrint.pr("service disconnected.");
            }
        };
        Intent i = new Intent(this, MyService.class);
        bindService(i, connection, BIND_AUTO_CREATE);
    }

    public void onCommunication(View view) {
        myService.increaseNumber();
    }

    @Override
    public void onProgressChanged(int progress) {
        HolaPrint.pr("value via interface: " + progress);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        LocalBroadcastManager.getInstance(getApplicationContext()).unregisterReceiver(myReceiver);
    }

    /**
     * 广播接收Service消息
     */
    class MyReceiver extends BroadcastReceiver {

        @Override
        public void onReceive(Context context, Intent intent) {
            String value = intent.getStringExtra("extra_data");
            HolaPrint.pr("value via broadcast: " + value);
        }
    }
}

```
注：HolaPrint是我自己写的一个类，用于输出的，不必关心。
执行结果：

![这里写图片描述](https://github.com/boomstack/boomstack.github.io/blob/master/assets/all/nbmndsfas454342.png?raw=true)
