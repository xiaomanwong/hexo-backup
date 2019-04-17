---
title: Android 获取 IP 地址
date: 2019-04-16 17:46:00
tags:
---

开发中经常会需要判断当前是否连接网络, WiFi 或 移动数据连接判断的需求,


**第一种方法: **

``` java
  WifiManager wifiManager = (WifiManager) getSystemService(WIFI_SERVICE);  
  WifiInfo wifiInfo = wifiManager.getConnectionInfo();  
  int ipAddress = wifiInfo.getIpAddress();  
```

通过这种方式获取到的 IP 地址为一串数字,我们并不能看懂,因此我们需要通过下面的方法进行转换:

```
String ip = (ipAddress & 0xff) + "." + (ipAddress>>8 & 0xff) + "." + (ipAddress>>16 & 0xff) + "." + (ipAddress >> 24 & 0xff);
```

这样转换之后,我们获取到的 IP 地址就是我们平时认识的, 比如: 192.168.1.108

这种方法在飞行模式下获取到的 IP 地址为 0.0.0.0

**第二种方法:**

```
  public String getLocalIpAddress() {  
    try {  
        for (Enumeration<NetworkInterface> en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements();) {  
            NetworkInterface intf = en.nextElement();  
            for (Enumeration<InetAddress> enumIpAddr = intf.getInetAddresses(); enumIpAddr.hasMoreElements();) {  
                InetAddress inetAddress = enumIpAddr.nextElement();  
                if (!inetAddress.isLoopbackAddress()) {  
                    return inetAddress.getHostAddress().toString();  
                }  
            }  
        }  
    } catch (SocketException ex) {  
        Log.e(LOG_TAG, ex.toString());  
    }  
    return null;  
}  
```

第二种方式是比较通用的,在WiFi和3G/4G 状态下,都可以获取到正确的地址.比如: fe80::8e3a:e3ff:fe45:a018

这种方法在手机处于飞行状态下时, 获取到的 IP 地址为 null