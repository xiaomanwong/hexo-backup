---
title: Jenkins + gradle + git 构建 Android CI 构建环境
date: 2019-04-16 17:42:05
tags:
---

# 前言

在项目开发中，我们需要将最新的代码更新，提供给测试人员进行测试，以及发布。

目前 Android 工作中都在使用很强大的开发、构建以及打包工具，例如： android-studio、Gradle、Git等。

然，在企业组织并不是很完善的公司里，开发打包发布等工作，时常会由开发人员进行操作，难免在一些地方疏忽掉。

因此，CI 构建的出现，使得这些繁琐的工作变得轻松起来。

对于开发工程师，只负责向版本库提交代码，不用关心打包，发布之类的流程。

对于产品和测试，只需要从发布页面下载 APK 安装文件，不需要每一次都去工程师哪里索取最新的安装文件。

CI 的基本工作流程如下：
![图1](http://upload-images.jianshu.io/upload_images/1550996-5101eac8d87352a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们每一次提交代码（通过git/svn作为版本库）到主干上，根据 CI 的定时任务，检测到版本更新，通过 CI ，将进行打包发布等流程操作。

# 准备工作

本文使用 Linux Ubuntu 系统为大家介绍环境的搭建

## 环境工具
    1. PC 机(mac/linux)
    2. Java JDK
    3. Android SDK
    4. Gradle
    5. Git
    6. Tomcat
    7. Jenkins

## 环境搭建

###  Java 环境 安装
Java JDK, Android SDK, Gradle 可从[AndroidDevTools](http://androiddevtools.cn)处下载获取。

### git 安装
git 可通过终端进行安装
```
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
```

安装完成之后通过`git --version` 检查是否成功

安装后 git 存储在 `/usr/bin/git`下

### Jenkins 

通过 Jenkins [官方网站](https://jenkins.io/index.html)下载最新Jenkins.war包

![Jenkins 下载](http://upload-images.jianshu.io/upload_images/1550996-d2654c0a1383423a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 环境变量

打开 `vi /etc/profile`

将下列语句添加在文件的末尾后， 执行 esc->:wq

其中环境位置根据自己的所在位置进行相应的更改

![环境变量](http://upload-images.jianshu.io/upload_images/1550996-ffd5f1b87d0e5bbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 启动

激动的你，是不是已经被这些繁琐的东西搞的不耐烦了呢，下面我们开始启动 Jenkins

将下载好的 Jenkins.war 包， 放入 Tomcat 的 webapps 目录下，进入 bin 目录执行 ./startup.sh 启动 Tomcat。

启动后，在浏览器中输入： `localhost:8080/jenkins`


![jenkinsmain.png](http://upload-images.jianshu.io/upload_images/1550996-cbdb39d4f2f4a85f.png?
imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 插件安装

系统管理-> 插件管理->可选插件：

在搜索框中搜索以下插件，并进行安装

git plugin
gitlab plugin
grade plugin
Android Lint Plugin
Build Pipeline plugin
build timeout plugin
build name plugin
change assembly-version plugin
credentials binding plugin
description setter plugin
Dynamic parameter plugin
Email Extension plugin
FindBugs plugin
JaCoco plugin
Unit attachments plugin
Project Description plugin
Timestamper
Workspace cleanup plugin

安装完成后，重启。

### 系统设置

系统管理->系统设置：

配置Android 环境，将地址指向本机的 SDK 目录
![Android environment](http://upload-images.jianshu.io/upload_images/1550996-5987aabcffb290c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

系统管理->全局工具配置

配置 Java， Git ， Gradle 目录等

JDK：

![Java JDK](http://upload-images.jianshu.io/upload_images/1550996-b9e29a1ec2b746e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Git：

![git](http://upload-images.jianshu.io/upload_images/1550996-9c4e88d64fcc2ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Gradle：


![gradle](http://upload-images.jianshu.io/upload_images/1550996-9fbabc7d5c7ca319.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到此，环境配置，已基本完成。

下面我们开始进行项目够将操作

## 项目构建

### 创建Job

新建->构建一个自由风格的软件项目:

![创建CI工程](http://upload-images.jianshu.io/upload_images/1550996-2390d86256678c54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 参数化构建

通常我们在使用 Android-studio 进行打包时以及签名时，都会用到build.gradle并在其中配置相关属性。再此，我们可以用Jenkins，配置我们的项目参数，例如发布的版本号，构建时间， 上传路径，发布地址，签名打包等等。

在这里我们先看看 build.gradle 中的构建信息:

在项目的 moudle 下 build.gradle 文件
```
def getDate () {
    def date = new Date()
    def formattedDate = date.format("yyyyMMddHHmm")
    return formattedDate
}

def verName = APP_VERSION
def verCode = 14

android {
    ....
    signingConfigs {
        release {
          keyAlias ''
          keyPassword ''
          storeFile file ('')
          storePassword ''
       }   
   }

  defaultConfig {
      applicationId "cn.zhuangbudong.example"
      minSdkVersion 18
      targetSdkVersion 25
      multiDexEnabled true
      versionCode verCode
      versionName verName

      resValues("string", 'app_version', verName)
  }
  buildTypes {
    release {
      signingConfig signingConfigs.release
      minifyEnabled false
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }

   lintOptions {
    abortOnError false
   }

   dexOptions {
      javaMaxHeapSize '2g'
   }

applicationVariants.all { variant ->
    variant.outputs.each { output ->
        def newName
        def timeNow
        def oldFile = output.outputFile
        def outDirectory = oldFile.parent
        if ("true".equals(IS_JENKINS)) { 
            timeNow = JENKINS_TIME
            outDirectory = "/media/nexd/work/android/package/release/"
            newName = 'zhuangbudong_example_' + verName + "_" + timeNow + "_" + variant.buildType.name + ".apk" 
       } else {
            timeNow = getDate()
            if (variant.buildType.name.equals('debug')) { 
               newName = 'zhuangbudong_example_' + verName + "_debug.apk"  
          } else { 
               newName = 'zhuangbudong_example_' + verName + "_" + timeNow + "_" + variant.buildType.name + ".apk" 
           } 
       } 
       output.outputFile = new File(outDirectory, newName)
    }}

  }
}
```

gradle.properties:

```
APP_VERSION=2.0.2
IS_JENKINS=false
JENKINS_TIME=''
```

在工程中添加以上代码，并在Jenkins中为这些参数赋值。

下面介绍 Jenkins 参数配置

勾选参数化构建过程，如下图：
![参数化构建](http://upload-images.jianshu.io/upload_images/1550996-20f69e63a527814c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照下图，添加相关类型的参数，此处注意，Jenkins 配置的参数名要和在android-studio中配置的参数名保持一致

![JENKINS_TIME](http://upload-images.jianshu.io/upload_images/1550996-e0f4ceb7015ac0da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![APP_VERSION](http://upload-images.jianshu.io/upload_images/1550996-6e3f8d0311399d74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![IS_JENKINS](http://upload-images.jianshu.io/upload_images/1550996-673c2465bcdc9a89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![ENVIRONMENT](http://upload-images.jianshu.io/upload_images/1550996-a82221191b980ad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 源码管理

此处负责从版本库中拉去最新的代码


![git 仓库](http://upload-images.jianshu.io/upload_images/1550996-6dcbd8dd2dd9456c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此处如果需要验证，点击 Add， 选择： Username with password

在对应窗口输入用户名和密码信息
![用户身份验证](http://upload-images.jianshu.io/upload_images/1550996-8af978712a010864.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击添加。

使用 gitlab 进行源码库管理。

### 触发器

触发器负责拉取代码，编译，打包，发布等操作。通过触发器，执行Jenkins。


![触发器构建](http://upload-images.jianshu.io/upload_images/1550996-beb6ff8faa9843f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 构建环境

此处只是在Jenkins在打包时，配置任务名称即可。如下图：

![构建环境](http://upload-images.jianshu.io/upload_images/1550996-45b210c5df916dbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 构建

这里是最重要滴，配置以下命令，才能进行打包签名等等。

如下配置，这里需要注意下，构建文件，根目录。在你的目录结构比较复杂的时候，即你的根目录没有 build.gradle 文件时，需要指定一下 build.gradle 目录的位置。

同时，也是最重要的，勾选上pass job parameters as gradle properties ，不然之前配置的参数无法传递给项目中的 gradle.properties。

![构建](http://upload-images.jianshu.io/upload_images/1550996-4e4e6e139fc1e856.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 构建后操作

当项目构建完成后，我们可以通过邮件的方式将产生的Apk文件，以及测试报告，构建日志等信息，发送出来 如下图：


![邮件发送](http://upload-images.jianshu.io/upload_images/1550996-63b6bcb7b2dfd551.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
<hr/>
(本邮件是程序自动下发的，请勿回复！)<br/><hr/>
项目名称：${PROJECT_NAME}<br/><hr/>
构建编号：${BUILD_NUMBER}<br/><hr/>
构建状态：${BUILD_STATUS}<br/><hr/>
触发原因：${CAUSE}<br/><hr/>
测试报告：<a href="${PROJECT_URL}ws/${PROJECT_NAME}app/build/reports/tests/release/index.html">${PROJECT_URL}ws/${PROJECT_NAME}app/build/reports/tests/release/index.html</a><br/><hr/>
构建日志地址：<a href="${BUILD_URL}console">${BUILD_URL}console/</a><br/><hr/>
构建地址：<a href="${PROJECT_URL}">${PROJECT_URL}</a><br/><hr/>
构建报告：<a href="${BUILD_URL}testReport">${BUILD_URL}testReport</a><br/><hr/>
变更集:${JELLY_SCRIPT,template="html"}<br/>

<hr/>
```

## 开始构建

回到 Jenkins 首页，点击创建的项目，点击 build with parameters:


![Build with Parameters](http://upload-images.jianshu.io/upload_images/1550996-58ecc4965ab845ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击开始构建，启动 Jenkins 构建任务。

构建成功时，显示为蓝色， 失败为红色，如下图：


![构建结果](http://upload-images.jianshu.io/upload_images/1550996-4b2fc4b1190ddcd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

构建后生成的 Apk 文件，存在 build.gradle 文件中配置的目录。同时也可以使用蒲公英或fir.im 进行发布管理。

谢谢~