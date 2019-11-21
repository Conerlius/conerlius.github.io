---
layout:     post
title:      "Unity可寻址资源系统(4)"
subtitle:   " \"unity\""
date:       2019-11-14 22:00:00
author:     "Conerlius"
category: unity
keywords: unity,可寻址系统
tags:
    - unity
---

[link](https://docs.unity3d.com/Packages/com.unity.addressables@1.1/manual/AddressableAssetsDevelopmentCycle.html)

## Addressable Assets development cycle
一个重要的指标就是怎么安排资源间的截藕，构建，和加载内容。 通常在开发的时候都是大量捆绑一起的。

### Traditional asset management
如果将资源安排到`Resources`目录，那么资源就会打包到应用程序内，必须通过`Resources.Load`来加载，该方法支持路径参数。你可以使用`direct references`或者`asset bundles`来访问其他地方的资源。如果使用的是`asset bundles`，也是通过路径去加载和组织他们，如果`asset bundles`是远端的，就需要写代码去管理下载、加载和卸载了。

## Addressable Asset management
你可以通过地址来加载资源，无论该资源你放在工程的哪里或者是以何种方式打包。你随意可以修改可寻址资源的路径或文件名也不会产生问题。甚至可以在不改动代码的情况下改变其加载方式

### Asset group schemas
你可以在`Inspector`中指定`asset groups`的模式，这定义了构建管线怎么处理这些内容。eg:当使用的是`packed mode`去构建时, `asset groups`的`Inspector`里面的`BundledAssetGroupSchema`有上设置的模板，可以修改其设置。`AddressableAssetsSettings `还可以添加模式。

## Build scripts
构建脚本是工程中实现`IDataBuilder`接口的`ScriptableObject`资源。 你可以创建自己的构建脚本，然后透过`Inspector`将它们添加到`AddressableAssetSettings`。想要使用构建脚本,打开`Addressables window`(`Window` > `Asset Management` > `Addressables`)，请选择`Build Script`，然后选择一个下拉选项。 当前，已实现三个脚本来支持完整的应用程序构建，并提供三个用于在编辑器中进行迭代的播放模式脚本。

### Play mode scripts
可寻址资源打包有3种构建脚本来加速开发

#### Fast mode
Fast mode (BuildScriptFastMode) :使得你可以快速启动游戏。`Fast mode`通过直接快速从`asset database`中加载，且不带分析非`asset bundle`的官方是创建。

#### Virtual mode
Virtual mode (BuildScriptVirtualMode)：在没有构建`asset bundles`的情况下分析内容和依赖。通过`ResourceManager`从`asset database`加载资源，如同通过`bundles`来加载（模拟）。可以通过`Addressable Profiler window` (`Window` > `Asset Management` > `Addressable Profiler`) 来观察

`Virtual mode`可以帮助你模拟加载策略和协助你去平衡`release`版本的`group`均衡。

#### Packed Play mode
Packed Play mode (BuildScriptPackedPlayMode)：使用的是已经打包完成的`bundle`. 这个模式是分接近正式部署的应用，但是这个需要独立的步骤去构建内容。 如果在没有改动资源的情况下，在次模式就不需要处理任何资源了。 该模式也需要先构建资源内容 (`Window` > `Asset Management` > `Addressables`) 选中 `Build` > `Build Player Content`, 或者执行`AddressableAssetSettings.BuildPlayerContent() `。

Choosing the right script
To apply a Play mode script, from the Addressables window menu (Window > Asset Management > Addressables), select Play Mode Script and choose from the dropdown options. Each mode has its own time and place during development and deployment. The following table illustrates stages of the development cycle, in which a particular mode is useful.

| - | Design | Develop | Build | Test / Play | Publish|
| --- | --- | --- | --- | --- | --- |
| Fast | x | x | | In-Editor only	|
|Virtual | x | x | Asset bundle layout | In-Editor only	|
|Packed| | |Asset bundles|x|x

Analysis and debugging
By default, Addressable Assets only logs warnings and errors. You can enable detailed logging by opening the Player settings window (Edit > Project Settings > Player), navigating to the Other Settings > Configuration section, and adding "ADDRESSABLES_LOG_ALL" to the Scripting Define Symbols field.

You can also disable exceptions by unchecking the Log Runtime Exceptions option in the AddressableAssetSettings object Inspector. You can implement the ResourceManager.ExceptionHandler property with your own exception handler if desired, but this should be done after Addressables finishes runtime initialization (see below).

Initialization objects
You can attach objects to the Addressable Assets settings and pass them to the initialization process at runtime. The CacheInitializationSettings object controls Unity's caching API at runtime. To create your own initialization object, create a ScriptableObject that implements the IObjectInitializationDataProvider interface. This is the Editor component of the system responsible for creating the ObjectInitializationData that is serialized with the runtime data.

Content update workflow
Unity recommends structuring your game content into two categories:

Static content that you never expect to update.
Dynamic content that you expect to update.
In this structure, static content ships with the application (or downloads soon after install), and resides in very few large bundles. Dynamic content resides online, ideally in smaller bundles to minimize the amount of data needed for each update. One of the goals of the Addressable Assets System is to make this structure easy to work with and modify without having to change your scripts.

However, the Addressable Assets System can also accommodate situations that require changes to the "static" content, when you don't want to publish a whole new application build.

How it works
Addressables uses a content catalog to map an address to each asset, specifying where and how to load it. In order to provide your app with the ability to modify that mapping, your original app must be aware of an online copy of this catalog. To set that up, enable the Build Remote Catalog setting on the AddressableAssetSettings Inspector. This ensures that a copy of the catalog gets built to and loaded from the specified paths. This load path cannot change once your app has shipped. The content update process creates a new version of the catalog (with the same file name) to overwrite the file at the previously specified load path.

Building an application generates a unique app content version string, which identifies what content catalog each app should load. A given server can contain catalogs of multiple versions of your app without conflict. We store the data we need in the addressables_content_state.bin file. This includes the version string, along with hash information for any asset that is contained in a group marked as StaticContent. By default, this file is located in the same folder as your AddressableAssetSettings.asset file.

The addressables_content_state.bin file contains hash and dependency information for every StaticContent asset group in the Addressables system. All groups building to the StreamingAssets folder should be marked as StaticContent, though large remote groups may also benefit from this designation. During the next step (preparing for content update, described below), this hash information determines if any StaticContent groups contain changed assets, and thus need those assets moved elsewhere.

Preparing for content updates
If you have modified assets in any StaticContent groups, you'll need to run the Prepare For Content Update command. This will take any modified asset out of the static groups and move them to a new group. To generate the new asset groups:

Open the Addressables window in the Unity Editor (Window > Asset Management > Addressable Assets).
In the Addressables window, select Build on the top menu bar, then Prepare For Content Update.
In the Build Data File dialog that opens, select the addressables_content_state.bin file (by default, this is located in the Assets/AddressableAssetsData Project directory.
This data is used to determine which assets or dependencies have been modified since the application was last built. The system moves these assets to a new group in preparation for the content update build.

Note: This command will do nothing if all your changes are confined to non-static groups.

Important: Before running the prepare operation, Unity recommends branching your version control system. The prepare operation rearranges your asset groups in a way suited for updating content. Branching ensures that next time you ship a new player, you can return to your preferred content arrangement.

Building for content updates
To build for a content update:

Open the Addressables window in the Unity Editor (Window > Asset Management > Addressable Assets).
In the Addressables window, select Build on the top menu, then Build For Content Update.
In the Build Data File dialog that opens, select the build folder of an existing application build. The build folder must contain an addressables_content_state.bin file.
The build generates a content catalog, a hash file, and the asset bundles.

The generated content catalog has the same name as the catalog in the selected application build, overwriting the old catalog and hash file. The application loads the hash file to determine if a new catalog is available. The system loads unmodified assets from existing bundles that were shipped with the application or already downloaded.

The system uses the content version string and location information from the addressables_content_state.bin file to create the asset bundles. Asset bundles that do not contain updated content are written using the same file names as those in the build selected for the update. If an asset bundle contains updated content, a new asset bundle is generated that contains the updated content, with a new file name so that it can coexist with the original. Only asset bundles with new file names must be copied to the location that hosts your content.

The system also builds asset bundles for static content, but you do not need to upload them to the content hosting location, as no Addressables asset entries reference them.

Content update examples
In this example, a shipped application is aware of the following groups:

|Local_Static|Remote_Static|Remote_NonStatic|
|---|---|---|
|AssetA|AssetL|AssetX|
|AssetB|AssetM|AssetY|
|AssetC|AssetN|AssetZ|

As this version is live, there are players that have Local_Static on their devices, and potentially have either or both of the remote bundles cached locally.

If you modify one asset from each group (AssetA, AssetL, and AssetX), then run Prepare For Content Update, the results in your local Addressable settings are now:

|Local_Static|Remote_Static|Remote_NonStatic|content_update_group (non-static)|
|---|---|---|---|
| | |AssetX|AssetA|
|AssetB|AssetM|AssetY|AssetL|
|AssetC|AssetN|AssetZ|	
Note that the prepare operation actually edits the static groups, which may seem counter intuitive. The key, however, is that the system builds the above layout, but discards the build results for any static groups. As such, you end up with the following from a player's perspective:

Local_Static
AssetA
AssetB
AssetC
The Local_Static bundle is already on player devices, which you can't change. This old version of AssetA is no longer referenced. Instead, it is stuck on player devices as dead data.

Remote_Static
AssetL
AssetM
AssetN
The Remote_Static bundle is unchanged. If it is not already cached on a player's device, it will download when AssetM or AssetN is requested. Like AssetA, this old version of AssetL is no longer referenced.

Remote_NonStatic (old)
AssetX
AssetY
AssetZ
The Remote_NonStatic bundle is now old. You could delete it from the server, but either way it will not be downloaded from this point forward. If cached, it will eventually leave the cache. Like AssetA and AssetL, this old version of AssetX is no longer referenced.

Remote_NonStatic (new)
AssetX
AssetY
AssetZ
The old Remote_NonStatic bundle is replaced with a new version, distinguished by its hash file. The modified version of AssetX is updated with this new bundle.

content_update_group
AssetA
AssetL
The content_update_group bundle consists of the modified assets that will be referenced moving forward.

Here are the implications of this example:

Any changed local assets remain unused on the user's device forever.
If the user already cached a non-static bundle, they will need to re-download the bundle, including the unchanged assets (in this instance, for example, AssetY and AssetZ). Ideally, the user has not cached the bundle, in which case they simply need to download the new Remote_NonStatic bundle.
If the user has already cached the Static_Remote bundle, they only need to download the updated asset (in this instance, AssetL via content_update_group). This is ideal in this case. If the user has not cached the bundle, they must download both the new AssetL via content_update_group and the now-defunct AssetL via the untouched Remote_Static bundle. Regardless of the initial cache state, at some point the user will have the defunct AssetL on their device, cached indefinitely despite never being accessed.
The best setup for your remote content will depend on your specific use case.