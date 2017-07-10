---
title: 代码Review系列之3-Debug
date: 2017-07-10 11:03:19
categories: Android
tags:
    - Android
    - 代码Review
---

这里要说的不是怎么去debug，我相信那很多人已经会了。这里要说的怎么利用一些好的工具，配合合理的配置，实现高效的debug。

## Debug配置
![Debug相关配置](/images/2017/Debug相关配置.png)

<!-- more --> 

![Debug相关配置](/images/2017/Debug相关配置2.png)

这里通过在buildType为debug的情况下这是相应的DebugApp，初始化只有在开发debug的时候才进行的操作，比方说使用square的内存泄露检测工具[LeakCanary](https://github.com/square/leakcanary)。[Stetho](https://github.com/facebook/stetho)是facebook的一个能够在在Chrome上调试Android网络、数据库和webview的强大工具，他甚至能够直接读取webview的cookie、session等字段，还能查看js报错。他们都值得你拥有...

## Gradle参考代码
```
    debugCompile 'com.facebook.stetho:stetho-okhttp3:1.4.1'
    debugCompile 'com.facebook.stetho:stetho-js-rhino:1.4.1'
    debugCompile 'com.facebook.stetho:stetho:1.4.1'
    debugCompile 'com.squareup.okhttp3:okhttp:3.4.1'
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4-beta2'
```