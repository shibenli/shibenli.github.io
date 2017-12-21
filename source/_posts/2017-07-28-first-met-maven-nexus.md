---
title: 初识maven-nexus
date: 2017-07-28 10:22:51
categories: Maven
tags:
    - Maven
    - Nexus
---
## What
Nexus,通俗的来讲，是配合maven使用的仓库管理器，是避免资料外泄的私服仓库管理器。当多个项目存在多个相同的依赖或者工具类时候，便可以将相同的依赖或者工具类，放置在私服中，组成公司内部局域网。
## Why
 我们为什么要用这个？当然是本着有轮子用轮子，没轮子改轮子的原则，能复制，就不要自己敲的原则，当有多个项目需要用到某一个依赖库或者工具类的时候，直接在gradle添加依赖即可，不必将代码Ctrl c + Ctrl v 重复操作，极大的减少的项目的代码量。

<!-- more -->

## How

### 1.下载与安装

-   连接地址：[点我下载](https://www.sonatype.com/download-oss-sonatype)

-   解压之后出现如下两个文件夹（本人使用的是版本3.4，目前为最新版本）
![nuxus全家福](/images/2017/nuxus全家福.png)

-   其中，nexus程序就保存在第一个文件夹中，在work目录下就是工作空间了
启动方式：**在nexus-x-x/bin目录中调出命令窗口，输入 net start nexus 即可启动服务。**
验证方式：在浏览器中输入**http://localhost:8081**显示如下图所示即表示搭建成功。默认初始登录账号为 admin,默认初始密码为 admin123。
![nuxus后台](/images/2017/nuxus后台.jpg)

-   用户可以自己修改主机地址和端口号（配置文件位于nexus-x-x/etc/nexus）
![nuxus配置](/images/2017/nuxus配置.png)

### 2.创建仓库

-   在登录之后，设置>repositories>create repository 创建新的仓库，其他主要参数如图所示：
![nuxus仓库创建](/images/2017/nuxus仓库创建.png)

-   创建成功之后再主页能够看到自己创建的仓库
![nuxus创建仓库结果](/images/2017/nuxus创建仓库结果.png)

### 3.与AS关联，并将依赖库上传到仓库

-   为了方便管理，在**需要上传的依赖库**中创建一个新gradle文件进行配置如下：
![AS中配置gradle打包上传脚本](/images/2017/AS中配置gradle打包上传脚本.png)

-   下图是gradle.properties的配置信息：
![AS中gradle常量配置](/images/2017/AS中gradle常量配置.png)

-   要让自己配置的gradle起作用，需要在**依赖库自身的gradle**中配置关联如下代码：
```
applyfrom:'./upload_archives.gradle' 
```

-   接下来就是上传操作了,已经检验操作了（在自己创建的仓库中找得到上传的项目就表示成功），看图说话：
![AS运行脚本uploadArchives上传](/images/2017/AS运行脚本uploadArchives上传.png)
右键uploadArchives->run开始执行该task，当提示success的时候可以去查看后台是否上传成功。一般应该能看到如下图所示
![nuxus上传成果展示](/images/2017/nuxus上传成果展示.png)

### 4.使用

-   当一切准备妥当之后，就是用法了。**前提，**在工程目录下中的gradle添加maven引用：
```
allprojects {
    repositories {
        jcenter()

        maven{url 'http://localhost:8081/repository/wyw/'}
    }
}
```
-   （依赖地址为之前创建的地址，实际开发中，将localhost换成服务器地址），就和普通的依赖添加一样操作，在需要使用该库的module中添加：
```
compile'common:toastutils-lib:1.0.0'
```


## 参考
-   [木小五童鞋](https://www.jianshu.com/p/0d89418ee298)
-   此文由木小五童鞋整理，我指导并做后期修改，最终发布在简书上。现再次发布优化版本