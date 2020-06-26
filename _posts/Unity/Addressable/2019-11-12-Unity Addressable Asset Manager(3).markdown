---
layout:     article
title:      "Unity可寻址资源系统(3)"
subtitle:   " \"unity\""
date:       2019-11-12 22:00:00
author:     "Conerlius"
category: unity
keywords: unity,addressable
tags:
    - unity
---
* content
{:toc}

## Getting started with Addressable Assets
### Installing the Addressable Assets package
自己去安装就行，这里就不说了。

### Preparing Addressable Assets
#### Marking assets as Addressable
在unity Editor下有两种方式可以讲资源标志为可寻址资源:

* 资源的Inspector界面。
* 在`Addressables`窗口

#### Using the Inspector
在`Project`窗口，选中资源，然后在`Inspector`界面将`Address`勾选上,输入名称来指定资源的寻址id。
![png](/images/Unity/adressassetinspectorcheckbox.png)

#### Using the Addressables window
选中`Window` > `Asset Management` > `Addressables` 打开`Addressables`窗口。将希望变成可寻址的资源拖拽到窗口的某一个`Group`下即可
![png](/images/Unity/addressableswindow.png)

### Specifying an address
默认的资源地址就是资源在项目中的路径 (for example, Assets/images/myImage.png). 在`Addressables window`中可以改变他们的地址, 右键资源，选中`Rename`.

如果是第一次启动使用可寻址系统，系统会保存edit和运行时的资源数据，将其放置在`Assets/AddressableAssetsData` 文件中, 这也将添加到你的资源版本控制中

### Building your Addressable content
可寻址系统需要把运行需要的资源构建到文件系统中，这个步骤是不会自动执行的，可以通过一下方式去构建:

* 打开`Addressables window`, 选择`Build` > `Build Player Content`。
* 或者使用api ```AddressableAssetSettings.BuildPlayerContent()```。

## Using Addressable Assets
### Loading or instantiating by address
在运行时加载`adressable Assets`，首先要声明``UnityEngine.AddressableAssets``，然后调用一下的方法去加载:
```
Addressables.LoadAssetAsync<GameObject>("AssetAddress");
```
上面的就是通过制定的`adress`地址去加载`asset`资源。
```
Addressables.InstantiateAsync("AssetAddress");
```
上面的就是通过制定的`adress`地址去实例化`asset`资源到场景上。

Note: `LoadAssetAsync` 和 `InstantiateAsync` 都是异步操作。 你可以在加载的时候添加回调来监听该操作
```
using System.Collections;
using System.Collections.Generic;
using UnityEngine.AddressableAssets;
using UnityEngine;

public class AddressablesExample : MonoBehaviour {

    GameObject myGameObject;

        ...
        Addressables.LoadAssetAsync<GameObject>("AssetAddress").Completed += OnLoadDone;
    }

    private void OnLoadDone(UnityEngine.ResourceManagement.AsyncOperations.AsyncOperationHandle<GameObject> obj)
    {
        // In a production environment, you should add exception handling to catch scenarios such as a null result.
        myGameObject = obj.Result;
    }
}
```

### Sub-assets and components
`Sub-assets` 和 `components` 对于资源加载来说是一个特例。

* Components: GameObject上的component不能以资源的形式直接加载出来。需要先加载和实例化GameObject后再从中获取。
* Sub-assets: 系统支持加载`sub-assets`，但是需要特殊的语法糖。 比如潜在`sub-assets`图集中的Sprite或FBX文件中的动画。 要加载它们，可以使用以下示例语法：
```
Addressables.LoadAssetAsync<IList<Sprite>>("MySpriteSheetAddress");
```

## Using the AssetReference class
需要使用`AssetReference`类型，只需要在代码上把资源的类型声明为`AssetReference`去访问就可以。

* 在场景的`hierarchy` 或者`Project window`中选中GameObject。
* 在`Inspector`窗口添加一个有`public AssetReference variable`成员的脚本。（`public AssetReference explosion;`）
* 在`Inspector`界面,可以展开列表选择指定的可寻址资源。或者在`Project Window`上将可寻址资源拖拽到`Inspector`的属性上；
![png](/images/Unity/adressassetInspectorreferenceselection2.png)

加载和实例化可寻址资源，可以通过相应的方法执行:
```
AssetRefMember.LoadAssetAsync<GameObject>();
```

or

```
AssetRefMember.InstantiateAsync(pos, rot);
```

注意: 通常可寻址资源的`LoadAssetAsync`和`InstantiateAsync`都是异步的操作。你可以提供一个加载完成回调的句柄;([Async operation handling]())

## Build considerations
### Local data in StreamingAssets
可寻址资产系统在运行时需要一些文件来知道要加载什么以及如何加载它。这些文件是在构建`Addressables`数据并在StreamingAssets文件夹中生成的。在生成资源数据的时候，资源系统会存储这些资源到`library`,当你构建你的程序的时候，系统会拷贝必须的文件到`StreamingAssets`目录下。通过这种方式，你可以构建多平台的数据。

除了特定于可寻址对象的数据外，任何构建其数据供本地使用的组也将使用特定于库平台的暂存位置。 要验证此方法是否有效，请分别将构建路径和加载路径设置为概要文件变量，以
```
UnityEngine.AddressableAssets.Addressables.BuildPath
```
And
```
UnityEngine.AddressableAssets.Addressables.RuntimePath
```
您可以在AddressableAssetSettings检查器中指定这些设置（默认情况下，此对象位于项目的Assets / AddressableAssetsData目录中）。

### Downloading in Advance
通过调用`Addressables.DownloadDependenciesAsync()`，依据地址或标签参数去加载依赖资源。通常这些都是`asset bundle`。

通过`PercentComplete`属性得到的`AsyncOperationHandle`结构包含了加载完成的进度，你可以使用这个来展示下载进度，也可以让应用等待内容的下载完成。

通过`Addressables.GetDownloadSize()`可以获取要下载内容的大小。值得注意的是，这包含了先前已经下载到unity缓存中的资源。

预先下载资源也许对应用比较有利，但是在某些特殊情况下却也不一定。

* 如果您的应用程序包含大量在线内容，并且您通常希望用户仅与应用程序的一部分进行交互。
* 您有一个必须在线连接才能运行的应用程序。 如果您所有应用程序的内容都捆绑在一起，则可以根据需要选择下载内容。

### Building for multiple platforms
在构建应用程序内容时，可寻址资源系统将可寻址资源打包成为`Asset Bundle`。 `Asset Bundle`是平台相关的，因此必须为要每个平台重建`Asset Bundle`。

默认情况下，构建Addressables数据时，给定平台的数据存储在Addressables构建路径的特定于平台的子目录中。 运行时会根据平台选择相应的平台文件夹，并指向相应的资源文件。

注意：如果在Editor模式下Addressables使用的是BuildScriptPackedPlayMode脚本，则Addressables将尝试加载当前活动平台的资源。 因此，如果你指向资源的`Platform`和当前的Editor Platform不一致（不兼容），则可能会出现问题。 
