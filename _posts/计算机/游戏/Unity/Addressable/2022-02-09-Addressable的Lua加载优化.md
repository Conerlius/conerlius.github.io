---
layout:     article
title:      "Addressable的Lua加载优化"
subtitle:   " \"unity\""
date:       2022-02-09 15:00:00
author:     "Conerlius"
category: unity
tags:
    - unity
    - addressable
---

## 现象

在开始排查当前情况的时候,项目工程中使用的lua代码已经达到了6M

在本地电脑中加载的时候需要5s以上的时长！据项目的反馈，手机上的加载时间有些机器需要十几秒！

![png](/images/computer/game/unity/addressable/9.png)

## 排查过程

- 第一步是先统计一下加载的耗时

在 `AddressResource` 中，在创建和加载完成的两个方法中添加计时，完成的时候打印耗时；

![png](/images/computer/game/unity/addressable/10.png)

- 确定ab的加载时间

![png](/images/computer/game/unity/addressable/11.png)

ab加载耗时极少！

-定位加载过长的原因

通过断点，看到因为`label`需要把所有的同样`label`的资源都进行`provider`加载，使得lua的每一个文件都会进行`BundledLuaProvider`加载！而且还是异步的！

```c#
m_RequestOperation = bundle.LoadAssetAsync(assetPath, m_ProvideHandle.Type);
m_RequestOperation.completed += ActionComplete;
```

## 方案：

### 方案一：

直接将`BundledLuaProvider`中从`ab`回去资源的异步方式改为同步！

![png](/images/computer/game/unity/addressable/12.png)

### 方案二：

`lua`的加载不使用`addressable`记载，类似于

```c#
string path = m_ProvideHandle.ResourceManager.TransformInternalId(m_ProvideHandle.Location);
if (File.Exists(path))
{
    m_RequestOperation = AssetBundle.LoadFromFileAsync(path, m_Options == null ? 0 : m_Options.Crc);
    m_RequestOperation.completed += LocalRequestOperationCompleted;
}
```

直接加载`assetbundle`，然后对`ab`进行操作，这样就可以自己处理了！

### 方案三：

使用工具，在打包`lua`之前，将所有的`lua`文件合并成为一个文件，这样的话一个`lua ab`就只有一个`asset`了（即`lua`之类的`label`加载的时候就只有一个`provider`需要处理！），好处在于目前的`addressable`不需要修改，只需要写个工具，还有在`lua`资源加载完后解释就可以了

### 方案四：

修改`addressable`的加载，但是里面牵连了`AsyncOperationBase`、`AsyncOperationHandle`、`ProviderOperation`等一系列的操作，改动过大！

### 方案五：

在`addressable`生成的`catelog`进行删除修改，使得一个`label`下只有一个`asset name`,这样在调用`BundledLuaProvider`的时候直接使用`loadall`就可以了！

### 其他：

还有不少是基于`addressable`的修改，但是改动都不少


## 结果

目前看来`方案一`、`方案二`、`方案三`的可行性较高！