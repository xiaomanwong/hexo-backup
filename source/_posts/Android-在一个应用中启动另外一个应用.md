---
title: Android 在一个应用中启动另外一个应用
date: 2019-04-16 17:45:20
tags: Android
---



# Android 在一个应用中启动另外一个应用
Android 中,从当前 APP 启动另外一个 APP 的需求,不是很常见, 但确实存在着,比如说在某宝还没有现在这么强大时,支付需要启动他们的 APP. 再比如说,某米的钱包系统,需要和他们的金融 APP 在某些业务上是相互依赖的,需要相互启动.

综上所述, 从一个 APP 去启动另外一个 APP 的需求还是有它存在的价值.因此,一下是我在工作和学习中总结的如何通过一个 APP 去启动另外一个 APP 的方式.

<!--more-->

## 1. 通过 APP 启动另一个 APP

```
  String packageName = "com.android.calendar";
  Intent intent = getPackageManager().getLaunchIntentForPackage(packageName);
  intent.putExtra("type", true);
  startActivity(intent);
```

上面的代码,就可以完成从一个 APP 启动另一个 APP 的业务需求, 这里需要注意的是,我们需要检测一下要启动的 APP 是否已经安装,如果应用未安装, 则会 NullPointException.

通过这种方式, Android 虚拟机会自己在目标 APP 下寻找标签为 `android.intent.action.MAIN` 的 Activity 启动.

这里介绍两种方式,检测目标应用是否已经安装

** 方法一: **

```
Intent intent = getPackageName().getLaunchIntentForPackage(packageName);
if (intent == null) {
  // 这里判断 Intent 为空, 说明应用不存在 
}
```

**方法二:**

```
  PackageInfo packageInfo = getPackageManager(0.getPackageInfo(packageName, 0);
  if (packageInfo == null) {
      // 这里如果 packageInfo 为 null, 说明应用不存在
  }
```

## 2. 打开另外一个 APP 指定的 Activity

```
    Intent intent = new Intent();
    ComponentName componeneName = new ComponeneName("com.android.calendar", "com.android.calendar.LaunchActivity");
    intent.setComponent(componeneName);
    startActivity(intent);
```

值得注意: 

*  需要将目标 Activity 的 android:export="true" 属性在所属应用的 AndroidMainfest 里设置为 true, 意思是当前 Activity 允许被外部应用访问, 否则会报错

```
  Caused by: java.lang.SecurityException: Permission Denial: starting  Intent 
  { cmp=com.example.fm/.MainFragmentActivity (has extras) } from ProcessRecord
  {39282a97 11545:com.xing.toolbardemo1/u0a71}  (pid=11545,     uid=10071) not exported from uid 10067
```
* 在 5.0 以前的设备上,需要在当前的 AndroidMainfest 里也生命目标Activity, 否则会报错; 但在 5.0 以后的设备上,就不会报错哦

```
  Caused by: android.content.ActivityNotFoundException: Unable to find explicit activity class {com.example.fm/com.example.fm.MainFragmentActivity}; 
have you declared this activity in your AndroidManifest.xml?
```



