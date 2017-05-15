---
title: Android 启动页优化
date: 2016-11-23 22:41:54
categories: Android
tags: 优化
---

# 问题描述
通常我们打开一个应用的时候，如果没有做过优化，会有一段时间是白屏的，时间长短取决于应用本身。而作为一个应用的入口，一段时间的白屏是影响用户体验的，要是时间长的话，用户的第一印象就不好，甚至就放弃这个应用了。为了让应用有更好的用户体验，启动页的优化是必要的，本文主要介绍如何通过页面上的效果进行优化。

现在大多数的应用在启动的时候都会有一个启动页面，大多是显示应用的图标，slogan，或者广告。这样有内容的启动页面，即使时间相对较长，用户也不会觉得难以忍受。这是从视觉效果上进行的”优化“。下面提供三种方法进行优化。

# 解决方案
## 设置 SplashActivity
对于新建 Activity , 有以下两种解决方法。
 ### 1. 为 SplashActivity 设置自定义背景

 ```xml
 <style name="SplashStyle" parent="AppTheme">
        <item name="android:windowBackground">@drawable/splash_bg</item>
 </style>
 ```
为 SplashActivity 设置 theme
```xml
<activity
                android:name=".ui.activity.SplashActivity"
                android:theme="@style/SplashStyle">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
</activity>
```

这样在应用启动的时候便显示设置的背景资源，实现应用秒开

这个方法主要试用于启动页需要有其他显示的场景，比如需要设置动画效果，显示广告
 ### 2. 为 SplashActivity 设置透明背景

```xml  
 <style name="SplashStyle"
        parent="android:Theme.Translucent.NoTitleBar.Fullscreen">
 </style>
 ```
为 SplashActivity 设置透明主题以后，点击应用图标会有短时间延迟，然后界面一下就出来，会让用户认为是手机卡顿，这个锅就让手机厂商去背吧。

## 不设置 SplashActivity
如果只是为了显示应用图标，大可不必新建一个 Activity，同样的，设置好我们需要的 Style，然后为需要启动的 Activity 设置自定义主题。** 这里需要注意的是，为需要启动的 Activity 设置主题而不是 Application。**

当 Activity 正常启动以后，要切回正常的主题只需要在 `onCreate()` 函数里面调用 `setTheme()` 便可，** 必须在 super.onCreate 之前调用。 **
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
    	// 必须在 super.onCreate 之前调用
    	setTheme(R.style.APPTheme);
        super.onCreate(savedInstanceState);
        // ...
        
    }
 }
```

这个方法的优点在于
1. 不用新建 SplashActivity, 不用设置等待时间
2. 没有额外的性能损耗
3. 启动速度更快

# 总结
几种方式没有好坏之分，主要看业务需求，新建 Activity 主要为了展示产品的其他内容，而如果只是想显示产品图标，就不用新建一个 Activity。当然，作为开发者，我们还应该从应用的入口进行优化，减少优化不必要的耗时操作。