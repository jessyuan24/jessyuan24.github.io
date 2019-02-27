---
layout: post
title:  "Android NDK开发中引入OpenCV库"
date:   2019-02-18 12:00:00 +0800
categories: Android
tags: android, opencv
---
Android NDK开发中，我们作图片处理时，运用到OpenCV第三方库，这时需要倒入OpenCV SDK到Ａndroid Studio项目中.

想了解更多代码，可以查看我的[github][github]代码

如下是导入OpenCV到Android Studio项目的步骤。  （如果麻烦，直接把我项目的module导入你的项目中，修改CMake文件和gradle）
1. 在OpenCV官方上下载opencv android sdk包。https://opencv.org/releases.html
2. 在Android Studio创建新的module  
Library name: ```OpenCV```  
Module name: ```opencv```  
Package name: ```org.opencv```   
3. 复制```opencv_sdk_path/sdk/java/src```下的所有文件到```your_project_path/opencv/src/main/java```目录下
4. 在main的目录下创建一个路径目录```aidl/org/opencv/engine```，以及把```java/org/opencv/engine/OpencOpenCVEngineInterface.aidl```文件移动到刚创建的目录下
5. 复制资源文件， 复制```opencv_sdk_path/sdk/java/res```下所有文件到```your_project_path/opencv/src/main/res```目录下
6. 在```your_project_opencv/src```目录下创建```sdk```目录，并把```opencv_sdk_path/sdk/native```所有文件复制到刚创建的目录下
7. 在opencv module下创建CMakeLists.txt文件，并添加如下代码
```
cmake_minimum_required(VERSION 3.4.1)

set(OpenCV_DIR "src/sdk/native/jni")
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
include_directories(${OpenCV_INCLUDE_DIRS})
```
8. 修改```opencv module```的```build.gradle```文件，如下
```
android {
    compileSdkVersion 28
    
    defaultConfig {
        minSdkVersion 19
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

        externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
                // 根据你电脑配置添加或删除这些类型
                abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
            }
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
    sourceSets {
        main {
            jni.srcDirs = [jni.srcDirs, 'src/sdk/native/jni/include']
            jniLibs.srcDirs = [jniLibs.srcDirs, 'src/sdk/native/3rdparty/libs', 'src/sdk/native/libs']
        }
    }

}android {
    compileSdkVersion 28

    defaultConfig {
        minSdkVersion 19
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

        externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
                // 根据你电脑配置添加或删除这些类型
                abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
            }
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
    sourceSets {
        main {
            jni.srcDirs = [jni.srcDirs, 'src/sdk/native/jni/include']
            jniLibs.srcDirs = [jniLibs.srcDirs, 'src/sdk/native/3rdparty/libs', 'src/sdk/native/libs']
        }
    }

}
```
9. 在```app module```同样创建```CMakeLists.txt```文件，添加如下代码
```
cmake_minimum_required(VERSION 3.4.1)

add_library(native-lib
        SHARED
        native-lib.cpp)


# Include libraries needed for hello-jni lib
target_link_libraries(native-lib
        android
        log)

set(OpenCV_DIR "../../../../opencv/src/sdk/native/jni") #根据CMakeLists.txt路径修改(相对目录)
find_package(OpenCV REQUIRED)
message(STATUS "OpenCV libraries: ${OpenCV_LIBS}")
target_link_libraries(native-lib
        ${OpenCV_LIBS})
```
10. 修改```app module```的```build.gradle```文件，如下
```
android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.coldwizards.nativedemo"
        minSdkVersion 21
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags "-frtti -fexceptions"
                // 根据你电脑配置添加或删除这些类型
                abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
            }
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    externalNativeBuild {
        cmake {
            version '3.10.2'
            path "src/main/cpp/CMakeLists.txt"
        }
    }
}
```
11.  同步gradle。在C++文件中是否可以include```<opencv2/opencv.hpp>```
![inclue opencv](/images/include_opencv.png)
13.  如果打包完成，在分析apk中可以查看到opencv的so文件
![libopencv](/images/libopencv.png)
步骤完成。

如有什么问题可以在github上提问我。


[github]: https://github.com/jessyuan24/opencv-for-android