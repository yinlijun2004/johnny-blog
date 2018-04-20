---
title: 带进度条按钮的实现
date: 2018-04-20 19:34:08
tags: [android, button, progressbar]
---

写一个带进度条的按钮：[ProgressActionButton](https://github.com/yinlijun2004/ProgressActionButton)，支持外观定制。

可以当单纯的进度条使用，也可以当单纯的按钮使用，还可以当带进度的按钮使用。

主要难点就是实现椭圆按钮的绘制，思路就是，先绘制背景图片，再绘制前景图片，然后剪切一个圆角矩形出来。

<!-- more -->

```java
private Bitmap drawRadiusBitmap(Bitmap bg, Bitmap fg, Rect rect, RectF rectF) {
    Bitmap output = Bitmap.createBitmap(rect.width(), rect.height(), Bitmap.Config.ARGB_8888);
    Canvas canvas = new Canvas(output);
    int color = 0xff424242;
    Paint paint = new Paint();
    paint.setAntiAlias(true);
    canvas.drawARGB(0, 0, 0, 0);
    paint.setColor(color);

    canvas.drawRoundRect(rectF, buttonRadius, buttonRadius, paint);
    paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
    canvas.drawBitmap(bg, rect, rect, paint);
    if(fg != null) {
        Rect fgRect = new Rect(rect.left, rect.top, rect.left + (int) (rect.width() * progress / 100.0f), rect.bottom);
        canvas.drawBitmap(fg, fgRect, fgRect, paint);
    }

    return output;
}
```

支持定制属性：

```xml
<declare-styleable name="ProgressActionButton">
    <attr name="initBg" format="reference" />
    <attr name="successBg" format="reference" />
    <attr name="failBg" format="reference"/>
    <attr name="progressBg" format="reference"/>
    <attr name="progressFg" format="reference"/>
    <attr name="disableBg" format="reference"/>
    <attr name="buttonRadius" format="dimension"/>
</declare-styleable>
```

用法：
```xml
<com.trendit.progressactionbutton.ProgressActionButton
    android:id="@id/progress_bar"
    android:layout_width="match_parent"
    android:layout_height="@dimen/progress_action_button_height"
    android:textSize="@dimen/progress_action_button_text_size"
    android:textColor="#DEFFFFFF"
    app:initBg="@drawable/default_progress_bg"
    />
```
