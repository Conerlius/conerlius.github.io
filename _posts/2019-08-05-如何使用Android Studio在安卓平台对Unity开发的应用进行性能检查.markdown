---
layout:     post
title:      "如何使用Android Studio在安卓平台对Unity开发的应用进行性能检查？[转]"
subtitle:   " \"Android debug\""
date:       2019-08-05 13:00:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - 转载;
---
[原文link:https://www.cnblogs.com/murongxiaopifu/p/10605053.html](https://www.cnblogs.com/murongxiaopifu/p/10605053.html)
# 前言
> 大家常常会抱怨安卓平台没有一个统一、好用的性能检查工具。不能像iOS的instrument那样方便。
> ![](https://pic4.zhimg.com/80/v2-36c80a6f7cbbb34cf3d981bf1875f023_hd.jpg)
> 图片来自：Instruments Help
> 比如，Unity Blog在3年前就已经教大家如何使用instrument来进行性能检测了。
> [Profiling with Instruments – Unity Blog​blogs.unity3d.com](https://blogs.unity3d.com/2016/02/01/profiling-with-instruments/)
> 其实目前的Android Studio已经提供了很好的安卓平台的性能检查工具。
> https://developer.android.com/studio/profile
> 这里主要介绍CPU Profiler来检查Unity原生函数的性能。就像iOS上的instrument一样。
> 图形相关的性能检测，可以使用https://github.com/google/gapid。
# 导出设置
> 下文中我使用的Android Studio版本为：3.5 preview，Unity版本为：2018.3.0b11。测试工程为：SurvivalShooter（Asset Store） 。
> 首先，根据android studio的文档：
https://developer.android.com/studio/profile/cpu-profiler?hl=en
>
> ```
> Sampled (Native): Captures sampled traces of your app’s native threads. To use this configuration, you must deploy your app to a device running Android 8.0 (API level 26) or higher.
> ```
> 进行native sample的设备系统版本要求是：Android 8.0 (API level 26)
> ![](https://pic1.zhimg.com/80/v2-68130ce931fe936a4c8dc79c20858f0c_hd.jpg)
> 为了可以检测脚本的代码开销，（同样，就像instrument那样）推荐Scripting Backend使用il2cpp。
> 我们可以自己写一个函数，用来测试。（当然，工程中的脚本也是一样的）
> ！[](https://pic4.zhimg.com/80/v2-1e4975e34bd86c9763e845ead85a6def_hd.jpg)
> 之后，我们把工程按照Gradle的形式导出，以便之后使用as打开。
> ![](https://pic3.zhimg.com/80/v2-0662dd5bd7735d6a1ecff8a053a704c6_hd.jpg)
# 符号信息
> 正常使用as打开导出的工程。在
> src/main/jniLibs/armeabi-v7a/
> 目录下，可以找到相关的so文件。我们主要关注libunity.so以及libil2cpp.so。前者是引擎部分，后者是开发者的脚本部分。此时的so都是符号信息不全的。所以我们要做的只是替换符号信息更全的so。
> ![](https://pic4.zhimg.com/80/v2-d882050f4562504a57c8018f768ebacb_hd.jpg)
> 首先来替换libunity.so，可以在Unity目录下
> playbackEngines/AndroidPlayer/Variations/il2cpp/Development/Libs/armeabi-v7a/
> 这里找到带符号信息的so。
> ![](https://pic1.zhimg.com/80/v2-88f5af0bef26708dde727b8e618b5190_hd.jpg)
> 之后，如果还想查看脚本的调用开销（il2cpp），我们就需要将带符号信息的libil2cpp替换。
> 它的路径在：
> 工程``/Temp/StagingArea/symbols/armeabi-v7a/libil2cpp.so.debug``
> ![](https://pic3.zhimg.com/80/v2-8f935db1b74765e481106841994d16ba_hd.jpg)
> 可以看到，gradle工程中的libil2cpp.so只有7.7mb，而带符号信息的libil2cpp则有88.3mb。
# Profiling!
> ![](https://pic1.zhimg.com/80/v2-406ca0862ebba948449e276a9b2b14f0_hd.jpg)
> 点击右上角的Profile Button，就可以开始进行性能检测了。
> 选中CPU栏，检测项目中有Java Method、Sample C/C++等等。选择C/C++，即可看到一个timeline。点击recorde，则开始记录call trace。
> ![](https://pic1.zhimg.com/80/v2-efebe54c51504bdcbc094d3b7129188c_hd.jpg)
> 可以看各个线程的函数调用的chart，整个流程比较流畅。
> 也可以按照时间开销，整理为堆栈调用的形式。这里可以看到使用il2cpp的脚本函数调用。
> 我们可以找一找我们的UpdateTest1那个函数
> ![](https://pic2.zhimg.com/80/v2-c779420f37b86897a13611f554128071_hd.jpg)
> 当然，这里和本文开头Unity Blog中在iOS平台上使用Instrument类似，主要是用来检测一些原生函数的性能，大家可以将它作为Unity Profiler在安卓平台上的一种补充。