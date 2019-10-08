---
layout:     post
title:      "Unity PackageManager与git的结合"
subtitle:   " \"github和Package\""
date:       2019-10-08 11:00:00
author:     "Conerlius"
category: unity
keywords: unity,PackageManager,git
tags:
    - unity
---

[TOC]

## 前提
> 本文里使用到的是自己已经完成的个人框架，不公开
> 
> 先说明一下准备的内容:
> - 一个独立的git仓(github或gitlab都可以)

## 独立仓的目录结构规范
> ```
> <root>
>  ├── package.json
>  ├── README.md
>  ├── CHANGELOG.md
>  ├── LICENSE.md
>  ├── Editor
>  │   ├── Unity.[YourPackageName].Editor.asmdef
>  │   └── EditorExample.cs
>  ├── Runtime
>  │   ├── Unity.[YourPackageName].asmdef
>  │   └── RuntimeExample.cs
>  ├── Tests
>  │   ├── Editor
>  │   │   ├── Unity.[YourPackageName].Editor.Tests.asmdef
>  │   │   └── EditorExampleTest.cs
>  │   └── Runtime
>  │        ├── Unity.[YourPackageName].Tests.asmdef
>  │        └── RuntimeExampleTest.cs
>  └── Documentation~
>       └── [YourPackageName].md
> ```
> 也可以不！（在2019.3beta上测试过）

## 重要的asmdef
如果没有asmdef,导入的package代码是不会编译的，但是可以在工程上使用

## Package下的路径

`Packages/com.[yourcomponay].[packageName]/`

## 如何编写package
在unity的`packages`目录下创建文件夹,格式是`com.[yourcomponay].[packageName]@[version]`