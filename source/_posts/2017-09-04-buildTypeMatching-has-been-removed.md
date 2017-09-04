---
title: buildTypeMatching-has-been-removed
date: 2017-09-04 17:16:05
categories: Android plugin
tags: 
    - Android plugin
    - Gradle
---

# 升级到Android Studio 3.0 Beta4

## 发现问题

当你升级到Android Studio 3.0 Beta4的时候一般情况下会报错的，我遇到如下信息：

```
Error:(115, 1) A problem occurred evaluating project ':app'.
> buildTypeMatching has been removed. Use buildTypes.<name>.fallbacks ...
```

<!--  more -->

## 分析找资料
百度一搜发现压根没有，可能是因为Beta4是这几天才发布的原因吧。Google一下也没找到什么有的信息，就想啊android开发者网站的[migrat](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html)应该是不错的参考，还真找到了，官方解释如下：

```
Android plugin 3.0.0-beta4 and higher include DSL elements to help control how the plugin should resolve situations in which a direct variant match between an app and a dependency is not possible. The table below helps you determine which DSL property you should use to resolve certain build errors related to variant-aware dependency matching.
```

大概的意思就是Android plugin 3.0.0-beta4 以后buildTypeMatching不能用啦，你应该为你自定义的buildType里面给定如果其他库里面没有这个buildType的话应该按什么顺序查找。注意：这里是一个数组，可以顶意思查找顺序的。

## 解决啦
我在app module 中定义了dev(buildType),然后我就用
```
    matchingFallbacks = ["dev", "release", "debug"]
```
来解决了问题，意思就是指定安装dev到release的顺序查找buildType。匹配到了就会break哦。到此问题就解决了。。。