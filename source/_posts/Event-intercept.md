---
title: 事件拦截机制
date: 2016-09-12 14:42:00
tags: android
---

要了解事件拦截，首先要了解触摸事件，触摸事件是捕获触摸屏幕后发生的事件。按一下屏幕通常会有几个事件发生，当按下屏幕，这是事件1。滑动了一下，这是事件2。当手抬起，这是事件3。当重写onTouchEvent方法时，会给我们一个事件封装类MotionEvent。滑动，按下，对应不同的Action(如MotionEvent.ACTION_DOWN,MotionEvent.ACTION_UP)，通过对Action的判断就可以实现不同的逻辑了。  

 咋一看触摸事件好像比较简单，但Android的View是树形结构的，一个View可能放在一个ViewGroup里面，而一个ViewGrop可能又放在另一个ViewGroup里面，可能会存在多层的嵌套结构，那么里面的触摸事件要给谁处理呢？这就要用到事件拦截了。

先附上代码。
MyViewGroupA.java
 ```   
public class MyViewGroupA extends LinearLayout {
    public MyViewGroupA(Context context) {
        super(context);
    }
    public MyViewGroupA(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public MyViewGroupA(Context context, AttributeSet attrs,
                        int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d("LOG", "ViewGroupA dispatchTouchEvent" + ev.getAction());
        return super.dispatchTouchEvent(ev);
    }
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d("LOG", "ViewGroupA onInterceptTouchEvent" + ev.getAction());
        return super.onInterceptTouchEvent(ev);
    }
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("LOG", "ViewGroupA onTouchEvent" + event.getAction());
        return super.onTouchEvent(event);
    }
}
 ```


MyViewGroupB.java
 ```  
public class MyViewGroupB  extends LinearLayout {
    public MyViewGroupB(Context context) {
        super(context);
    }
    public MyViewGroupB(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public MyViewGroupB(Context context, AttributeSet attrs,
                        int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Log.d("LOG", "ViewGroupB dispatchTouchEvent" + ev.getAction());
        return super.dispatchTouchEvent(ev);
    }
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Log.d("LOG", "ViewGroupB onInterceptTouchEvent" + ev.getAction());
        return super.onInterceptTouchEvent(ev);
    }
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("LOG", "ViewGroupB onTouchEvent" + event.getAction());
        return super.onTouchEvent(event);
    }
}
 ```
MyView.java
 ```  
public class MyViewC extends View {
    public MyViewC(Context context) {
        super(context);
    }
    public MyViewC(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public MyViewC(Context context, AttributeSet attrs,
                   int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Log.d("LOG", "View onTouchEvent" + event.getAction());
        return super.onTouchEvent(event);
    }
    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        Log.d("LOG", "View dispatchTouchEvent" + event.getAction());
        return super.dispatchTouchEvent(event);
    }
}
 ```

activity_main.xml
 ```  
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <com.example.administrator.testview.MyViewGroupA
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@android:color/holo_blue_bright">
        <com.example.administrator.testview.MyViewGroupB
            android:layout_width="300dp"
            android:layout_height="300dp"
            android:background="@android:color/holo_green_dark">
            <com.example.administrator.testview.MyView
                android:layout_width="150dp"
                android:layout_height="150dp"
                android:background="@android:color/darker_gray" />
        </com.example.administrator.testview.MyViewGroupB>
    </com.example.administrator.testview.MyViewGroupA>
</RelativeLayout>
 ```
这里有2个ViewGroup,一个View,结构如下

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-5a0d0e152f9d4985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以看到MyViewGroupA,在最外层，MyViewGroupB在中间，MyViewC在最底层。
ViewGroup分别重写了dispatchTouchEvent，onInterceptTouchEvent，onTouchEvent
View重写了onTouchEvent，dispatchTouchEvent
可以看到**ViewGroup比View多了一个方法**，看名字是拦截的意思。

当我们点击MyViewC打印log如下
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-46fd698386ff623d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到事件的传递顺序是ViewGroupA -> ViewGroupB - > MyView
事件的处理顺序是MyView - > ViewGroupB - >ViewGroupA

Android对dispatchTouchEvent 的解释如下
```
/**
  * Pass the touch screen motion event down to the target view, or this
  * view if it is the target.
  *
  * @param event The motion event to be dispatched.
  * @return True if the event was handled by the view, false otherwise.
**/
```

dispatchTouchEvent 方法用来传递事件，返回True ,拦截，返回值false不拦截，继续传递。

onTouchEvent也类似，返回True处理，返回False交给上级处理。
可以知道无论是dispatchTouchEvent还是 onTouchEvent，如果返回True，表示这个事件被消费了、处理了不再往下传。

为了了解拦截过程，先忽略dispatchTouchEvent与onTouchEvent方法，简单修改ViewGroupB    onInterceptTouchEvent为true，同样点击MyViewC，log如下

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1816215-cd610dd471001640.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以看到ViewGroupB拦截后，果然MyView就没有事件继续传递了，事件被ViewGroupB自己完成。