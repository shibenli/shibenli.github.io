---
title: android-library-aar
date: 2018-04-12 13:14:08
categories: Android
tags:
    - Android
    - aar
    - consumerProguardFiles
---
项目中不可避免的要映入第三方库，Google的aar方式的库推出的确带来了很多的方便，但是同时还有一些局限性需要注意，这里记录一下在使用aar过程中遇到的问题。

### 1、consumerProguardFiles功能的使用

项目设置混淆的时候，就必要要考虑到第三方库的混淆规则。曾经的jar方式引入的年代，都是在项目中配置使用到的代码的混淆规则，或者是干脆不混淆一个不确定规则的库。但是在aar引入的后期，带来了consumerProguardFiles功能，实现了再库中配置自己的混淆规则，项目就可以不用关心三方库的混淆规则。

<!-- more -->

![consumerProguardFiles](/images/2018/aar-consumer-proguard-file-20180412134308.png)

上图展示了consumerProguardFiles的使用，其实这里还是有个隐藏的技巧点： 

`fileTree(dir: './', include: '*.pro').asList().toArray()`

，表示了查找当前module目录下面的以“.pro”结尾的文件。这里默认.pro文件都是混淆配置文件，因为当前库可能会引用其他库，这种循环遍历的方式极大的增强了灵活性。但是这对混淆配置文件提出了一些要求。

- 不能使用include 关键字。库工程最终是会合并库里的规则，但是include关键字是不会在合并的时候生效，这就导致库打包的时候报混淆规则文件找不到的错误。
- “-ignorewarnings” 关于混淆不得不说的配置，默认情况下混淆的warnings会阻断编译。

### 2、SDK开发过程中依赖外部aar的处理

我遇到的一个问题是需要在第三方人脸识别和活体检测基础上实现自己的SDK封装，这就涉及到在自己的SDK中依赖第三方库，但是情况还能更糟一点：对方是以aar方式发送的文件（理由是只想发给合作方）。都知道aar要么使用maven仓库依赖，要么使用provide方式依赖，但是provide的方式很不利于升级，更新一下第三方库要同时更新SDK和项目（主要是aar无法把另一个aar打入包中），我还遇到两个版本更新不同步导致的莫名其妙的bug，同步更新就解决了。当然这里是要解决这个痛点的，模拟maven目录结构（没有代码无法直接发到maven私服中），使用"@aar"的方式引用。

` compile 'com.other:moxie-client:1.3.6.4@aar' `

这种引用的方式其实就需要注意目录和版本的对应关系就可以了，当然groupId就可以自定义了（建议是第三方和自己开发的分开管理）

![aar-nexus](/images/2018/aar-nexus-20180412154341.png)

对照着上图注意如下规则
- groupId可以任意，尽量不要和项目中使用的重复；
- projectId就是区分SDK功能的，最好是见名知意；
- 版本号要严格按照目录和对应目录下的文件名后缀对应；