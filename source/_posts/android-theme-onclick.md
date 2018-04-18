---
title: Android 5.0中的布局文件onClick回调报错
date: 2018-03-29 20:16:55
tags: [android]
---

监听控件的onClick事件有两种方法：
- 方法1
调用控件的setOnClickListener设置监听回调。

```java
    Button button = (Button) findViewById(R.id.button);
    button.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
        }
    });
```

<!--more-->

- 方法2
将回调方法名赋值给布局文件中的onClick属性，然后在Activity中实现该方法。

```xml
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"
        android:onClick="onClickButton"/>
```

```java
    public void onClickButton(View view) {
    }
```

但是方法2在5.0上有个坑，如果当前控件的layout的中xml中写了theme属性，那么就会报找不到该方法的错误。
如
```xml
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:background="@color/colorPrimary"
        android:onClick="onClickButton"
        android:theme="@style/AppTheme.MenuDefaultItem">
        <TextView
            android:text="text"
            android:layout_width="100dp"
            android:layout_height="100dp" />
    </LinearLayout>
```
这是一个LinearLayout嵌套一个TextView，将onClick回调写在LinearLayout里，然后加上一个theme属性，这时点击LinearLayout就会报错。
```
03-30 08:48:24.060 29147-29147/trendit.com.posusageintro E/AndroidRuntime: FATAL EXCEPTION: main
                                                                           Process: trendit.com.posusageintro, PID: 29147
                                                                           java.lang.IllegalStateException: Could not find a method onClickButton(View) in the activity class android.view.ContextThemeWrapper for onClick handler on view class android.widget.LinearLayout
                                                                               at android.view.View$1.onClick(View.java:4008)
                                                                               at android.view.View.performClick(View.java:4781)
                                                                               at android.view.View$PerformClick.run(View.java:19874)
                                                                               at android.os.Handler.handleCallback(Handler.java:739)
                                                                               at android.os.Handler.dispatchMessage(Handler.java:95)
                                                                               at android.os.Looper.loop(Looper.java:135)
                                                                               at android.app.ActivityThread.main(ActivityThread.java:5254)
                                                                               at java.lang.reflect.Method.invoke(Native Method)
                                                                               at java.lang.reflect.Method.invoke(Method.java:372)
                                                                               at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:902)
                                                                               at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:697)
                                                                            Caused by: java.lang.NoSuchMethodException: onClickButton [class android.view.View]
                                                                               at java.lang.Class.getMethod(Class.java:664)
                                                                               at java.lang.Class.getMethod(Class.java:643)
                                                                               at android.view.View$1.onClick(View.java:4001)
                                                                               at android.view.View.performClick(View.java:4781) 
                                                                               at android.view.View$PerformClick.run(View.java:19874) 
                                                                               at android.os.Handler.handleCallback(Handler.java:739) 
                                                                               at android.os.Handler.dispatchMessage(Handler.java:95) 
                                                                               at android.os.Looper.loop(Looper.java:135) 
                                                                               at android.app.ActivityThread.main(ActivityThread.java:5254) 
                                                                               at java.lang.reflect.Method.invoke(Native Method) 
                                                                               at java.lang.reflect.Method.invoke(Method.java:372) 
                                                                               at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:902) 
                                                                               at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:697) 
                                                                               
```

[StackOverflow](https://stackoverflow.com/questions/27531381/android-5-and-onclick-in-xml-layout/28345359#28345359)上有一段解释，在Android 5.0上的layout布局上，如果应用android:theme属性，就会报这个错误。较老的版本不会有这个问题。解决方法就是要么使用上面的方法1，要么将android:theme应用到application上。
```xml
<application 
         ...
         android:theme="@android:style/Theme.Holo.Light"
         ...  >

```