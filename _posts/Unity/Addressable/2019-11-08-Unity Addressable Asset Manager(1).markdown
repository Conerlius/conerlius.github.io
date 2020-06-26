---
layout:     article
title:      "Unity可寻址资源系统(1)"
subtitle:   " \"unity\""
date:       2019-11-08 15:00:00
author:     "Conerlius"
category: unity
keywords: unity,addressable
tags:
    - unity
---
* content
{:toc}

## Unity Addressable Asset System
unity 可寻址系统是提供了一种简单的资源加载方式，那就是通过`address`（地址）来加载。它通过简化内容包的创建和部署来处理资产管理的开销。

该资源系统采用的是异步的方式去加载资源，对于可寻址系统来说，不管你使用的是直接引用、传统的asset bundle还是resource，可寻址系统提供了一套简单的方式来实现动态加载。

## What is an Addressable Asset?
使用资源寻址可以让你通过资源的唯一地址去访问资源（无论该资源在何处）。无论资源是在本地还是在远程的网络上，可寻址资源系统都能定位和返回它。你可以通过资源的唯一地址去加载单个资源，也可以通过自定义的`Group`名称来加载大量的资源。

## Why use Addressable Assets?
简单点说就是以前项目需要自己实现资源管理系统，现在unity已经帮你做了，你可以花更多的时间去做其他事情了。
* 迭代时间
* 依赖管理，可寻址资源系统会帮你把加载资源的依赖找出来并且加载，然后在加载指定的资源返回。
* 内存管理，可寻址资源系统会对资源进行自动释放，自动引用计数，还提供了分析工具
* 内容打包，因为该系统能处理复杂依赖之间的关系链，就算你移动或者更名资源，打包都会更加效率，而且还支持远程加载内容，使得游戏的包体更加小。

## What about my existing game?
可寻址系统提供了一个`路径迁移`来升级。无论使用的是直接引用还是Resource,assetbundle！