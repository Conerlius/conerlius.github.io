---
layout:     article
title:      "自动化addressable"
subtitle:   " \"unity\""
date:       2020-05-29 15:00:00
author:     "Conerlius"
category: unity
tags:
    - unity
    - addressable
---


文章里说到的都是基于代码实现`addressable`资源配置的！文章使用的是unity 2019.3.3f1,addressable 1.9.2

## 创建`Addressable`的配置

首先瞅瞅unity是怎么创建的
unity通过`AddressableAssetSettings.Create`创建了第一个项目配置！

> `public static AddressableAssetSettings Create(string configFolder, string configName, bool createDefaultGroups, bool isPersisted)`

通过断点的方式，我们获取到了unity创建时的参数

![png](/images/computer/game/unity/addressable/1.png)

> configFolder : 整`Addressable`配置的目录，可以自定自己喜欢的目录(前提是`assetDatabase`能识别)
> 
> configName : 配置名称
> 
> createDefaultGroups : 是否构建默认的`group schema`(如果是`false`，在没有添加group的情况下，unity的`Addressable Window`是没有任何展示的,甚至认为项目没有创建`Addressable`)
> 
> isPersisted : 如果是`true`，在目录下没有配置的时候会自动创建一个新的配置，`false`则不会自动创建

```c#
public class AddressableWindow
{
	/// <summary>
	/// 配置的目录，需要assetDatabase识别
	/// AddressableAssetsData这个文件夹的名称最好不要改，要不然unity现在回生成两个文件夹
	/// 一个是AddressableAssetsData，还有一个是你指定的文件夹
	/// </summary>
	private static string configFolder = "Assets/AddressableAssetsData";
	/// <summary>
	/// 配置名称
	/// 最好是使用AddressableAssetSettings，否则你使用unity addressable window的时候，unity会创建多一个配置文件
	/// </summary>
	private static string configName = "AddressableAssetSettings";
	[MenuItem("Tools/构建Addressable")]
	static void CreateAstralAddressable()
	{
		AddressableAssetSettings setting =AddressableAssetSettings.Create(configFolder, configName, true, true);
	}
}
```

## 添加`Group Schema`

还是先瞅瞅unity是怎么创建的
unity通过`AddressableAssetSettings.CreateGroup`创建了第一个项目配置！

> `public AddressableAssetGroup CreateGroup(string groupName, bool setAsDefaultGroup, bool readOnly, bool postEvent, List<AddressableAssetGroupSchema> schemasToCopy, params Type[] types)`

通过断点的方式，我们获取到了unity创建时的参数

![png](/images/computer/game/unity/addressable/2.png)

> groupName : group名称
> 
> setAsDefaultGroup : 是否是默认的group
> 
> readOnly : 是否是只读
> 
> postEvent : 是否开启post event
> 
> schemasToCopy : 未测试是做什么的
> 
> types : 这里我们通过下图来看看group的信息，结合断点看到的types
> 
> ![png](/images/computer/game/unity/addressable/3.png)
> 
> 实际上就是group上的两组数据,既然是数据，那么我们就需要生成了。

```c#
/// <summary>
/// 查找group，如果没有就创建一个
/// </summary>
/// <param name="setting">addressable setting</param>
/// <param name="groupName">group名称</param>
/// <returns>group</returns>
private static AddressableAssetGroup GetGroup(AddressableAssetSettingssetting, string groupName)
{
	// 遍历所有的group看看是否已经存在(按名称)
	foreach (var addressableAssetGroup in setting.groups)
	{
		if (addressableAssetGroup.name == groupName)
		{
			return addressableAssetGroup;
		}
	}
	// 构建新的group
	return setting.CreateGroup(groupName, false, false, true, null, new Type[]
	{
		typeof(BundledAssetGroupSchema),
		typeof(ContentUpdateGroupSchema)
	});
}
```

### `Group Schema`的配置

- BundleMode

```c#
var groupSchema = group.GetSchema<BundledAssetGroupSchema>();
groupSchema.BundleMode = BundledAssetGroupSchema.BundlePackingMode.PackTogetherByLabel;
```

