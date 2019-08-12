---
title: Android 内存管理
date: 2019-08-04 21:54:08
tags: Android
---


内存管理的目的就是我们在开发中怎么有效的避免我们的应用程序出现内存泄露问题。内存泄露简短粗俗的讲，就是该被释放的对象没有释放，一直被某个或某些实例所持有却不被使用，导致 GC 不能回收。



## Java 内存分配策略

Java 程序运行时的内存分配策略有三种，分别是静态分配、栈式分配和堆式分配。对应的三种存储策略使用的内存空间主要分别是静态存储区（方法区）、栈区和堆区。

* 静态存储区（方法区）：主要存放静态数据，全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
* 栈区：当方法被执行时，方法体内的局部变量（其中包括基础数据类型、对象引用）都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因此栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
* 堆区：又称动态内存分配，通常就是在程序运行时直接 new 出来的内存，也就是对象的实例。这部分内存在不使用时，将会由 Java 垃圾回收器负责回收。

<!-- more -->

## 堆与栈的区别：

在方法体内定义的（局部变量）一些基本类型的变量和对象的引用都是在方法的栈内存中分配的。当在一段方法块中定义一个变量时，Java 就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给它的内存空间也将被释放掉，该内存空间可以被重新使用。

堆内存用来存放所有由 new 创建的对象（包括该对象其中的所有成员变量）和数组。在堆中分配的内存，将由 Java 垃圾回收器来自动管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊变量，这个变量的取值等于数组或者对象在内存中的首地址，这个特殊的变量就是我们上面说的引用变量。我们可以通过这个引用变量来访问堆中的对象或者数组。

举例：

```Java
public class Sample {
    int s1 = 0;
    Sample sample1 = new Sample();
    
    public void method () {
        int s2 = 1;
        Sample sample2 = new Sample();
    }
}


Sample sample3 = new Sample();
```

Sample 类的局部变量 s2 和引用变量 sample2 都是存在于栈中，但 smaple2 指向的对象是存在于堆中。 sample3 指向的对象存放在堆中，包括这个对象的所有成员变量 s1 和 sample1， 而它自己存在与栈中。

**结论：**

* 局部变量的脚本数据类型和引用存储在栈中，引用的对象实体存储于堆中。—— 因为他们属于方法中的变量，生命周期随方法而结束
* 成员变量全部存储于堆中（包括基本数据类型，引用和引用的对象实体）。—— 因为他们属于类，类对象终究是要被 new 出来使用的。



## Java 是如何管理内存

Java 的内存管理就是对象的分配和释放的问题。在 Java 中，coder 需要通过关键字 new 为每一个对象申请新的存储空间（基本类型除外），所有的对象都在 堆（Heap）中分配空间。另外对象的释放是由 GC 决定和执行的，在 Java 中，内存分配是由 coder 完成的，而内存释放是由 GC 完成的。这种收支两条线的方法简化了 coder 的工作。同时也加重了 JVM 的工作。也是 Java 程序运行速度慢的原因之一。因为 GC 为了能够正确释放对象， GC 必须监控每一个对象的运行状态，包括对象的申请、引用、被引用、赋值等，GC 都需要监控。



监控对象状态是为了更准确、及时地释放对象，而释放对象的根本原则就是该对象不在被引用。



