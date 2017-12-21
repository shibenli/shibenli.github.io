---
title: useragentswitcher
date: 2017-12-13 09:19:35
categories: H5
tags: 
    - UA
    - UserAgentSwitcher
---

![useragentswitcher](/images/2017/header_p.png)

UserAgent（以下简称UA）通常被用作区分客户端来源或版本、系统版本等一系列信息，常见的浏览某些微信的页面提示“只能在微信中打卡”或者“请使用微信版本高于xxx打开”，这里都是UA匹配的功劳。最常见的就是根据UA确定是手机版还是PC的，然后跳转到对于的网页（或提示某一个不可用）。

相信大家在做H5和移动app混合开发的时候，经常有被userAgent困扰的时候(其实我是一个移动开发者，经常遇到h5开发人员测试个UA就要我们单独打个包，没调好的时候还要动不动再来找你几次)。这里要介绍的是一款H5开发调试工具[useragentswitcher](https://useragentswitcher.org/)，它是一个可以修改浏览器的扩展程序，下次再要关于UA的配合你就直接把这个浏览器扩展扔给他，然后说：“工具给你找好了，自己玩去！！！”

### 优点：
  - 能够完整的自定义UA，可以预定义多个随时切换；
  - 兼容性好，兼容Chrome、Opera、Firefox浏览器
  - 工具栏快速换、恢复默认，还预定义了一些常用的UA

<!-- more -->

### 缺点：
  - 等你去发现

当然你也可以把UA设置成手机的，然后在电脑浏览手机版本的网页（谁会这么吃饱了没事干呢(*￣︶￣)），最后上一张图，有图有真相。
![useragentswitcher](/images/2017/20171214144942.png)