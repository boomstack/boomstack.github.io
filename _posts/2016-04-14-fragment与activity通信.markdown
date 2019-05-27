---
layout: post
title:  "Android Fragment与activity通信实践"
date:   2016-04-14 14:52:35 +0800
categories: Android
---
这篇博客与上篇组成一个Surfaceview中自由手绘的demo，源码地址：https://github.com/boomstack/MySurfaceView  
欢迎follow and fork！  

Google推荐使用DialogFragment创建对话框，因为Android系统设计的初衷就是高度组件化，不是我说的，详见api guides第一课：http://developer.android.com/guide/index.html，而fragment本身就是非常灵活的组件，使用fragment是大势所趋。

本博客就在DialogFragment中使用HoloColorpicker这个开源库实现在Dialog中选择颜色并传递给MainActivity，（HoloColorpicker源码地址： https://github.com/LarsWerkman/HoloColorPicker）  

首先跟普通fragment一样，先继承自DialogFragment：  
```java
public class ColorPickerDialogFragment extends DialogFragment {
    ColorPicker picker;//颜色
    int pickedColor;

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        LayoutInflater inflater = getActivity().getLayoutInflater();
        View convertView = inflater.inflate(R.layout.layout_fragment_colorpicker, null);
        picker = (ColorPicker) convertView.findViewById(R.id.picker);
        picker.setOnColorSelectedListener(new ColorPicker.OnColorSelectedListener() {
            @Override
            public void onColorSelected(int color) {
                pickedColor = color;
            }
        });
        builder.setView(convertView).setPositiveButton("OK", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                OnColorChosenListener li = (OnColorChosenListener) getActivity();
                if (pickedColor != 0) {
                    li.onColorChosen(pickedColor);
                }
            }
        }).setNegativeButton("Cancel", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                dialogDismiss();
            }
        });
        return builder.create();
    }


    public void dialogDismiss() {
        this.dismiss();
    }

    public interface OnColorChosenListener {
        void onColorChosen(int color);
    }
}
```

复写了onCreateDialog方法，在里边创建AlertDialog，我们最终要向Activity中传的是选择好的颜色，即pickedColor，可以看到，最后有一个回调接口OnColorChosenListener，在“确定”按钮中调用，在Activity中回调此方法，下面是MainActivity部分代码：
```java
public class MainActivity extends AppCompatActivity implements ColorPickerDialogFragment.OnColorChosenListener {
    private MainSurfaceView sv;
。。。。。。。。。。
 @Override
    public void onColorChosen(int pickedColor) {
        this.pickedColor = pickedColor;
        updateColor();
    }

    public void updateColor() {
        sv.setPaintColor(pickedColor);
    }
```
这样通过回调将Dialog中的pickedColor传入到了MainActivity中去，然后surfaceview实例就可以调用自家的方法了~
这里就可以看到回调的解耦和作用了。