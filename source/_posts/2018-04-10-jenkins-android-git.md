---
title: jenkins-android-git
date: 2018-04-10 16:32:08
categories: Jenkins
tags:
    - Jenkins
    - Android
    - Git
---

之前公司一直是使用使用Jenkins+Android+Svn，后面有以前的同事问到Jenkins+Git相关的问题，其实从根本上来说他们最大的区别就是使用Git去管理源代码，其他的都是一样的。但是涉及到Git，就有分支需要注意了（当然Svn也是有分支的概念的，只不过Svn分支更强调的是目录）。这里不讨论怎么搭建Jenkins以及和Android结合的具体步骤（这类的教程网上一抓一大把），只是记录其中需要注意的点。

### 1、注意url、访问权限、分支（如下图）

![jenkins-git](/images/2018/jenkins-git-20180410164825.png)

<!-- more -->

这里可能需要注意的就是访问权限了，涉及到使用ssh方式和账号密码的方式，根据对应的方式添加账号就可以了。然后分支注意使用全称（部分远端分支可能包含“/”），我现在的分支是使用参数选择的（这个后面会提到），代码就基本上就能拉下来了。

### 2、build-name-setter插件的使用

其实这个只是一个优化项，可以实现下面这种效果，很分明的表示每一次构建。

![jenkins-setter_pre](/images/2018/jenkins-setter-pre-20180410170833.png)

这里的配置也很简单，类似java等语言的format格式化，这里我使用到了打包序号和分支编号。注意这里需要装了插件才会有对应的设置项目（可能这个插件后续版本命名变化，需要自己去搜索一个就能找到最新的）。

![jenkins-setter](/images/2018/jenkins-setter-20180410171119.png)

### 3、配合Sonar做代码Review

![jenkins-sonar](/images/2018/jenkins-sonar-20180410172427.png)

这个就涉及到SonarQube Scanner插件了，这个连接SonarQube服务器做代码扫描的Jenkins插件。主要配置如下：

- sonar.projectKey 区分不同的项目；
- sonar.projectName 显示名称；
- sonar.projectVersion 版本；
- sonar.language 编程语言，Sonar 是支持很多语言的，不过Object-C 好像是收费插件。当一个项目中包含多语言的时候用逗号分隔（例如： sonar.language=java,grvy）；
- sonar.java.binaries 暂时也没搞懂，不过新版本的SonarQube 如果不配置会报错的；
- sonar.sources 代码路径；

### 4、参数化构建

![jenkins-param](/images/2018/jenkins-param-20180410174042.png)

这里添加了String类型的CUS_CODE变量和选择器类型的BRANCHE变量，BRANCHE每一行是一个选项，这里使用参数化达到编号和分支选择功能。当选择下图中的选项时，会把一些parameters放到临时环境变量和项目编译参数里面。具体就是会把他拼接到打包参数，可以在项目gradle脚本直接使用（当然是要判断的哦，方式本地编译找不到参数报错）。这里可以实现很多有趣、实用的功能，自己发现去吧。

![jenkins-gradle](/images/2018/jenkins-gradle-20180410175606.png)

### 5、使用全局属性

大家都知道Android项目编译依赖于SDK，但是我们的local.properties文件是被忽略的。这个时候全局属性就起作用了。系统管理->系统设置->全局属性，添加ANDROID_HOME等环境变量，就能避免找不到SDK等错误了。

![jenkins-env](/images/2018/jenkins-env-20180410180507.png)

### 6、命令行下载、更新SDK

打包可能会遇到找不到Platform、buildTools等版本找不到，就需要安装指定版本的。用Windows或者其他系统GUI版本的更新就很方便了，可是对于只有命令行环境的就会遇到些麻烦，由于我们的Jenkins部署在Linux的命令行环境下，不能通过GUI去选择下载SDK，这就引出下面的命令行更新SDK的方式。

- ANDROID_HOME->tools；
- ./android list sdk --all 列出全部sdk；
- ./android update sdk -u --all --filter 13 安装编号13的sdk 项目，此处不带filter 默认安装全部（建议是按需添加）；