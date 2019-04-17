---
title: Android Studio Dependencies Library Resolve
date: 2019-04-16 17:44:39
tags: Android
---

在使用 Android Studio 开发时,使用 Gradle 脚本构建项目, 同 Maven 一样,所引用的包之间也存在着相互依赖的关系, 当你使用某个包之后,发现有包版本冲突, 那么解决方案就来了.


先说点不正经的:
1.  你可以把你自己引入的包去掉,使用依赖包
2.  放弃治疗


哈哈,言归正传:

当我们引入的包之间存在冲突(不是同一个)的关系时, 也就是说,我们需要保留一个项目依赖包使用,那么我们需要在 `build.gradle` 中将我们不需要的包删除掉.

## 举个栗子:

当我使用 `com.squareup.retrofit2:adapter-rxjava:2.1.0` 时, 它默认依赖使用 `RxJava 1.5.0` 版本. 当我使用 `io.reactivex.rxjava2:rxjava:2.0.6` 时, 就会引起包冲突.

## 解决方案:

build.gradle

```

...
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')

    compile ('com.squareup.retrofit2:adapter-rxjava:2.1.0'){
        exclude group: 'io.reactivex'
    }
    compile 'io.reactivex.rxjava2:rxjava:2.0.6'
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
}
```

