---
layout:     post
title:      "Unity可寻址资源系统(2)"
subtitle:   " \"unity\""
date:       2019-11-08 21:00:00
author:     "Conerlius"
category: unity
keywords: unity,可寻址系统
tags:
    - unity
---
* content
{:toc}

## Addressable Assets Overview
可寻址资源系统包含两个packages:

* Addressable Assets package (primary package)
* Scriptable Build Pipeline package (dependency)

当安装`the Addressable Assets`的时候，`Scriptable Build Pipeline package`会作为依赖同时安装。

## Concepts

* Address: asset运行时的唯一id
* `AddressableAssetData`目录: 在工程中用于存储可寻址数据的
* Asset group: 一组可寻址资源
* Asset group schema: 定义一组在特定的在运行时的资源
* AssetReference: 和直接加载的方式类似的对象引用，但是会延时实例化。 `AssetReference object`以 GUID作为`Address`去加载。
* Asynchronous loading: 在不修改代码的情况下，可以修改资源的位置。
* Build script: 运行`asset group`处理器的时候，提供了一套用于匹配资源地址的。
* Label: 在运行时提供一种额外的用于指定加载相似的资源的便签(for example, Addressables.DownloadDependenciesAsync("spaceHazards");).