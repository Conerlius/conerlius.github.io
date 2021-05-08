---
layout:     article
title:      "Unity PackageManager与git的结合"
subtitle:   " \"github和Package\""
date:       2019-10-08 11:00:00
author:     "Conerlius"
category: unity
keywords: unity,PackageManager,git
tags:
    - unity
---

[Link](https://docs.unity3d.com/Manual/cus-layout.html)

## 前提
本文里使用到的是自己已经完成的个人框架，不公开

> 先说明一下准备的内容:
> - 一个独立的git仓(github或gitlab都可以)

## 独立仓的目录结构规范
 ```
<root>
 ├── package.json
 ├── README.md
 ├── CHANGELOG.md
 ├── LICENSE.md
 ├── Editor
 │   ├── Unity.[YourPackageName].Editor.asmdef
 │   └── EditorExample.cs
 ├── Runtime
 │   ├── Unity.[YourPackageName].asmdef
 │   └── RuntimeExample.cs
 ├── Tests
 │   ├── Editor
 │   │   ├── Unity.[YourPackageName].Editor.Tests.asmdef
 │   │   └── EditorExampleTest.cs
 │   └── Runtime
 │        ├── Unity.[YourPackageName].Tests.asmdef
 │        └── RuntimeExampleTest.cs
 └── Documentation~
      └── [YourPackageName].md
```

> 也可以不是这样的目录结构，只要有asmdef就可以了！（在2019.3beta上测试过）

## 重要的asmdef
如果没有asmdef,导入的package代码是不会编译的，但是可以在工程上使用。

## Package下的路径
### 相对路径
相对路径的格式一般如下
`Packages/com.[yourcomponay].[packageName]/`

eg. 加载一张png
`Texture2D texture = (Texture2D)AssetDatabase.LoadAssetAtPath("Packages/com.unity.images-library/Example/Images/image.png", typeof(Texture2D));`

### 绝对路径
unity中提供的`path`里有转换绝对路径的方法
`string absolute =   Path.GetFullPath("Packages/com.unity.images-library/Example/Images/image.png");`

## 如何编写package
因为package Manager直接添加的方式在unity 2019.3后才添加进去的，如果在2017（即unity发布package Manager后的所有版本）中使用，或者是开发package，应该怎么处理？

### 创建或直接引用package
> 这里说的引用并不是在unity 2019.3的add方式

在unity工程的root目录下，有一个`packages`的目录，在目录下创建文件夹,格式是`com.[yourcomponay].[packageName]@[version]`；

然后在创建的`com.[yourcomponay].[packageName]@[version]`目录下添加一个`package.json`文件；

并在`package.json`文件里写入

```
{
	"name": "com.yourcomponay.packageName",
	"displayName": "yourcomponay packageName",
	"version": "version"
}
```

> 如果没有在`package.json`下写入这些内容，unity是不会识别的。

### 修改配置
写入完成`package.json`后，回到unity，展开自己的package,点击期目录下的`package.json`，在unity的Inspector下可以看到下图的样式，然后我们直接在unity里修改package的配置就可以下，更高阶的使用，可以自己google学习

![jpg](/images/computer/game/unity/packageManager.jpg)

到这里，就基本是各位的自由发挥时间了！