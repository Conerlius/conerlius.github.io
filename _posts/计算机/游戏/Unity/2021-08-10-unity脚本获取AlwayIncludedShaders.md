---
layout:     article
title:      "unity脚本获取AlwayIncludedShaders"
subtitle:   " \"unity\""
date:       2021-08-10 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

## 大纲

- 大致原理
- 具体

## 大致原理

`AlwaysIncludedShaders`在`unity`的`Project Setting`->`Graphics`中有展示。但是无论是去查看`RenderSettings`、`GraphicsSettings`、`EditorGraphicsSettings`下都没有这个的一个字段或者方法！

![png](/images/computer/game/unity/unity_alwayincludedshaders.png)

所以最终的解决就是使用放射的方式去访问！

## 具体

`GraphicsSettings.GetGraphicsSettings()`返回的是一个`UnityEngine.Object`的对象，但时这个对象里就有我们需要的`AlwaysIncludedShaders`

通过查看`unity`反射的代码，可以看到以下的内容:

![png](/images/computer/game/unity/unity_alwayincludedshaders1.png)

```c#
var graphicsSettings = GraphicsSettings.GetGraphicsSettings();
var serializedObject = new SerializedObject(graphicsSettings);
SerializedProperty serializedProperty = serializedObject.FindProperty("m_AlwaysIncludedShaders");
```

这样就能获取到`m_AlwaysIncludedShaders`的数组了！

然后我们能通过`SerializedProperty`对`m_AlwaysIncludedShaders`进行读写了！

完整代码！

```C#
var graphicsSettings = GraphicsSettings.GetGraphicsSettings();
var serializedObject = new SerializedObject(graphicsSettings);
SerializedProperty serializedProperty = serializedObjeFindProperty("m_AlwaysIncludedShaders");
if (serializedProperty.isArray)
{
    for (int x = 0; x < serializedProperty.arraySize; x++)
    {
        SerializedProperty property = serializedProperGetArrayElementAtIndex(x);
        var sh = property.objectReferenceValue;
        Debug.LogError(sh);
    }
}
```