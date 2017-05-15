---
title: 解决 Button 和 TabLayout 英文自动大写的问题
date: 2016-11-25 14:58:26
categories: Android
tags: Tips
---

在使用 Button 和 TabLayout 时，英文字体会自动变成大写，查看 Android 源码发现，TabLayout 的默认属性中设置了 `textAllCaps` 为 `true` 

`TabLayout.java`
```java
mTabTextAppearance = a.getResourceId(R.styleable.TabLayout_tabTextAppearance,
                R.style.TextAppearance_Design_Tab);
```
`styles.xml`

```xml
<style name="TextAppearance.Design.Tab" parent="TextAppearance.AppCompat.Button">
        <item name="android:textSize">@dimen/design_tab_text_size</item>
        <item name="android:textColor">?android:textColorSecondary</item>
        <item name="textAllCaps">true</item>
</style>
```

解决方法很简单，为 TabLayout 设置如下：
```xml
app:tabTextAppearance="@android:style/TextAppearance.Widget.TabWidget"
```

对于 Button 也就很简单了
```xml
 android:textAllCaps="false"
```