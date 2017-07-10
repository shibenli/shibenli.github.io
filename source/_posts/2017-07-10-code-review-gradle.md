---
title: 代码Review系列之4-Gradle配置
date: 2017-07-10 13:43:34
categories: Android
tags:
    - Android
    - 代码Review
---
## Gradle配置
![Gradle配置](/images/2017/Gradle配置.png)

通过添加新的buildType->dev，让dev除服务器地址以外的配置都和release一致，保证在发布提交测试的时候代码和release同步。并且在dev的时候使用```.dev```作为versionNameSuffix，增加svn的提交版本号来作为App名称的后缀。为测试人员反馈bug、开发人员定位bug提供版本依据，更是对于自动打包和自动分发测试提供方便。其他的比如加入打包时间、服务器地址配置等

<!-- more -->

## 附录

### 完整gradle代码



	apply plugin: 'com.android.application'
	apply plugin: 'walle'

	import org.tmatesoft.svn.core.wc.*
	
	def getSvnRevision() {
	    try {
	        ISVNOptions options = SVNWCUtil.createDefaultOptions(true);
	        SVNClientManager clientManager = SVNClientManager.newInstance(options);
	        SVNStatusClient statusClient = clientManager.getStatusClient();
	        SVNStatus status = statusClient.doStatus(projectDir, false);
	        SVNRevision revision = status.getCommittedRevision();
	        return revision.getNumber();
	    } catch (e) {
	        return "0";
	    }
	}
	
	def CVSVersionCode = getSvnRevision();
	
	def getDate() {
	    def date = new Date()
	    def formattedDate = date.format('yyyyMMddHHmm')
	    return formattedDate
	};
	
	android {
	    compileSdkVersion rootProject.ext.compileSdkVersion
	    buildToolsVersion rootProject.ext.buildToolsVersion
	
	    dexOptions {
	        maxProcessCount 4
	        javaMaxHeapSize "2g"
	    }
	
	    defaultConfig {
	        applicationId "com.*.*"
	
	        minSdkVersion rootProject.ext.minSdkVersion
	        targetSdkVersion rootProject.ext.targetSdkVersion
	        versionCode rootProject.ext.versionCode
	        versionName rootProject.ext.versionName
	
	        // Enabling multidex support
	        multiDexEnabled true
	
	        jackOptions {
	            enabled false
	        }
	
	        ndk {
	            abiFilters "armeabi"
	        }
	
	        resValue "string", "app_name", rootProject.ext.appName
	        buildConfigField "String", "CVS_VERSIONCODE", "\"" + CVSVersionCode + "\""
	        buildConfigField "String", "APP_NAME", "\"" + rootProject.ext.appName + "\""
	        buildConfigField "String", "PROJECT_NAME", "\"" + rootProject.ext.projectName + "\""
	
	        ...
	    }
	
	    productFlavors{
	        app{
	
	        }
	        lite{
	            resValue "string", "app_flavors_display", "信用卡管理"
	            applicationIdSuffix name
	        }
	
	        z{
	            minSdkVersion 21 //仅仅用作开发测试
	
	        }
	    }
	
	    lintOptions {
	        abortOnError false
	    }
	
	    compileOptions {
	        sourceCompatibility JavaVersion.VERSION_1_7
	        targetCompatibility JavaVersion.VERSION_1_7
	    }
	
	    signingConfigs {
	        release {
	            storeFile file("android.keystore")
	            storePassword "android"
	            keyAlias "android"
	            keyPassword "android"
	        }
	    }
	
	    buildTypes {
	        def String[] proguards = [getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro', 'proguard-fresco.pro', 'proguard-umshare.pro', 'proguard_x5.cfg']
	
	
	        debug {
	            minifyEnabled false
	            shrinkResources false
	            signingConfig signingConfigs.release
	            useLibrary 'org.apache.http.legacy'
	            versionNameSuffix ".deb"
	
	            resValue "string", "app_display", rootProject.ext.appName
	            buildConfigField "String", "BUILD_FLAG", String.format("\"(%s)\"", getDate())
	            buildConfigField "String", "BASE_URL", "\"http://www.apptest.***.cn/\""
	            //buildConfigField "String", "BASE_URL", "\"http://10.141.10.110:9090/\""
	            buildConfigField "String", "BASE_H5_URL", "\"http://www.apptest.***.cn/20170420Client/\""
	            buildConfigField "String", "BusType", "\"CP\""
	            buildConfigField "boolean", "ENCRYPT", "true"
	            buildConfigField "String", "BASE_SERVER", "\"20170420App\""
	            buildConfigField "boolean", "CREDIT_SETTING", "false"
	        }
	
	        release {
	            minifyEnabled true
	            zipAlignEnabled true
	            shrinkResources true
	            signingConfig signingConfigs.release
	            proguardFiles proguards
	
	            resValue "string", "app_display", rootProject.ext.appName
	            buildConfigField "String", "BUILD_FLAG", "\"\""
	            buildConfigField "String", "BASE_URL", "\"http://www.app.***.cn/\""
	            buildConfigField "String", "BASE_H5_URL", "\"http://www.app.***.cn/20170420Client/\""
	            buildConfigField "String", "BusType", "\"CP\""
	            buildConfigField "boolean", "ENCRYPT", "true"
	            buildConfigField "String", "BASE_SERVER", "\"20170420App\""
	            buildConfigField "boolean", "CREDIT_SETTING", "true"
	        }
	
	        dev {
	            minifyEnabled release.minifyEnabled
	            zipAlignEnabled release.zipAlignEnabled
	            shrinkResources release.shrinkResources
	
	            signingConfig release.signingConfig
	            proguardFiles proguards
	
	            versionNameSuffix ".dev"
	
	            resValue "string", "app_display", rootProject.ext.appName + "-" + CVSVersionCode
	            buildConfigField "String", "BUILD_FLAG", String.format("\"(%s)\"", getDate())
	            buildConfigField "String", "BASE_URL", "\"http://www.apptest.***.cn/\""
	            buildConfigField "String", "BASE_H5_URL", "\"http://www.apptest.***.cn/20170420Client/\""
	            buildConfigField "String", "BusType", "\"CP\""
	            buildConfigField "boolean", "ENCRYPT", "true"
	            buildConfigField "String", "BASE_SERVER", "\"20170420App\""
	            buildConfigField "boolean", "CREDIT_SETTING", "false"
	        }
	    }
	
	    applicationVariants.all { variant ->
	        variant.outputs.each { output ->
	            if (!variant.buildType.name.equals('debug')) {
	                def outputFile = output.outputFile
	                //这里修改文件名
	                def fileName = rootProject.ext.projectName + ".apk"
	                output.outputFile = new File(rootProject.buildDir.path, fileName)
	            }
	        }
	    }
	
	    File propFile = file('../config/signing_new.properties');
	    if (propFile.exists()) {
	        def Properties props = new Properties()
	        props.load(new FileInputStream(propFile))
	        if (props.containsKey('STORE_FILE') && props.containsKey('STORE_PASSWORD') &&
	                props.containsKey('KEY_ALIAS') && props.containsKey('KEY_PASSWORD')) {
	            android.signingConfigs.release.storeFile = file(props['STORE_FILE'])
	            android.signingConfigs.release.storePassword = props['STORE_PASSWORD']
	            android.signingConfigs.release.keyAlias = props['KEY_ALIAS']
	            android.signingConfigs.release.keyPassword = props['KEY_PASSWORD']
	        } else {
	            android.buildTypes.release.signingConfig = null
	        }
	    } else {
	        android.buildTypes.release.signingConfig = null
	    }
	}
	
	//packer {
	//    // 指定渠道打包输出目录
	//    archiveOutput = file(new File(project.rootDir.path, "release/" + CVSVersionCode))
	//    // 指定渠道打包输出文件名格式
	//    archiveNameFormat = '${projectName}-${flavorName}-${buildType}-v${versionName}-${versionCode}-${buildTime}-${fileMD5}-' + CVSVersionCode
	//    //是否检查Gradle配置中的signingConfig
	//    checkSigningConfig = true
	//}
	
	walle {
	    // 指定渠道包的输出路径
	    apkOutputFolder = new File("${project.rootDir}/release/" + CVSVersionCode);
	    // 定制渠道包的APK的文件名称
	    apkFileNameFormat = '${projectName}-${channel}-${buildType}-v${versionName}-${versionCode}-${buildTime}-${fileSHA1}-' + CVSVersionCode + '.apk'
	    // 渠道配置文件
	    channelFile = new File("${project.rootDir}/config/markets.txt")
	//    configFile = new File("${project.rootDir}/config/config.json")
	}
	
	dependencies {
	    compile fileTree(include: ['*.jar'], dir: 'libs')
	    compile fileTree(include: ['*.jar'], dir: 'libs/share')
	    //compile project(':roboto-calendar')
	    compile 'com.vcredit:roboto-calendar:1.0.0'
	    //compile project(':lib-zxing')
	    compile 'com.vcredit:lib-zxing:1.0.0'
	    compile project(':liveness')
	//    compile(name: 'liveness-release', ext: 'aar')
	//    compile 'com.vcredit:liveness:1.0.0'
	    compile project(':handwrite')
	//    compile 'com.vcredit:handwrite:1.0.0'
	    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4-beta2'
	    debugCompile 'com.facebook.stetho:stetho-okhttp3:1.4.1'
	    debugCompile 'com.facebook.stetho:stetho:1.4.1'
	    debugCompile 'com.squareup.okhttp3:okhttp:3.4.1'
	    debugCompile 'com.facebook.stetho:stetho-js-rhino:1.4.1'
	    compile 'com.android.support:appcompat-v7:22.2.1'
	    compile 'com.android.support:recyclerview-v7:22.2.1'
	    compile 'com.android.support:support-v4:22.2.1'
	    compile 'com.android.support:multidex:1.0.1'
	    compile 'com.android.support:design:22.2.1'
	    compile 'com.android.volley:volley:1.0.0'
	    compile 'com.google.code.gson:gson:2.7'
	    compile 'com.jakewharton:butterknife:8.4.0'
	    annotationProcessor 'com.jakewharton:butterknife-compiler:8.4.0'
	    compile 'com.umeng.analytics:analytics:5.6.1'
	    compile 'com.zhy:percent-support-extends:1.1.1'
	    compile 'com.klinkerapps:link_builder:1.3.3'
	    compile 'com.github.shibenli:DateTimePicker:v0.1.0'
	    compile 'com.github.siyamed:android-shape-imageview:0.9.3'
	    compile 'org.greenrobot:eventbus:3.0.0'
	    compile 'com.yancy.imageselector:imageselector:1.3.3'
	    compile 'com.github.bumptech.glide:glide:3.7.0'
	    //画图图表
	    compile 'com.github.PhilJay:MPAndroidChart:v3.0.2'
	    //广告栏控件
	    compile 'com.bigkoo:convenientbanner:2.0.5'
	    //多渠道打包
	    compile 'com.meituan.android.walle:library:1.1.2'
	    compile 'in.srain.cube:ultra-ptr:1.0.11'
	    compile 'com.github.vcredit-zj:commons-langa:v3.5-1'
	    compile 'com.liulishuo.filedownloader:library:1.4.2'
	    compile 'com.github.mcxtzhang:SuspensionIndexBar:V1.0.0'
	    compile 'com.jakewharton:disklrucache:2.0.2'
	    compile 'com.github.CymChad:BaseRecyclerViewAdapterHelper:2.9.18'
	}
