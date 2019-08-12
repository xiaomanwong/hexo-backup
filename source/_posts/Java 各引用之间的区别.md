---
title: Java Reference
date: 2019-08-09 21:54:08
tags: Java
---



## Reference 引用

继承自 Object， 有 SoftReference, WeakReference, PhantomReference 三个直接子类。

三个子类的构造方法中涉及到 ReferenceQueue 和 Referent：

* Referent： 被引用对象
* ReferenceQueue：当引用（软引用、弱引用、虚引用）的 Referent 被回收后，该引用（软引用、弱引用、虚引用）会被 enqueue 到这个 ReferenceQueue 中。



```java
public class ReferenceTest{
    public static void main(String[] args) {
        
    }
    
    static class Person　{
        private String name;
        private String getName(){
            return name;
        }
        private void setName(String name) {
            this.name = name;
        }
        
        @Override
        protected void finalize() throws Throwable {
            super.finalize();
            System.out.println("in Person finalize");
        }
    }
}

```





## StrongReference 强引用

Java 中使用的最多，普通的引用 `Object obj = new Object();  Person p = new Person();` 都属于`强引用` 

强引用本身存储在栈中，new 出来的对象存储在堆中。栈中保存的引用指向堆中对象的地址。

一般情况下，当引用不在指向堆中对象的地址时（person = null） GC collector 就开始考虑对此内存（堆中的对象）进行回收。



```Java
Person p = new Person();
```

person 就是一个强引用，强引用不会被 GC，即使内存不够抛出 OOM 时也不会被回收。



## SoftReference 软引用

软引用普通使用形式：

```java
Person p = new Person();
SoftReference<Person> sr = new SoftReference<Person>(person);
```

强引用 person 作为参数，创建了一个软引用对象 sr, 下面是例子：

```java
private static void testSofeReference(){
    // 创建强引用对象
    Person person = new Person();
    System.out.println("person 对象为" + person);
    
    // 创建软引用对象
    SoftReference<Person> sr = new SoftReference<Person>(person);
    person = null; // 之前 new 出的 Person 对象不会立即被回收，除非 JVM 需要内存(OOM 之前)
    if(sr.get() == null) {
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + sr.get());
    }
    
    System.gc();
    
    if(sr.get() == null){
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + sr.get());
    }
}

```

执行上面的例子：

```
person 对象为 com.example.ReferenceTest$Person@522d9d8c
person 对象尚未被回收 com.example.ReferenceTest$Person@522d9d8c
person 对象尚未被回收 com.example.ReferenceTest$Person@522d9d8c
```

* 当执行 person = null 后，堆内存的 Person 对象不再有任何强引用指向它，但此时还存在 sr 引用的对象指向 Person 对象。此时调用 sr.get() 方法，返回 Person 对象，即之前堆中的强引用对象了。我们可以合理猜测GC collector 很有可能尚未进行垃圾回收，所以此时 sr.get() 方法返回不为空。
* 我们继续执行 `System.gc()` 强制进行垃圾回收，打印结果可以看到， sr.get() 返回依然不为空，说明 Person 对象依旧没有被回收。



**软引用所指向的对象要进行回收，需要满足两个条件：**

1. 没有任何强引用指向软引用指向的对象（内存中的 Person 对象）
2.  JVM 需要内存时，即在抛出 OOM 之前

**总结：** SoftReference 变相延长了其只是对象占据堆内存的时间，直到虚拟机内存不足时垃圾回收才回收此堆内存空间。



**软引用还可以和一个 ReferenceQueue 一起使用，** 当 SoftReference 的 Referent 被回收以后，这个 SoftReference 会被自动 enqueue 到这个 ReferenctQueue 中。

```java
private static void testSoftReferenceWithQueue(){
    Person person = new Person();
    System.out.println("person 对象为：" + person);
    
    ReferenceQueue<Person> queue = new ReferenceQueue<>();
    SoftReference<Person> sr = new SoftReference<>(person, queue);
    
    person = null; // 之前 new 出的 Person 对象不会立即被回收，除非 JVM 需要内存 （OOM 前）
    if(sr.get() == null) {
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + sr.get());
    }
    
    System.out.println("加入 ReferenceQueue 的对象为：" + queue.poll());
    
    System.gc();
    
    if(sr.get() == null) {
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + sr.get());
    }
    
    System.out.println("加入 ReferenceQueue 的对象为： " + queue.poll());
}
```

执行上述例子，结果如下：

```
person 对象为 com.example.ReferenceTest$Person@522d9d8c
person 对象尚未被回收 com.example.ReferenceTest$Person@522d9d8c
加入 ReferenceQueue 的对象为 null
person 对象尚未被回收 com.example.ReferenceTest$Person@522d9d8c
加入 ReferenceQueue 的对象为 null
```

**注意：** 当 SoftReference 或 WeakReference 的 get() 方法返回 null 时，仅表明其指示的对象已经进入垃圾回收流程，此时对象不一定已经被垃圾回收。

而只有确认被垃圾回收后，如果有 ReferenceQueue ，其引用才会被放置于 ReferenceQueue 中。



## WeakReference 弱引用

弱引用的一般使用形式：

```Java
private static void testWeakReference(){
    Person person = new Person();
    System.out.println("person 对象为： " + person);
    
    WeakReference<Person> wr = new WeakReference<>(person);
    person = null; // 被 GC 后，之前 new 出的 Person 对象会立即被回收，进入 GC 流程。
    if (wr.get() == null) {
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + wr.get());
    }
    
    System.gc();
    
    if (wr.get() == null) {
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + wr.get());
    }
}
```

执行结果为：

