title: stetho使用体验
date: 2015-10-30 10:00:50
tags:
---
前言：
    听说stetho功能极其强大，包括查看：网络请求的整个过程，查看sqlite数据库，所以就报着好奇心checkout了官方的demo。


构建-编译-运行
    更改build.gradle..的环境与本地环境相匹配。
    -----
    构建过程中出现个错误：can not find 'org.apache.http.legacy' 注释编译通过，其作用待查...

使用展示图：
    网络请求
    ![](http://7xl7ew.com1.z0.glb.clouddn.com/t_stetho_network.png)
    source查看
    ![](http://7xl7ew.com1.z0.glb.clouddn.com/t_stetho_source.png)


    目前体验到的强功能点。（sharepreference sqlitedatabase network request）。
    #这工具，感觉还是挺溜的吗