- Compression

```c#
var groupSchema = group.GetSchema<BundledAssetGroupSchema>();
groupSchema.Compression = BundledAssetGroupSchema.BundleCompressionMode.LZ4;
```

- AssetBundleProviderType

```c#
// 由于groupSchema.AssetBundleProviderType只有get，所以只能通过反射了获取
// groupSchema.AssetBundleProviderType = new SerializedType(){Value = typeof(LuaBundleProvider)};
var fieldInfo = groupSchema.GetType().GetFiel("m_AssetBundleProviderType", BindingFlags.Instance |BindingFlags.NonPublic);
fieldInfo?.SetValue(groupSchema, new SerializedType { Value =typeof(LuaBundleProvider) });
```

- BundledAssetProviderType

```c#
// 由于groupSchema.BundledAssetProviderType只有get，所以只能通过反射了获取
// groupSchema.BundledAssetProviderType = new SerializedType(){Value = typeof(LuaBundleProvider)};
var fieldInfo2 = groupSchema.GetType().GetFiel("m_BundledAssetProviderType", BindingFlags.Instance |BindingFlags.NonPublic);
fieldInfo2?.SetValue(groupSchema, new SerializedType { Value =typeof(LuaBundleProvider) });
```

## `Group Schema`添加资源

```c#
string guid = AssetDatabase.AssetPathToGUID(path);
var group = CreateOrGetDirectory(bundle);
var assetEntry = _setting.CreateOrMoveEntry(guid, group);
```

使用资源的`guid`添加

## 指定`Build and Play Mode Scripts`

unity默认自带的mode在创建的时候已经绑定在`Settings`里了（如果不需要的同学可以使用`Clear`清空所有的mode，然后自己添加）

- 首先需要构建mode的asset资源

	![png](/images/computer/game/unity/addressable/5.png)

	> 或者通过代码创建
	> 
	> ```c#
	> /// <summary>
	> /// 添加自定义editor加载
	> /// </summary>
	> /// <param name="setting">addressable setting</param>
	> /// <returns>BuildScriptBase</returns>
	> private static BuildScriptBase AddCustomPackedPlayMod	> (AddressableAssetSettings setting)
	> {
	> 	// mode asset的路径
	> 	var filePath = Path.Combine(configFolder, "DataBuilders/AstralPackedPlayMode.asset");
	> 	AstralPackedPlayMode playMode;
	> 	if (File.Exists(filePath))
	> 	{
	> 		// 获取已经存在的mode
	> 		playMode = AssetDatabase.LoadAssetAtPath<AstralPackedPlayMode>(filePath);
	> 	}
	> 	else
	> 	{
	> 		// 创建新的mode
	> 		playMode = ScriptableObject.CreateInstance<AstralPackedPlayMode>();
	> 		AssetDatabase.CreateAsset(playMode, filePath);
	> 	}
	> 	return playMode;
	> }
	> ```

- 然后在`AddressableAssetSettings` -> `Build and Play Mode Scripts`里添加新创建的`asset`即可
	![png](/images/computer/game/unity/addressable/6.png)

	> 或者通过代码添加
	> 
	> ```
	> // 如果尚未添加引用
	> if (!setting.DataBuilders.Contains(playMode))
	> {
	>  	 // 添加引用
	>  	 setting.DataBuilders.Add(playMode);
	> }
	> ```

## Label

`Addressable`的`Label`管理有很大的资源优势，所以避免不了需要添加，只有先添加到`LabelManager`的`label`才能在资源里使用

ui上怎么添加在本文就不说了，直接说说代码的添加方式.

```c#
/// <summary>
/// 往addressable中添加Label到LabelManager
/// </summary>
/// <param name="setting">addressable setting</param>
/// <param name="list">label列表</param>
private static void AddSettingsLabels(AddressableAssetSettingssetting, List<string> list)
{
	foreach (var label in list)
	{
		// 添加
		setting.AddLabel(label, true);
	}
}
```