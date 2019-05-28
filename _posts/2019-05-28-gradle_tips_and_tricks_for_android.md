---
layout: post
title:  "技巧地使用Gradle"
date:   2019-05-28 12:00:00 +0800
categories: android
tags: android, kotlin
---
# 技巧地使用Gradle

## Android编译过程

Android Studio是用gradle编译Android项目的代码和资源。  
我们先来了解下编译Android应用的过程

![编译Android应用过程](/images/gradle-tips-tricks-build-process-image.png)

过程如下:

1. 首先源码会转化为dex文件，Dex文件包含运用应用需要的字节码，dex文件就是.class文件
2. 这些dex文件和资源文件相结合构建成APK包
3. APK包会使用debug或release版本的keystore来签名
4. 如果编译的app用于测试和分析，Apk packager会使用debug的keystore签名生成APK包
5. 如果编译的app是用于发布的版本，Apk packager会使用release的keystore签名生成APK包
6. 在生成APK包之前，Apk packager会优化App和删除没引用的代码

完成这些过程后，会得到一个可运用的APK包

## 配置文件

当创建一个新的Android Studio项目时，会自动创建一些文件，下面我们先了解下Android项目的目录结构

![ANdroid项目的目录结构](/images/gradle-tips-tricks-project-structure-image.png)

* Gradle setting文件: setting.gradle是在根目录下的文件，它包含App所需的modules
* Top-level的build文件: top-level build.gradle是也是在根目录下的文件，它是一个全局配置文件，如果配置用于全部的modules，则可以在此文件配置
* Module-level的build文件: module-level build.gradle是在project/module目录下的文件，它的配置只对当前目录的module有效
* Gradle properties文件: 该文件下的配置用于gradle

## 技巧地使用Gradle

下面列举一些使用gradle的使用技巧

1. 添加构建依赖:  
   Android项目可以添加多个module或library作为依赖，此时可以module-level的build文件中添加

   ```gradle
   apply plugin: 'com.android.application'

    android { ... }

    dependencies {
        // 添加本地库
        implementation project(":mylibrary")

        // 添加本地二进制库
        implementation fileTree(dir: 'libs', include: ['*.jar'])

        // 添加远程二进制库
        implementation 'com.example.android:app-magic:12.3'
    }
   ```

2. 管理依赖的版本:  
   在有几个module项目中，有很多依赖库，最麻烦的是有很多依赖库版本号冲突，类似于如下的build.gradle文件

   ```gradle
    apply plugin:'com.android.application'

    android{
        defaultConfig{...}
        buildTypes{...}
        ProductFlavours{...}
    }

    dependencies{
        //android support库
        implementation'com.android.support:appcompat-v7:28.0.0'
        implementation'com.android.support:design:28.0.0'
        implementation'com.android.support:cardview-v7:28.0.0' 

        //其他的依赖库
    }
   ```

   这样很难管理依赖库的版本，解决的办法是在gradle的ext中定义变量值

   ```gradle
    apply plugin: 'com.android.application'
            
    android {
        defaultConfig{...}
        buildTypes{...}
        ProductFlavours{...}
    }

    ext {
        supportLibraryVersion = '28.0.0'
    }

    dependencies {
        //android support库
        implementation "com.android.support:appcompat-v7:$supportLibraryVersion"
        implementation "com.android.support:design:$supportLibraryVersion"
        implementation "com.android.support:cardview-v7:$supportLibraryVersion"

        //其他库
    }
   ```

   如果作用于多个module项目中，可以在top-level的build文件中定义变量值

   ```gradle
    buildscript {
    }

    allprojects {
    }

    ext {
        supportLibraryVersion = '28.0.0'
    }
   ```

   在module-level文件中添加

   ```gradle
    com.android.support:appcompatv7:$rootProject.supportLibraryVersion
   ```

3. 加快编译速度:  
   有时我们编译项目，会花费很多时间在编译中，大大降低了开发过程速度。现在我们忽略硬件设备来提高编译速度。

   * 实时更新Android Studio。每次Android Studio更新，都会优化工具和加快编译速度
   * 避免编译一些暂时不需要的资源文件，可以在module-level的build文件中定义flavor来指定某些资源文件

   ```gradle
    android {
      ...
      productFlavors {
        dev {
          ...
          // 这里指定使用xxhdpi的资源文件
          resConfigs "xxhdpi"
        }
        ...
      }
    }
   ```

   * 如果不使用Crashlytics报告，可以停止使用，加快编译速度

   ```gradle
    android {
        ...

        buildTypes {
            debug {
                ext.enableCrashlytics = false
            }
        }
    }
   ```

   * 避免使用动态的版本号来管理依赖库，类似`'com.android.tools.build:gradle:2.+'`，这样会每次编译都会检查最新版本，之后下载更新，使得编译更慢

   * 如果你网络本身慢的，也会影响编译速度，gradle每次编译会尝试使用网络资源的依赖库，因此使用离线模式(offline model)。这个可以在首选项(Preference)->Build->Gradle中设置，这里不详细写。

4. 优化和瘦身App
   如果使你的App更小，则你可以使用shrinking来瘦身的App，它会删除不使用的代码和资源文件。很多时候，我们只是使用依赖库的某些功能我，而不需要加载整个依赖库，我们只保留使用的功能。ProGuard可以做到瘦身优化，减少不使用的我代码和资源文件。可以阅读如下资源
   * [官方的ProGuard介绍](https://developer.android.com/studio/build/shrink-code)
   * [Android ProGuard详解](https://www.cnblogs.com/cr330326/p/5534915.html)

## 总结
如果我们掌握Gradle的开发技巧，可以极大提升编译速度，不用浪费更多的时间在等待中，同时Gradle也是很好的构建编译工具，需要了解更多可以阅读[官方的Gradle文档](https://developer.android.com/studio/build)