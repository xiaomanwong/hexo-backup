---
title: 线程通信基础流程
date: 2019-08-01 21:54:08
tags: Android
---



[TOC]



Android 中线程通信靠的就是 Handler、Looper、Message、MessageQueue 这四个。



## Looper 

在 Looper 中，维持一个 Thread 对象以及 MessageQueue， 通过 Looper 的构造函数可以看出

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);// 传入的参数代表这个 Queue 是否能够被退出
    mThread = Thread.currentThread();
}
```

Looper 在构造函数里做了两件事儿：

1. 将线程对象指向了创建的 Looper 的线程
2. 创建了一个新的  MessageQueue

分析完构造函数后，看下面两个方法

1. looper.loop()
2. looper.prepare()

<!--more-->

### looper.loop()

在当前线程中启动一个 Message loop 机制

```Java
public static void loop(){
    final Looper me = myLooper();// 获取当前线程绑定的 Looper
    if(me == null) {
		throw new RuntimeException("No Looper; Looper.prepare() wasn`t called on this thread");        
    }
    final MessageQueue queue = me.mQueue();// 获取与 Looper 绑定的 MessageQueue
    // make sure the identity of this thread is that of the local process,
    // And keep track of what that identity token actuall is.
    Binder.clearCallingIdentity();
    fianl long ident = Binder.clearCallingIdentity();
    
    // 进入死循环，不断获取对象，分发对象到 Handler 中去消费
    for(;;) {
        Message msg = queue.next();// 不断获取下一个 message 对象，这里可能会造成阻塞。
        if(msg == null) {
            // No message idicates that the message queue is quittig.
            return;
        }
        
        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if(logging != null) {
            logging.println(">>> Dispatching to " + msg.target + " " + msg.callback _ ": " + msg.what);
        }
        
        // 在这里开始分发 Message 
        msg.target.dispatchMessage(msg);
        
        if(logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }
        
        // Make sure that during the course of dispatching the identity of the thread wasn`t corrupted.
        
        final long newIdent = Binder.clearCallingIdentity();
        if(ident != null) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
        }
        // 当分发完 Message 后，当然要标记将该 Message 为 “正在使用”
        msg.recycleUnchecked();
    }
}
```

从上面的代码可以看出，最重要的方法是：

1. `queue.next()`
2. `msg.target.dispatchMessage(msg)`
3. `msg.recycleUnchecked()`

其实 Looper 中最重要的部分都是由 Message、MessageQueue 组成的，这段代码设计到的四个对象，他们彼此的关系如下:

1. MessageQueue： 装食物的容器
2. Message ：被装的食物
3. Handler （msg.target 实际上就是 Handler）：食物的消费者
4. Looper：负责分发食物的人



### looper.prepare() 在当前线程关联一个 Lopper 对象

```java
private static void prepare(boolean quitAllowed) {
    if(sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Lopper may be created per thread");
    }
    
    // 在当前线程绑定一个 looper
    mThreadLocal.set(new Looper(quiteAllowed));
}
```

在上面的代码中，做了两件事儿：

1. 判断当前线程有没有 Looper，如果有则抛出异常（在这里我们就可以知道，Android 规定一个线程只能拥有一个与自己关联的 Looper。
2. 如果有的话，那么就设置一个新的 Looper 到当前线程。

### Handler 的使用

```java
Handler handler = new Handler (){
    // 
    @Override
    public void handleMessage(Message msg) {
        // handle your message
    }
}
```

我们先来看下 Handler 的构造：

```java
// 空参构造与之对应
public Handler(Callback callbacck, boolean async) {
    // 大姨泄漏提醒log
    ...
    // 获取与创建 Handler 线程的 Looper
    mLooper = Looper.myLooper();
    if(mLooper == null) {
        throw new RunntimeException("Can`t create handler inside thread that has not called Looper.prepare()");
    }
    
    // 获取 Looper 绑定的 MessageQueue
    // 因为一个 Looper 只有一个 MessageQueue， 也就是与当前线程绑定的 MessageQueue
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

1. Looper.loop() 死循环中的 msg.target 是什么时候被赋值的？
2. handler.handleMessage(msg) 在什么时候被回调的？



**A1:** Looper.loop() 死循环中的 msg.target 是什么赋值的呢？要分析这个问题，自然的就想到从发送消息开始，无论是 handler.sendMessage(msg)  还是 handler.sendEmptyMessage(what)， 最后都可以追溯到下面这个方法：

```java
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    // 引用 Handler 中的 MessageQueue
    // 这个 MessageQueue 就是创建 Looper 时被创建的 MessageQueue
    MessageQueue queue = mQueue;
    
    if(queue == null) {
        RuntimeException e = new RuntimeException(
        this + " sendMessageAtTime() called withe no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    // 将新来的 Message 加入到 MessageQueue 中
    return enqueueMessage(queue, msg, uptimeMillis);
}
```

接下来看一下 `enququeMessage(queue, msg, uptimeMillis)`:

```java
private boolean enququeMessage(MessageQueue, queue, Message msg, long uptimeMillis) {
   	msg.target = this;
   	if(mAsynchronous) {
        msg.setAsynchronous(true);
   	}
   	return queue.enqueueMessage(msg, uptimeMillis);
}
```



**A2**:  handler.handleMessage(msg) 在什么时候被回调？通过上面的分析，我们很明确知道， Message 中的 target 是在什么时候赋值的，我们先来分析下在 Looper.loop() 中出现过的 dispatchMessage(msg) 方法

```Java
public void diapatchMessage(Message msg) {
    if(msg.callback != null) {
        handleCallback(msg);
    } else {
        if(mCallback != null) {
            if(mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg); // 开始回调
    }
}
```

通过上面的分析，可以清楚的知道 Handler, Looper, Message, MessageQueue 这四者的关系以及如何合作了。



## 总结

当我们调用 handler.sendMessage(msg)； 方法发送一个 Message 时， 实际上这个 Message 是发送到与当前线程绑定的一个 MessageQueue 中，然后与当前线程绑定的 Looper 将会不断的从 MessageQueue 中取出新的 Message， 调用 msg.target.dispatchMessage(msg) 方法将消息发送到与 Message 绑定的 handler.handleMessage() 方法中。

一个 Thread 对应多个 Handler， 一个 Thread 对应一个 Looper 和 MessageQueue， Handler 与 Thread 共享 Looper 和 MessageQueue。 Message 只是消息的载体，将会被发送到与线程绑定的唯一的 MessageQueue 中，并且被与线程绑定的唯一的 Looper 分发，被其自身绑定的 Handler 消费。