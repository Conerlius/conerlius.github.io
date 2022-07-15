---
layout:     article
title:      "PrefabStageUtility在Mac打包报错"
subtitle:   "unity"
date:       2022-07-15 16:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

## 环境说明
| | |
|-|-|
|打包机|马蜂窝|
|Unity| Unity2021.3.5f1|


## 测试情况

本地电脑环境
| | |
|-|-|
|打包环境|Win10 台式|
|Unity| Unity 2021.3.5f1|

测试结果，打包没有问题，能正常出Android和Win的ab，输出也能正常运行！


## 问题
但是在提交到CI出包的时候，遇到以下的error:
```
##### Output
UGUIViewElementsRegistryUtils.cs(15,31): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
UGUIViewElementsRegistryUtils.cs(32,77): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
UGUIViewElementsRegistryInspector.cs(34,35): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
UGUIViewElementsRegistryBuilder.cs(15,35): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
UGUIViewElementsRegistryUtils.cs(128,31): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
UGUIViewElementsRegistryUtils.cs(243,47): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
*** Tundra build failed (12.54 seconds), 1204 items updated, 1422 evaluated
```
```
##### Output
CreateCorrespondingLuaFiles.cs(16,31): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
CreateCorrespondingLuaFiles.cs(26,35): error CS0234: The type or namespace name 'PrefabStageUtility' does not exist in the namespace 'UnityEditor.Experimental.SceneManagement' (are you missing an assembly reference?)
```

而且也有不少人遇到的！

https://forum.photonengine.com/discussion/18715/prefabstageutility-does-not-exist-in-the-namespace-unityeditor-experimental-scenemanagement


## 解决方法
在代码里不要使用
```c#
UnityEditor.SceneManagement.PrefabStageUtility
```
如果代码一定要使用这个类，就按照下面的形式来使用
```
using UnityEditor.SceneManagement;
...
 
PrefabStageUtility.XXXX
```
这样在打包的时候就不会报错了！
估计是和`dll`中的`MovedFrom`有关！