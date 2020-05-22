---
layout:     article
title:      "Unity Android Profile"
subtitle:   " \"Profile\""
date:       2019-10-12 00:25:00
author:     "Conerlius"
category: unity
keywords: unity,profile
tags:
    - unity
---
* content
{:toc}

## Unity profile连接不上android真机
先说一下能profile的前提
- apk必须是develop的
- Unity导包的时候必须勾选`autoProfile`(如下图1所示)
- 了解profile的adb监听接口(如下图2所示)

图1:<br>
![png](/images/Unity/unity_android_profile2.png)

在unity 2019中，官方已经提供了`deep profile support`

图2:<br>
![png](/images/Unity/unity_android_profile.png)

上图看到的端口`34999`;

有些情况下，在启动apk的时候，unity就能直接连接上unity profile;

如果连接不上的话,请执行以下步骤:

- 手机打开游戏.
- 执行以下命令行

```shell
adb forward tcp:[YourPort] localabstract:Unity-com.xx.xx
```

> 如果还是不行，可以尝试`adb kill-server`后再次执行上述命令行

- 在 Unity Profiler 编辑器上选择 对应的`AndroidPlayer（ADB@127.0.0.1：[YourPort]）`

## android真机deep profiler

- 在android 上进行deep profiler

```shell
adb shell am start -n com.company.game/com.unity3d.player.UnityPlayerActivity -e ‘unity’ '-deepprofiling’
```

在执行上述命令后，手机就会自动启动游戏了。

如果需要和`forward`一起使用的话，可以像下面这样

```shell
adb shell am start -n com.company.game/com.unity3d.player.UnityPlayerActivity -e ‘unity’ '-deepprofiling’ forward tcp:[YourPort] localabstract:Unity-com.xx.xx
```

> 如果提示说`UnityPlayerActivity`找不到，那么可以展开包，查看一下该`Activity`的包路径，修正命令上的路径即可