```
person 对象为 com.example.ReferenceTest$Person@522d9d8c
person 对象尚未被回收 com.example.ReferenceTest$Person@522d9d8c
person 对象进入 GC 流程
in Person finalize
```

* 当执行 `person = null` 后，堆内存的  Person 对象不再有任何引用指向它，但此时还存在 wr 引用的对象指向 Person 对象。

  此时调用 `wr.get()` 方法，返回 Person 对象，即之前堆中的强引用对象，我们可以合理猜测 GC collector 很有可能尚未进行垃圾回收，所以此时 `wr.get()` 方法返回不为空。

* 继续执行 `System.gc()` 强制执行垃圾回收，打印结果可以看到，`wr.get()` 返回为空 “person 对象进入 GC 流程” ，且执行了静态内部类中的 finalize 方法。说明 Person 对象被回收，进入垃圾回收流程。



**弱引用所指向的对象要进行回收，只需要满足条件：**

​	没有任何强引用指向弱引用指向的对象（内存中的 Person 对象）



**总结：**

​	WeakReference 不改变原有的强引用独享的垃圾回收机制，一旦其指示对象没有任何强引用对象时，此对象即进入正常的垃圾回收流程。



**其主要使用场景见于：** 

​	当前已有强引用指向强引用对象，此时由于业务需要，需要增加对此对象的引用，同时又不希望改变此引用的垃圾回收机制，此时 `WeakReference` 正好符合需求，常见于一些与生命周期的场景中，比如 Activity 中的 Handler 的使用，为了防止内存泄露需要用到弱引用。



与 SoftReference 一样，可以同 ReferenceQueue 一起使用。当 WeakReference 的 Referent 被回收以后，这个 WeakReference 会被自动 enqueue 到这个 ReferenceQueue 中。

```java
private static void testWeakReferenceWithQueue () {
    Person person = new Person();
    System.out.println("person 对象为 " + person);
    
    ReferenceQueue<Person> queue = new ReferenceQueue<>();
    WeakReference<Person> wr = new WeakReference<>(person, queue);
    System.out.println("wr 对象为 " + wr);
    
    if (wr.get() == null) {
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + wr.get());
    }
    
    System.out.println("Whether or not this reference has been enqueued:" + wr.isEnqueued());
    System.out.println("Queue item: " + queue.pull());
    
    System.gc();
    
    if (wr.get() == null) {// 仅是表名指示的对象已经进入垃圾回收流程，此时对象不一定已经被垃圾回收，只有确认被垃圾回收后，如果有 ReferenceQueue ，其引用才会被放置于 ReferenceQueue 中。
        System.out.println("person 对象进入 GC 流程");
    } else {
        System.out.println("person 对象尚未被回收" + wr.get());
    }
    
    try {
        // 确保垃圾回收线程能够执行
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    System.out.println("Whether or not this reference has been enqueued: " + wr.isEnqueued());
    System.out.println("queue item: " + queue.poll());
}
```



**执行结果：**

```
person 对象为 com.example.ReferenceTEst$Person@522d9d8c
wr 对象为 java.lang.ref.WeakReference@603828d2
person 对象尚未被回收 com.example.ReferenceTEst$Person@522d9d8c
Whether or not this reference has been enqueued: false
queue item: null
person 对象进入回收流程
in Person finalize
Whether or not this reference has been enqueued: true
queue item: java.lang.ref.WeakReference@603828d2
```



从第二行到最后行可以看出， person 进入 GC 流程后，wr 被加入到 queue 中。

**注意：**

​	 当 SoftReference 或者 WeakReference 的 get() 方法返回 null 时，仅表示其指示对象已经进入垃圾回收流程，此时对象不一定已经被垃圾回收。

​	而只有确认被垃圾回收后，如果有 ReferenceQueue， 其引用才会被放置于 ReferenceQueue 中。



## PhantomReference 虚引用

虚引用源码：

```java
package java.lang.ref;

import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;

public class PhantomReference<T> extends Reference<T> {
    public T get(){
        return null;
    }
    
    public PhantomReference(T var1, ReferenceQueue<? super T> var2) {
        super.(var1, var2);
    }
}
```



* PhantomReference 只有一个构造函数 `PhantomReference(T referenct, ReferenceQueue<? super T> q) `  因此，PhantomReference 使用必须结合 ReferenceQueue；
* 不管有无强引用指向 PhantomReference 的指示对象， PhantomReference 的 get() 方法返回结果都是 null

举例：

```java
private static void testPhantomReference() {
    Person person = new Person();
    System.out.println("person 对象为" + person);
    
    ReferenceQueue<Person> queue = new ReferenceQueue<>();
    PhantomReference<Person> pr = new PhantomReference<>(person, queue);
    
    System.out.println("pr 对象为" + pr);
    System.out.println("pr.get() = " + pr.get());
    
    person = null;
    
    System.gc();
    
    try{
        // 确保垃圾回收线程执行
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    System.out.println("queue item: " + queue.poll());
}
```

**执行结果：**

```
person 对象为 com.example.ReferenceTEst$Person@522d9d8c
pr 对象为 java.lang.ref.PhantomReference@60e35b53
pr.get() = null
in Person finalize
queue ite: java.lang.ref.PhantomReference@60e35b53
```



**总结：**

​	与 WeakReference 一样，不改变原有的强引用对象的垃圾回收机制。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可以被垃圾回收。虚引用主要用来跟踪对象被垃圾回收后的活动（监听并才去必要的行动）



**用途：**

​	当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

​	程序可以通过判断医用队列中是否已经加入虚引用来了解被引用的对象是否将要被垃圾回收。

​	程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前才去必要的行动。