---
layout: okhttp
title: 获取参数
date: 2019-04-25 07:16:59
tags:
---


## Interceptor

拦截器，在 Okhttp 中就是可以拦截网络请求，并在 Interceptor 中实现自己想要的业务逻辑；
实现一个典型的拦截器很容易，主需要我们实现 `Interceptor` 接口，并重写 `intercept` 方法，

例如我们实现一个日志拦截器：

```java
public class LogInterceptor implements Intercept(){

    private static final String TAG = "LogInterceptor";

    @Override
    public Response intercept(Chain chain) throws IOException{
        Request request = chain.request();
        
        long t1 = System.nanoTime();
        Log.d(TAG, String.format("Send request %s on %s%n%s", 
        request.url(), chain.connection(), request.headers()));
        
        Response response = chain.proceed(request);
        long t1 = System.nanoTime();
        Log.d(TAG, String.format("Received response for %s in %.1fms%n%s",
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));
        
        return response;
    }
}

```

上面的操作很简单，那么下面我们来说说如何拦截 http 请求，获取其中的参数


## Get 请求

Get 请求，在网络请求中和 POST 一样是最常用的两种方式，一个正常的 http get 请求链接大概是这个样子 `https://stackoverflow.com/questions?id=32141241`

从上面的地址上我们大概能看出，当前请求的 HOST, PATH, 以及参数 id，故此，当我们需要拦截请求，获取请求参数时，


