
title: gradle 上传jcenter
date: 2016-4-28 17:55:18
tags:
---
鉴于目前blog没有深度的状态，继续写些零碎 -_- 

### 两个链接：
http://www.cnblogs.com/qianxudetianxia/p/4322331.html
http://www.jcodecraeer.com/a/anzhuokaifa/Android_Studio/2015/0227/2502.html
按照两个链接开始了jcenter的上传之旅。
按照上面步骤的步骤基本可以完成上传，在这提一下上传过程中遇到的问题。

### Issue
因为上两篇文章是基于androidStudio版本比较低的情况写的，所一些配置必须更新为与androidStudio2.1相匹配。
项目下的build.gradle如下：(其余跟原链接相同)
```groovy
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        jcenter()
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }


    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.0-rc1'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
        classpath "com.jfrog.bintray.gradle:gradle-bintray-plugin:1.6"
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
allprojects {
    repositories {
        jcenter()
    }
}
```

###相关知识点

###了解aar文件

仓库中存储的有两种类型的library：jar 和 aar。jar文件大家都知道，但是什么是aar文件呢？

aar文件时在jar文件之上开发的。之所以有它是因为有些Android Library需要植入一些安卓特有的文件，比如AndroidManifest.xml，资源文件，Assets或者JNI。这些都不是jar文件的标准。

因此aar文件就时发明出来包含所有这些东西的。总的来说它和jar一样只是普通的zip文件，不过具有不同的文件结构。jar文件以classes.jar的名字被嵌入到aar文件中。其余的文件罗列如下：

– /AndroidManifest.xml (mandatory)

– /classes.jar (mandatory)

– /res/ (mandatory)

– /R.txt (mandatory)

– /assets/ (optional)

– /libs/
.jar (optional)

/ .so (optional)
– /proguard.txt (optional)

– /lint.jar (optional)

可以看到.aar文件是专门为安卓设计的。因此这篇文章将教你如何创建与上传一个aar形式的library。

###上传的步骤

### 通过gradew执行这些任务即可,或通过gradle可视化窗口依次执行上面的命令,没有任何错误输出便说明已经成功。
- gradlew javadocJar
- gradlew sourcesJar
- gradlew install
- gradlew bintrayUpload

###提交审核
登陆bintray主页，搜索包名添加到jcenter（不然的话：buildscript你得这样写才能添加依赖

```groovy
maven {
       url "https://bintray.com/you package name"
      }）
```
 )
 提交说明可以不用填。

 ### 关于审核通过的通知
 大约过了两个小时，我在bintray主页和邮箱都没有看到审核通过的通知，并且http://jcenter.bintray.com/com/库的路径不存在，但本地测试该库的依赖情况，发现依赖成功了，可能是同步问题吧，费解。然后明天早我便发现库可以在jcenter找到，也收到了邮箱的通知，大概前后相距6，7个钟。