![](https://camo.githubusercontent.com/ba01b8ae9af4a5e588251316c826bf3e0e695f35/687474703a2f2f7777772e69626d2e636f6d2f646576656c6f706572776f726b732f636e2f6a6176612f6c2d4a6176614d656d6f72794c65616b2f312e676966)

Java 使用有向图的方式进行内存管理，可以消除引用循环的问题，例如有三个对象，相互引用，只要他们和根进程不可达，那么 GC 就可以回收他。这种管理方式的有点是管理内存的精度高，但是效率低。



## 什么是 Java 中的内存泄露

在 Java 中，内存泄露就是存在一些被分配的对象，这些对象有两个特点。

* 这些对象是可达的，即在有向图中，存在通路与其相连
* 这些对象是无用的，即程序以后不会在使用这些对象。

如果满足这两个条件，这些对象就可以判定为 Java 中的内存泄露，这些对象不会被 GC 回收，但却是占用着内存。

对于程序猿来说， GC 基本是透明的，不可见的。虽然我们只有几个函数可以方位 GC， 例如运行 GC 的函数 `System.gc()`，但是根据 Java 语言规范定义，该函数不保证 JVM 的垃圾回收器一定会执行。因为，不同的 JVM 实现着可能使用不同的算法管理 GC。 通常 GC 的线程优先级比较低。 JVM 调用 GC 的策略也有很多种，有的是内存使用达到一定成都时， GC 才开始工作；也有定时执行，有的是平缓执行 GC， 有的是中断式执行 GC。 通常来说，我们不需要关心这些。除非在一些特定的场合， GC 的执行影响应用程序的性能，例如对于基于 Web 的实时系统，如网络游戏等，用户不希望 GC 突然终端应用程序执行而进行垃圾回收，那么我们需要调整 GC 的参数，让 GC 能够通过平缓的方式释放内存，例如将垃圾回收分解为一系列的小步骤执行， Sun 提供了 HotSpot JVM 支持这一特性。

同样给出一个 Java 内存泄露的典型例子

```java
Vector v = new Vector(10);
for (int i = 1; i < 100; i++) {
    Object o = new Object();
    v.add(o);
    o = null;
}
```

在这个例子中，我们循环申请 Object 对象，并将所申请的对象放入一个 Vector 中，如果我们仅仅释放引用本身，那么 Vector 仍然应用该对象，所以这个对象对 GC 来说是不可回收的。因此，如果对象加入到 Vector 后，还必须从 Vector 中删除，最简单的方法就是将 Vector 对象设置为 null。





## 详细的 Java 中的内存泄露

### Java 内存回收机制

不论那种语言的内存分配方式，都需要返回所分配的真实地址，也就是返回一个指针到内存块的首地址。Java 中对象是采用 new 或者反射的方法创建的，这些对象的创建都是在堆（Heap）中分配的。所有对象的回收都是由 Java 虚拟机通过垃圾回收机制完成。 GC 为了能够正确释放对象，会监控每个对象的运行状态，对他们的申请、引用、被引用、赋值等状况进行监控， Java 会使用有向图的方式进行管理内存，实时监控对象是否可以到达，如果不可以到达，则将其回收，这样也可以消除引用的循环问题。在 Java 语言中，判断一个内存空间是否符合垃圾回收标准有两个：

 	1. 给对象赋予了空值 null
		2. 给对象赋予了新值，这样重新分配了内存空间。



### Java 内存泄漏引起的原因

内存泄露是指无用对象（不再使用的对象）持续占有内存活无用对象的内存得不到及时释放，从而造成内存空间的浪费称为内存泄漏。内存泄漏有时不严重，不易察觉，这样开发者就不知道存在内存泄漏，但有时也很严重，会提示 Out of memory。

Java 内存泄漏的根本原因是什么呢？长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄漏，尽管短生命周期对象已经不再需要，但是因为长生命周期持有它的引用而导致不能被释放回收，这就是 Java 中内存泄漏的发生场景，主要有一下几大类：

1. 静态集合类引起的内存泄漏：

   像 HashMap、Vector 等的使用最容易出现内存泄漏，这些静态变量的生命周期和应用程序一致，他们所引用的所有的对象 Object 也不能被释放，因为他们也将一直被 Vector 等引用着。

   例如：

   ```java
   Static Vector v = new Vercor(10);
   for (int i = 1; i < 100; i++) {
       Object o = new Object();
       v.add(o);
       o = null;
   }
   ```

   在这个例子中，循环申请 `Object` 对象，并将所申请的对象放入一个 Vector 中，如果仅仅释放引用本身（o = null），那么 Vector 仍然引用该对象，所以这个对象对 GC 来说是不可会说的。因此对象加入到 Vector 后， 还必须从 Vector 中删除，最简单的是将 Vector 设置为 null。

2. 当集合里面的对象属性被修改后，再调用 remove() 方法时，不起作用

   例如：

   ```java
   public static void main(String[] args) {
       Set<Person> set = new HashSet<Person>();
       Person p1 = new Person("唐僧", "pwd1", 25);
       Person p2 = new Person("孙悟空", "pwd2", 24);
       Person p3 = new Person("猪八戒", "pwd3", 26);
       
       set.add(p1);
       set.add(p2);
       set.add(p3);
   
       System.out.println("总共有：" + set.size() + " 个元素");// 结果： 总共有 3 个元素
       p3.setAge(2); // 修改 p3 的年龄，此时 p3 元素对应的 hashcode 值发生改变
       
       set.remove(p3); // remove 掉，造成内存泄露
       
       set.add(p3); // 重新添加，成功
       
      	System.out.println("总共有：" + set.size() + " 个元素"); // 结果：总共有 4 个元素
       for (Person person : set) {
           System.out.println(person);
       }
   }
   ```

3. 监听器

   在 Java 变成中，我们需要和监听器打交道，通常一个应用当中会有多个监听器，我们会调用一个控件的例如`addXXXXListener()` 等方法来增加监听器，但往往在释放对象的时候，却没有记住去删除这些监听器，从而增加了内存泄漏的机会。

4. 各种连接

   比如数据库连接（dataSourse.getConnection()）、 网络连接(socket) 和 io 连接，除非其显示的调用了其 `close()` 方法将其连接关闭，否则是不会自动被 GC 回收的。对于 Resultset 和 Statement 对象可以不进行显示回收，但 Connection 一定要显示回收，因为 Connection 在任何时候都无法自动回收，而 Connection 一旦回收， Resultset 和 Statement 对象就会立即2为 NULL。 但是如果使用连接池，情况就不一样了，除了要显示地关闭链接，还必须显示地关闭 Resultset 和 Statement 对象（关闭其中一个， 另外一个也会关闭），否则就会造成大量的 Statement 对象无法释放，从而引起内存泄漏。这种情况下一般都会在 try 里面去连接，在 finally 里面释放连接。

5. 内部类和外部模块的引用

   内部类的引用是比较容易遗忘的一种，而且一旦没释放可能导致一系列的后续类对象没有释放。此外 coder 还要小心外部模块不经意的引用，例如 coder A 负责 A 模块，调用了 B 模块的一个方法： `public void registerMsg(Object b);` 这种调用就要小心，传入了一个对象，很可能模块 B 就保持了对该对象的引用，这时候就需要注意模块 B 是否提供响应的操作去除引用。

6. 单例模式

   不正确的使用单例模式是引起内存泄漏的一个常见问题，单利对象在初始化后，将在 JVM 的整个生命周期中存在（以静态变量的方式），如果单例对象持有外部的引用，那么这个对象将不能被 JVM 正常回收，导致内存泄漏。

   ```java
   class A {
       public A () {
           B.getInstance().setA(this);
       }
       ...
   }
   
   // B 采用单例模式
   class B {
       private A a;
       private static B instance = new B();
       private b (){}
       public static B getInstance(){
           return instacne;
       }
   	
   	public void setA (A a) {
           this.a = a;
   	}
   	
   	// .........
   }
   ```

   显然 B 采用 singleton 模式， 它持有一个 A 对象的引用，而这个类的对象将不能被回收。想象下如果 A 是个比较复杂的对象或者集合类型会发生什么。



## Android  中常见的内存泄漏汇总



### 集合类泄漏

集合类如果仅仅有添加元素的方法，而没有相应的删除机制，导致内存被占用。如果这个集合类是全局性的变量（比如类中的静态属性，全局性的 map 等即有静态引用或 final 一直指向它），那么没有相应的删除机制，很可能导致集合所占用的内存只增不减。比如上面的例子中就是其中一种情况，当然实际上我们在项目中肯定不会这样谢代码，但稍不注意还是很容易出现这种情况。



### 单例造成的内存泄露

由于单利的静态特性使得其生命周期跟应用一样长，所以如果使用不恰当的话，很容易造成内存泄漏。

```java
public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager(Context context) {
        this.connect = connect;
    }
    public static AppManager getInstance(Context context) {
        if (instance == null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
```

这是一个普通的单例模式，当创建这个单例的时候，由于需要传入一个 Context， 所以这个 Context 的生命周期的长短至关重要：

1. 如果此时传入的是 Application 的 Context， 因为 Application 的生命周期就是这个应用的生命周期，所以这将没有任何问题。
2. 如果此时传入的是 Activity 的 Context，当这个 Context 所对应的 Activity 退出时，由于该 Context 的应用被单例对象所持有，其生命周期等于整个应用程序的生命周期，所以当前 Activity 退出时，它的内存并不会被释放，就会造成泄漏。

**正确的方式应修改为：**

```java
public class AppManager { 
    private static AppManager instance;
    private Context context;
    private AppManager (Context context) {
        this.context = context.getApplicationContext();// 使用 Application 的 context
    }
    
    public static AppManager getInstance(Context context) {
        if (instance == null) {
            instance = new AppManager(context);
        }
        return instance;
    }
}
```

或者这样写，连 Context 都不用传进来：

```java
在你的 Application 中添加一个静态方法， getContext() 返回 Application 的 context

...

context = getApplicationContext();

...

/**
 * 获取全局的 Context
 * @return 返回全局的 context 对象
 */
 
public static Context getContext(){
    reutnr context;
}

public class AppManager {
    private static AppManager instance;
    private Context context;
    private AppManager () {
        this.context = MyApplication.getContext();// 使用 Application 的 context
    }
    
    public staitc AppManager getInstance() {
        if (instance == null) {
            instance = new AppManager();
        }
        return instance;
    }
}
```



### 匿名内部类/非静态内部类和异步线程

非静态内部类创建静态实例造成的内存泄漏

有的时候我们可能会在启动频繁的 Activity 中，为了避免重复创建相同资源，可能会出现这种写法

```java
public class MainActivity extends AppCompatActivity {
    private staitc TestResource mResource = null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if(mResource == null) {
            mResource = new TestResource();
        }
        //....
    }
        
    class TestResource {
            //....
            
    }
}
```

这样就在 Activity 内部创建了一个非静态内部类的单例，每次启动 Activity 都会使用该单利的数据，这样虽然避免了资源的重复创建，不过这种写法却造成了内存写泄漏，因为**非静态内部类默认会持有外部类的引用**，而该非静态内部类又创建了一个静态实例，该实例的生命周期和应用一样长，这就导致了该静态实例一直会持有该 Activity 的引用，导致 Activity 的内存资源不能正常回收。正确的做法：

将该内部类设为静态内部类或者将该内部类抽取出来封装成单例，如果需要使用 Context， 按照上一个方法推荐使用 Application 中的 Context。 当然 Application 的 Context 不是万能的，所以也不能随便乱用，对于有些地方则是必须使用 Activity 的 Context ，对于 Application，Service， Activity 三者的 Context 的应用场景如下：

![Application, Service, Activity 的 Context 使用场景](https://camo.githubusercontent.com/dee4aecb8a80c4e73337b56ee01cbffa2a8049dd/687474703a2f2f696d672e626c6f672e6373646e2e6e65742f32303135313132333134343232363334393f73706d3d353137362e3130303233392e626c6f67636f6e742e392e437455316334)

其中：NO1 表示 Application 和 Service 可以启动一个 Activity，不过需要创建一个新的 task 任务队列。而对于 Dialog 而言，只有在 Activity 中才能创建。



### 匿名内部类

Android 开发经常会继承实现 Activity/Fragment/View， 此时如果你使用了匿名类，并被异步线程持有，如果没有任何措施这样一定会导致泄漏

```java
public class MainActivity extends Activity {
    ...
    Runnable ref1 = new MyRunnable();
    Runnable ref2 = new Runnable (){
       @Override
       public void run() {
           ....
       }
    };
    ...
}
```



ref1 和 ref2 的区别是， ref2 使用了匿名内部类。我们看一下运行时，这两个引用的内存：

![](https://camo.githubusercontent.com/2b1a52551d828d9640f23ee7c2802476b02ccec3/687474703a2f2f696d67322e746263646e2e636e2f4c312f3436312f312f666230356666366432653638663330396239346464383433353263383161636665306165383339653f73706d3d353137362e3130303233392e626c6f67636f6e742e31302e437455316334)

可以看到 ref1 没什么特别的。

但 ref2 这个匿名类的实现对象里面多了一个引用：

this$0 这个引用指向 MainActivity.this， 也就是说当前的 MainActivity 实例会被 ref2 持有，如果将这个引用再传入一个异步线程，此线程和此 Activity 生命周期不一致的时候，就会造成内存泄漏。



###  Handler 造成的内存泄漏

Handler 的使用造成的内存泄漏问题应该说是最常见的，但很多时候我们为了避免 ANR 而不在主线程中进行耗时操作，在处理网络任务或者封装一些请求回调等 api 都借助 Handler 来处理，但 Handler 不是万能的，对于 Handler 的使用代码编写不规范就有可能造成内存泄漏。另外，我们知道 Handler、Message 和 MessageQueue 都是相互关联在一起的，万一 Handler 发送的 Message 尚未被处理，则该 Message 及发送它的 Handler 对象将被线程 MessageQueue 一直持有。

由于 Handler 属于 TLS（Thread Local Storage）变量，生命周期和 Activity 是不一致的。因此这种实现方式一般很难保证跟 View 或者 Activity 的生命周期保持一致，很容易导致无法正确释放。

```java
public class MainActivity extends Activity {
    private final Handler mLeakyHandler = new Handerl(){
        @Override
        public void handleMessage(Message msg) {
            ...
        }
    };
    
    @Override
    protected void onCreate(Bundler savedInstanceState) {
        super.onCreate(savedInstanceState);
        // post a message and delay its execution for 10 minutes.
        mLeakHandler.postDelayed(new Runnable(){
           @Override
            public void run(){
                ...
            }
        }, 1000 * 6 * 10);
        
        // go back to the previous Activity
        finish();
    }
}
```

在该例中，生命了一个延时 10 分钟执行的消息 Message， mLeakyHandler 将其 push 进了消息队列 MessageQueue 中。当 Activity 被 finish() 掉，延时任务的 Message 还会继续存在于主线程中，它持有该 Activity 的 Handler 引用，此时 finish() 掉的 Activity 就不会被回收，从而造成内存泄漏（**因 Handler 为非静态内部类，会持有外部类的引用，在这里就是 MainActivity**）.

修复方法：在 Activity 中避免使用非静态内部类，比如上面我们将 Handler 生命为静态的，则其存活期和 Activity的 生命周期无关了，同时通过弱引用的方式引入 Activity，避免直接将 Activity 作为 Context 传入， 如下：

```java
public class SampleActivity extends Activity {
    /**
    * Instence of static inner classes do not bold an implicit reference to their outer class
    */
    
    private static class MyHandler extends Handler {
        private final WeakReference<SampleActivity> mActivity;
        
        public MyHandler (SampleActivity activity) {
            mActivity = new WeakReference<SampleActivity>(activity);
        }
        
        @Override
        public void handleMessage (Message msg) {
            SampleActivity activity = mActivity.get();
            if(activity != null) {
                //....
            }
        }
    }
    
    private final MyHandler mHandler = new MyHandler(this);
    
    /**
    * Instance of anonymous classes do not hold an implicit 
    * reference to their outer class when they are "static"
    */
    private static final Runnable sRunnable = new Runnable ()	{
     	@Override
        public void run(){
            ...
        }
    };
    
    @Override
    protected void onCreate(Bundler savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // Post a message and delay its execution for 10 mintues.
        mHandler.postDelay(sRunnable, 1000 * 60 * 10);
        
        // go back to the previous Activity
        finish();
    }
    
}
```

综述， 即推荐使用静态内部类 + WeakReference 这种方式。每次使用前注意判空。

前面提到的 WeakReference，所以这里简单说下 Java 中对象的几种引用类型。

Java 对引用分为 StrongReference， SoftReference， WeakReference和 PhantomReference 四种。

| 级别                   | 回收时机     | 用途                                                         | 生存时间           |
| ---------------------- | ------------ | ------------------------------------------------------------ | ------------------ |
| 强(StrongReference)    | 从来不会     | 对象的一般状态                                               | JVM 停止运行时终止 |
| 软(SoftReference)      | 在内存不足时 | 联合 ReferenceQueue 构造有效期短/占内存大/生命周期长的对象的二级高速缓冲器(内存不足才清空) | 内存不足时终止     |
| 弱（WeakReference）    | 在垃圾回收时 | 联合 ReferenceQueue 构造有效期短/占内存大/生命周期长的对象的一级高速缓冲器(系统发生 gc 则清空) | gc 运行后终止      |
| 虚（PhantomReference） | 在垃圾回收时 | 联合 ReferenceQueue 来跟踪对象被垃圾回收器回收的活动         | gc 运行后终止      |



在 Android 应用开发中，为了防止内存溢出，在处理一些占用内存大而且生命周期较长的对象时候，可以尽量应用软引用和弱引用技术。

软/弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java 虚拟机就会把这个软引用加入到与之关联得的引用队列中。利用这个队列可以得知被回收的软/弱引用的对象列表，从而为缓冲器清楚已失效的软/弱引用。

假设我们的应用会用到大量的默认图片，比如应用中有默认的头像，默认游戏图标等等，这些图片很多地方会用到。如果每次去读取图片，由于读取文件需要硬件操作，速度很慢，会导致性能较低。所以我们考虑将图片缓存起来，需要的时候直接从内存中读取。但是由于图片占用空间比较大，缓存很多图片需要很多的内存，就可能比较容易发生 OutOfMemery 异常。这时我们可以考虑使用软/弱引用技术来避免这个问题发生。以下是高速缓冲器的雏形：

首先定义一个 HashMap，保存软引用对象：

```
private Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
```

再定义一个方法，保存 bitmap 的软引用到 HashMap。

```
public class CacheBySoftRef {
    // 先定义一个 HashMap，保存软引用对象
    private Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
    
    // 再定义一个方法，保存 bitmap 的软引用到 HashMap。
    public void addBitmapToCache(String path) {
        // 强引用的 Bitmap 对象
        Bitmap bitmap = BitmapFactory.decodeFile(path);
        // 软引用的 bitmap 对象
        SoftReference<Bitmap> softBitmap = new SoftReference<Bitmap>(bitmap);
        imageCache.put(path, softBitmap);
    }
    
    // 获取的时候，可以通过 SoftReference 的 get() 方法得到 bitmap 对象
    public Bitmap getBitmapByPath(String path) {
        // 从缓存中取软引用的 bitmap 对象
        SoftReference<Bitmap> softBitmap = imageCache.get(path);
        // 判断是否存在软引用
        if (softBitmap == null)  return;
        // 通过软引用取出 bitmap 对象，如果由于内存不足 Bitmap 被回收，将取到空，如果未被回收，则可重复使用，提高速度
        Bitmap bitmap = softBitmap.get();
        return bitmap;
    }
}
```







