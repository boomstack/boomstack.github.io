---
layout: post
title:  "Android ViewStub"
date:   2016-07-30 17:29:45 +0800
categories: Android
---

很多时候我们想要的功能是，一个view在需要它的时候让它显示，不需要它的时候不希望它显示，最常见的思路是设置view的visible属性，gone、invisible或者visible，但这个做法有个缺陷，就是无论设置哪个属性，这个layout文件都会被解析，做了很多无用功。ViewStub就是为了解决这个问题的。

ViewStub继承自View，复写了View的onMeasure方法：
```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(0, 0);
    }
```
长高都是0，也就是说，ViewStub最开始是不存在的。但是ViewStub可以与一个布局进行绑定，“Stub”的意思就是存根嘛，凭着存根就可以找到最终想显示的布局文件，注意，这里说的是布局文件，ViewStub只能静态的显示布局，Java代码构造的view是不能用ViewStub的。

以下一个demo展示了ViewStub的简单使用，在主视图中点击按钮显示一个Button和一个ImageView：

main.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="showStubBtn"
        android:text="showBtn" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="showStubImg"
        android:text="showImg" />

    <ViewStub
        android:id="@+id/stub_btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout="@layout/layout_stub_btn"/>

    <ViewStub
        android:id="@+id/stub_img"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout="@layout/layout_stub_img"/>
</LinearLayout>

```

每个ViewStub“绑定”一个真实的布局文件：

(1)layout_stub_btn.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Button in stub" />
</LinearLayout>
```
(2)layout_stub_img.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/img"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```
MainActivity:
```java
package com.ethan.testviewstub;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.view.ViewStub;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }

    public void showStubBtn(View view) {
        ViewStub stub = (ViewStub) findViewById(R.id.stub_btn);
        if (stub != null) {
            stub.inflate();
        } else {
                    //这里第二个点击按钮会执行,Stub为null
            Toast.makeText(this, "stub is null now ", Toast.LENGTH_LONG).show();
        }
        Button btn = (Button) findViewById(R.id.btn);
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "hola", Toast.LENGTH_SHORT).show();
            }
        });
    }

    public void showStubImg(View view) {
        ViewStub stub = (ViewStub) findViewById(R.id.stub_img);
        stub.inflate();
        ImageView img = (ImageView) findViewById(R.id.img);
        img.setImageResource(R.drawable.ic_launcher);
    }
}
```

好啦，一个简单的例子完成了，可以看出ViewStub的使用场景很狭窄，个人认为只有在一个View被inflate之后不再改动（被解析的布局可以有事件响应）后才考虑使用ViewStub，毕竟拿着存根不如拿着实物好。
