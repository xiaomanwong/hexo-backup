---
title: adb command
date: 2019-04-16 17:38:08
tags: Android

---

1. 查看PID

  ```
  adb shell ps
  ```

2. 查看日志

  ```
  全部日志
  adb logcat
  
  按照 PID 筛选日志
  adb logcat | grep <PID>
  ```

3. 查询链接设备

  ```
  adb devices
  ```
<!--more-->

4. 重启机器

  ```
  adb reboot
  ```

5. 杀死进程服务

  ```
  adb kill-server
  ```

6. 重启进程服务

  ```
  adb start-server
  ```

7. 获取机器 Mac 地址

  ```
  adb shell cat /sys/class/net/wlan0/address
  ```

8. 安装 APK

  ```
  普通安装
  adb install <file path>
  
  保留数据和缓存文件
  adb install -r <file path>
  
  安装到 SD 卡上
  adb install -s <file path>
  ```

9. 卸载 APK

  ```
  普通卸载
  adb uninstall <package name>
  
  保留数据和缓存文件
  adb uninstall -k <package name>
  ```

10. 启动应用

    ```
    adb shell am start -n <package name>/.<activity_class_name>
    ```

11. 查看 CPU 占用率

    ```
    adb shell top
    
    查看内存占用前 6 的 app
    adb shell top -m 6
    
    刷新一次内存
    adb shell top -n 1
    ```

12. 杀死进程

    ```
    adb shell kill <pid>
    ```

13. 将 System 分区重新挂在为可读写分区

    ```
    adb remount
    ```

14. 从本地复制文件到设备

    ```
    adb push <local> <remote>
    ```

15. 从设备复制文件到本地

    ```
    adb pull <remote> <local>
    ```

16. 查看 WiFi 密码

    ```
    adb shell cat /data/misc/wifi/*.conf
    ```

17. 查看 bug 报告

    ```
    adb bugreport
    ```

18. 跑 monkey

    ```
    adb shell monkey -v -p your.package.name 500
    ```

19. 截图

    ```
    adb shell screencap -p /sdcard/screenshot.png
    ```

20. 录屏

    ```
    adb shell screenrecord /sdcard/demo.mp4
    ```

21. 查看安装列表

    ```
    adb shell pm list packages
    ```
    
11. 安装证书

    ```
    adb shell am start -n com.android.certinstaller/.CertInstallerMain -a android.intent.action.VIEW -t application/x-x509-ca-cert file:///sdcard/cacert.cer
    ```

22. 查看内存占用

    ```
    adb shell dumpsys meminfo <package> -d
    ```

23.  查询栈信息

    ```
    adb shell dumpsys activity
    // 获取自己应用
    adb shell dumpsys activity | grep <package>
    // 获取处于栈顶的 activity
    adb shell dumpsys activity | grep mFocusedActivity
    ```


