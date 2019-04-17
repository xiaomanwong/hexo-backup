---
title: Android 中设置全屏的方法
date: 2019-04-16 17:46:31
tags: Android
---

在开发中,我们经常需要把我们的应用设置为全屏,这里有两种方式: 一是在代码中设置; 二是在配置文件中设置

# 一. 在代码中设置

```
public class BaseActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(saveInstanceState);
        // 无title
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        // 全屏
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.activity_main);
    } 
}
```

强调一点: ** 设置全屏的两段代码,必须在 `setContentView()` 之前调用,不然会报错 **

# 二. 在配置文件中修改

```
<?xml version="1.0" encoding="utf-8"?>
<manifest 
  xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.android.test"
  android:versionCode="1"
  ndroid:versionName="1.0">
  <application android:icon="@drawable/icon"
    android:lable="@string/app_name>
    <activity android:name=".BaseActivity" 
        android:theme="@android:style/Theme.NotitleBar.Fullscreen"
        android:lable="@string/app_name">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
  </application>
</manifest>
```

使用第一种方法,会在应用运行后,看到一个短暂的状态来, 然后才全屏, 而第二种方法是不会有这种情况的,大家根据需要自行选择.

