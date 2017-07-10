---
title: 代码Review系列之1-Module分布
date: 2017-07-10 10:01:43
categories: Android
tags:
    - Android
    - 代码Review
---
## Module分布 ##

![Module分布](/images/2017/Module分布.png)

根据功能模块分不同的Module，对于一个上线的功能打包成aar的包上传到maven仓库中。

<!-- more --> 

## 打包参考脚本 ##

基于nuexs的maven私服打包脚本可以参考：

```
apply plugin: 'maven'

tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}

task androidSourcesJar(type: Jar) {
    classifier = 'sources'
    from android.sourceSets.main.java.source
}


artifacts {
    archives androidSourcesJar
    //archives androidJavadocsJar
}

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: MAVEN_REPO_RELEASE_URL) {
                authentication(userName: NEXUS_USERNAME, password: NEXUS_PASSWORD)
                println('userName: ' + NEXUS_USERNAME + ', password:' + NEXUS_PASSWORD)
            }
            pom.project {
                name "liveness"
                version "1.0.0"
                artifactId 'liveness'
                groupId 'com.vcredit'
                packaging 'aar'
            }
        }
    }
}

```

注：上面MAVEN_REPO_RELEASE_URL等是通过gradle.properties配置文件配置的。
- MAVEN_REPO_RELEASE_URL： maven仓库的地址
- NEXUS_USERNAME： maven仓库的用户名
- NEXUS_PASSWORD： maven仓库的密码
