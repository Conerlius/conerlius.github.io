---
layout:     article
title:      "使用Android GPU Inspector分析GPU"
subtitle:   "GPU"
date:       2022-10-19 12:00:00
author:     "Conerlius"
category: unity
keywords: unity,unreal
tags:
    - unity
    - unreal
---

`Android GPU Inspector` 能干嘛，怎么下载之类的就不说了！

# 机型限制

本来我是有一台 `一加6t` 的测试机的，但是看了一下官方的支持列表里没有（论坛里有人说支持，我试过升级 `Android12` ,后还是提示不支持！）

还是得看官方！[link](https://developer.android.com/agi/supported-devices)


# 简单使用

![png](/images/computer/render/fenxi/agpid_1.png)

启动 `AGPID` 后,打开设备列表中，选择对应的机型，然后`OK` 按钮开始监控程序，等待进入指定的界面后，点击`Start`开始截帧即可

> 中途不能断开，如果截帧完成后，会出现`Open Trace`的按钮，点击后即可展开截帧的数据

> 如果是 `opengl` 的话，需要使用 `ANGLE` 去模拟，手机需要安装 `ANGLE`

![png](/images/computer/render/fenxi/agpid_3.png)

使用是很简，但是因为一些条件的问题，我这里就遇到各种麻烦，所以才记录一下

# 遇到过的问题

## 不通过 `adb` 安装 `ANGLE`

在执行到安装的过程中的时候断开，就能在一下路径中找到当前需要安装的apk

![png](/images/computer/render/fenxi/agpid_2.png)


```
java.util.concurrent.ExecutionException: com.google.gapid.server.Client$InternalServerErrorException: Process returned error
   Cause: exit status 1

	... 6 more

Caused by: com.google.gapid.server.Client$Stack: For request: RPC->installApp(ID { data: d43a68c4200fac71fee6835ed483c2f663327bc7 }, C:\Users\CHENBO~1\AppData\Local\Temp\agi_angle4271162659286047735.apk)

	at com.google.gapid.server.Client.call(Client.java:311)

	at com.google.gapid.server.Client.installApp(Client.java:266)

	at com.google.gapid.models.Devices.installApp(Devices.java:282)

	at com.google.gapid.views.TracerDialog$InstallAngleDialog.downloadAndInstall(TracerDialog.java:1585)

	at com.google.gapid.views.TracerDialog$InstallAngleDialog.lambda$showDialogAndInstallApk$0(TracerDialog.java:1490)

	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Unknown Source)

	at com.google.common.util.concurrent.TrustedListenableFutureTask$TrustedFutureInterruptibleTask.runInterruptibly(TrustedListenableFutureTask.java:131)

	at com.google.common.util.concurrent.InterruptibleTask.run(InterruptibleTask.java:74)

	at com.google.common.util.concurrent.TrustedListenableFutureTask.run(TrustedListenableFutureTask.java:82)

	... 6 more

```

在上面的异常中，认真看看就能找到需要安装的 `ANGLE` 了。（当然你也可以自己编译，我就是为了分析项目数据，就怎么快怎么来了~）

```
C:\Users\{当前用户}\AppData\Local\Temp\agi_anglexxxxxxxxxxxx.apk
```


## 设备不被发现

- 禁止权限监控
  
  开启开发者调试，并且在设置中把 `禁止权限监控` 开启了！

- 关闭手机上所有的后台程序!

这个方式不是很确定，我也是在看了log后大致看到的（有错的话，知道的大佬望指点一下）。把后台所有的app都关掉，然后回到 `agpid` 中刷新设备列表就能看到了！

## 截帧中刷新数据异常

因为在 `Android` 上是使用 `ANGLE` 去跑渲染才能有数据的，所以需要对app开启 `ANGLE` 的渲染。


```
adb shell settings put global angle_debug_package org.chromium.angle.agi

adb shell settings put global angle_gl_driver_selection_values angle

adb shell settings put global angle_gl_driver_selection_pkgs {你的包名}
```

## 内存消耗

这个问题我也不知道怎么搞，反正就是让公司加内存了。