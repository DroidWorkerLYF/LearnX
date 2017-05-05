#零散记录
##Ripple效果
[链接](http://michaelevans.org/blog/2015/05/07/android-ripples-with-rounded-corners/)  
给View设置ripple效果时，需要让波纹是view background的形状，可以在ripple的xml中添加一个item和view background是同样的形状，并指定id为@android:id/mask。

```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:attr/colorControlHighlight">
    <item android:id="@android:id/mask">
        <shape android:shape="rectangle">
            <solid android:color="#000000" />
            <corners android:radius="15dp" />
        </shape>
    </item>
    <item android:drawable="@drawable/rounded_corners" />
</ripple>
```

##图片格式选择
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/img/android_image_format_choosen.png?raw=true)