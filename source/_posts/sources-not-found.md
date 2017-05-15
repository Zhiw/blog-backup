---
title: Android Studio 无法查看源码
date: 2016-08-26 09:14:27
tags: Android Studio
categories: Android
---

最近在使用最新的 Android Studio 2.1 版本的时候，发现无法查看源码，提示 “Sources for Android API 23 not found”，但事实上 SDK 已经下载了对应的源码。网上搜索以后，找到了对应的解决方法。
首先找到 Android Studio 里关于 Android SDK 的设置界面，然后点击`Edit`，然后点击`Next`，然后跟着接下来的导航一直点，完成以后等待刷新一下，就可以查看源码了。
![SDK](http://7xrac3.com1.z0.glb.clouddn.com/Android-Studio-SDK.png)
之前查到的方法是通过修改`jdk.table.xml`文件，但是我修改以后没有作用，设置SDK这个方法亲测有用。

参考资料
- [Sources for Android API 23 Platfrom not found (Android Studio 2.0)](http://stackoverflow.com/questions/36814755/sources-for-android-api-23-platfrom-not-found-android-studio-2-0)
