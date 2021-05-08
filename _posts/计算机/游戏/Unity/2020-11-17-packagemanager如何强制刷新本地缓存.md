---
layout:     article
title:      "packagemanager如何强制刷新本地缓存"
subtitle:   " \"unity\""
date:       2020-11-17 12:00:00
author:     "Conerlius"
category: unity
tags:
    - unity
---

## 背景

公司现在已经已经使用`npm`处理公司框架很长一段时间了，中途有各种意外；这里就简单记述一下其中的一个问题!

- `packagemanager`本地缓存错误，而unity提供的`reflesh`却无法生效的bug

## 技术点

unity的package通常都会缓存到本地，官方的说明总结了一下大致是以下原因：

- 为了能共享
- 为了能在离线的时候也能使用
  
unity还提供了`UPM_NPM_CACHE_PATH `和`UPM_CACHE_PATH `给开发者自定义缓存的路径

`upm`的`update`指令经过我的测试，貌似并不能对已经`cache`的`package`进行再次更新（也许是我的水平不够，如果有，欢迎指出！）unity官方说使用他们的`RunUnityPackageManagerDiagnostics.bat`可以测试看情况，但是我本地运行的时候一直是报错！

```
<path-to-unity-installation-folder>
   \Unity
      \Data
         \Resources
            \PackageManager
               \Diagnostics
                  \RunUnityPackageManagerDiagnostics.bat
```
下面是我使用的npm指令(在unity安装目录中``)

```nodejs
./npm --registry=自定义的npmLink search com.XXX.XXX
./npm --registry=自定义的npmLink install com.XXX.XXX
./npm list
删除download下来的某些文件
./npm --registry=自定义的npmLink update com.XXX.XXX
```

以下是`unity` 缓存`package`的目录

| Operating system | Default root directory |
| -- | -- |
Windows (non-system user account) | %LOCALAPPDATA%\Unity\cache
Windows (system user account) | %ALLUSERSPROFILE%\Unity\cache
macOS | $HOME/Library/Unity/cache
Linux | $HOME/.config/unity3d/cache

如果有在运行`unity`时，`package`的报错`reload`一直无效，就可以去到`package`的缓存目录清理一下指定的`pacakge`(全部清理需要花费的时间比较大);

经过`UnityEditor.dll`的逆向，看到`unity`提供了一个`IUpmClient`的类型，它就包含了一个`clearCache`的方法（目前没有看到有`refresh`的方法）。

目前可行的方法有以下一个:

- 使用脚本清理`package`的缓存目录！

正在测试的方法，通过反射的方式获取`IUpmClient`的单例，然后执行`clearCache`方法（看了一下方法，貌似是直接清理了全部的！）
