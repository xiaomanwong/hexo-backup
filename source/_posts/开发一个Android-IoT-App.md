---
title: 开发一个Android IoT App
date: 2019-04-16 17:43:18
tags:
---

# 构建 Android IoT App

本文翻译自[Building IoT APP for Android Things in 3 step](http://www.survivingwithandroid.com/2017/01/building-iot-app-android-things-android-iot-appplication-development.html)

## 前言

这篇文章主要描述了，如何为 Android Things 构建 Android IoT App。 也许你早已经知道了，最近 Google 发布了一个新的 IoT 操作系统-- Android Things。Android Things 系统，是由 Android 系统衍生出来的，更有意思的是，我们可以使用我们的 Android 知识来开发 Android IoT 应用程序。在开始之前，了解 [Android Things and how it works](http://www.survivingwithandroid.com/2017/01/android-things-android-internet-of-things.html) 是很有必要的。


## 名词索引

Android IoT App ： 安卓物联网应用

Raspberry Pi 3 ： 树莓派 3



## 目标

这篇文章的目标是：

1. 使用 Android Things 构建一个简单的 RGB Led 控制器
2. 使用 Android API 构建 Android IoT UI 开发

我们会使用 `Raspberry Pi 3 ` 作为 [IoT 开发板](http://www.survivingwithandroid.com/2016/08/iot-rapid-prototyping-board.html),你也可以使用其他的开发板去开发 Android Things。

此Android IoT应用可帮助您熟悉新的Android Things API。 此外，这个物联网应用程序对于开发Android IoT 应用 UI 的概述很有用。

## 步骤一

通常情况下，一个 IoT 工程有两部分， 电气/电子部分和软件部分。让事情变得简单，使我们可以集中精力在Android IoT App, 这个 IoT 应用控制着一个简单的 RGB LED （共阳极）灯。RGB Led 灯使用220Ω电阻链接到 `Raspberry`， 每个颜色一个，原理图如下：

![引脚原理图](http://upload-images.jianshu.io/upload_images/1550996-ce68329a358d9cb1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

共阳极 RGB LED 灯非常常见，因此 `Raspberry Pi 3` 为引脚阳极供电。控制 LED 颜色的 RGB 引脚连接到 `Raspberry` 引脚：

* Pin 29
* Pin 31
* Pin 33

这些引脚索引是非常重要的，因为我们会在 Android IoT App 上使用它。 上电前，请仔细检查 Raspberry 链接是否有异常。

现在，我们使用  `Android Studio` 创建一个 IoT 应用，第一步，配置 Android IoT 工程， build.gradle :

```
dependencies {
    provided 'com.google.android.things:androidthings:0.1-devpreview'
}
```

Android Things 使用 Activity ，就像我们在 Android 中使用一样。因此，让我们创建一个 `RGBThingsActivity` 类，并在 `onCreate` 方法中处理 `Pin` 通信。

## 步骤二

使用 `GPIO` 引脚与 RGB LED 传递信息。 `GPIO` 引脚使用可编程的接口去获取设备的状态或者设置输出值（高电平/低电平），使用 Respberry GPIO 音及哦啊，我们开启或关闭三个颜色的组件（红绿蓝）。

Android Things SDK 提供了一个 `PeripheralManagerService` 的服务，去抽象 GPIO 通信接口。每当我们想读写数据时都必须使用它。一开始， Android IoT App 初始化服务，并设置引脚值：

```
try {
   PeripheralManagerService manager = new PeripheralManagerService();
   blueIO = manager.openGpio("BCM5");
   blueIO.setDirection(Gpio.DIRECTION_OUT_INITIALLY_LOW);
   greenIO = manager.openGpio("BCM6");
   greenIO.setDirection(Gpio.DIRECTION_OUT_INITIALLY_HIGH);
   redIO = manager.openGpio("BCM13");
   redIO.setDirection(Gpio.DIRECTION_OUT_INITIALLY_LOW);
   redIO.setValue(false);
   blueIO.setValue(false);
   greenIO.setValue(false);
} catch (IOException e) {
   Log.w(TAG, "Unable to access GPIO", e);
}
```

这段代码介绍了一些新的重要的新方面。首先,我们必须选对引脚。如果使用的是 Respberry ，我们需要知道每一个引脚都有对应的序号。同样的方式，Android Things 使用相同的寻址模型，不管怎样，引脚的命名都是用不同的方式。通过 [Respberry Pin reference](https://developer.android.com/things/hardware/raspberrypi-io.html) ,下图：

![树莓派引脚图](http://upload-images.jianshu.io/upload_images/1550996-0957d79bc802bd94?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以了解到 `Respberry Pi 3` 的引脚地址。这些地址名称在上面的代码中使用。 例如，要使用引脚BCM5（或引脚29），代码为：

```
blueIO = manager.openGpio("BCM5");
```

开始， 我们设置所有的引脚为低电平状态（低电平即为关闭状态），此时 Led 灯为关闭状态。改变引脚的状态值，由低电平调整到高电平，或者有高电平调整为低电平，我们可以看到灯的颜色变化。

## 步骤三

Android Things 另外一个有趣的功能是，为我们提供了 UI Interface。 我们开发一个 UI Interface 给 Android IoT App 和开发 Android UI 一样。就像 Android app 一样， Android Things UI 同样是使用 xml 格式开发。 下面例子，我们去配置控制 RGB Led 显示的 3 个开关：

```
<?xml version="1.0" encoding="utf-8"?>
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:orientation="vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent">
 
 <Switch android:text="Red"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:id="@+id/switchRed"
         android:layout_marginTop="20dp"/>
 
 <Switch android:text="Green"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/switchGreen"
        android:layout_marginTop="20dp"/>
 
 <Switch android:text="Blue"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:id="@+id/switchBlue"
         android:layout_marginTop="20dp"/>
 
</LinearLayout>
```

在 `onCreate` 方法中，我们设置 layout 布局：

```
@Override
public void onCreate (Bundle savedInstanceState) {
    super.onCreate(saveInstanceState);
    setContentView(R.layout.activity_main);
    ...
}
```

处理用户开关：

```
Switch switchRed = (Switch)findViewById(R.id.switchRed);
switch.setOnCheckedChangedListener(new CompoundButton.OnCheckedChangedListener(){
    @Override
    public void onCheckedChanged (CompoundButton buttonView, boolean isChecked){
        try {
            redIO.setValue(!isChecked);
        } catch (IOException e) {
            Log.w(TAG,"Red GPIO Error", e);
        }
    }
});
```

我们必须为其他引脚重复同一段代码。最终结果如下：

因 MarkDown 模式下， 简书不支持视频播放，请点击一下链接观看。

[最终结果展示--需要翻墙 youtube 上观看](https://www.youtube.com/embed/KT_FAqMbbNQ)

<iframe width="560" height="315" src="https://www.youtube.com/embed/KT_FAqMbbNQ" frameborder="0" allowfullscreen></iframe>

最后，要使用我们的应用程序，我们必须在 `Manifest.xml` 文件中条件：

```
<uses-library android:name="com.google.android.things"/>
```

并且声明我的 `Activity` 是一个 IoT Activity， 启动脚本为：

```
<intent-filter>
    <category android:name="android.intent.category.IOT_LAUNCHER"/>
    <category android:name="android.intent.category.DEFAULT”/>
</intent-filter>
```

## 总结

文章最后，你已经知道了如何更好的使用 Android Things。 有趣的是，使用一些新的 API Android 开发人员可以准备下一次技术革命成为物联网。此外，开发过程与 Android 应用程序相同。

使用简单的几行代码，一个 Android 开发者就可以构建 Android IoT App.