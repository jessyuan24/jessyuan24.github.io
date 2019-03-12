---
layout: post
title:  "使用Kotlin的Extension快速打开Activity页面"
date:   2019-03-12 12:00:00 +0800
categories: android
tags: android, kotlin
---
我们传统打开Activity页面，通过创建```Intent```对象，并传值给```startActivity()```方法。

传统方法: 
```kotlin
val intent = Intent(activity, NextActivity::class)
activity.startActivity(intent)
```
如果需要传多个参数时:
```kotlin
val intent = Intent(activity, NextActivity::class)
intent.putExtra("int", 1)
intent.putExtra("string", "str")
activity.startActivity(intent)
```

现在，我们用Kotlin的Extension方法来简单，快捷打开Activity页面
```kotlin
inline fun <reified T : Any> Activity.launchActivity (
        requestCode: Int = -1,
        options: Bundle? = null,
        noinline init: Intent.() -> Unit = {}) 
{
    val intent = newIntent<T>(this)
    intent.init()
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) 
    {
        startActivityForResult(intent, requestCode, options)
    } else {
        startActivityForResult(intent, requestCode)
    }
}

inline fun <reified T : Any> Context.launchActivity (
        options: Bundle? = null,
        noinline init: Intent.() -> Unit = {}) 
{
    val intent = newIntent<T>(this)
    intent.init()
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) 
    {
        startActivity(intent, options)
    } else {
        startActivity(intent)
    }
}

inline fun <reified T : Any> newIntent(context: Context): Intent =
        Intent(context, T::class.java)
```
使用方法
```kotlin
// 简单使用
launchActivity<NextActivity>()

// 添加传递参数
launchActivity<NextActivity> {
    putExtra("int", 1)
}

// 添加flag
launchActivity<NextActivity> {
    putExtra("int", 1)
    addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)
}

// 添加Transistions
val options = ActivityOptionsCompat.makeSceneTransitionAnimation(this, avatar, "avatar")
launchActivity<NextActivity>(options = options) {
    putExtra("int", 1)
}

// 用startActivityForResult()启动
launchActivity<NextActivity>(requestCode = 1234) {
    putExtra("int", 1)
}
```
