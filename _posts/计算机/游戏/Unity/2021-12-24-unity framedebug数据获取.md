---
layout:     article
title:      "unity framedebug数据获取"
subtitle:   " \"unity\""
date:       2021-12-24 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - framedebug
---

## 大纲
- 背景
- 当前帧【event】数据获取
  - 获取`FrameDebug Window`
  - 获取`FrameDebug`中选中的`Event`数据
- 一次debug的所有数据
  - 切换`Event Index`
  - 获取`FrameEventData`的数据
- play状态下的问题
  - update的问题
  - 数据获取不到

## 背景

项目前期，基于人力和项目复杂度的情形下，我将项目使用的`shader variant`的采集搞成了`ci`化；
但是到了项目中后期，由于项目越来越复杂，`ci`采样的问题越发突出了！（而且由于不同项目使用的规则也有所不同。）

所以计划在年后更换方案;

但是现在有些`bug`，那就是运行时，有些场景中的某个物体不能正确渲染;

> 因为物体在不同的坏境下，都有可能导致使用的变体不一样！

因为`ci`已经跑了一轮比较泛的采样了，所以就只补充了一下在遇到渲染不一致的问题的时候，只需要项目组的成员在`FrameDebug`中找到渲染异常的物体后，保存该变体就可以！

## 当前帧【event】数据获取

想要获取`unity`当前帧要渲染的数据，肯定离不开`FrameDebug`的；但是`FrameDebug`是各种`internal`;想要获取数据就变得有点麻烦了~

### 获取`FrameDebug Window`

直接看看`FrameDebug`是怎么打开和`Enable`的

```c#
var assembly = typeof(AnimationUtility).Assembly;
var FrameDebugWindows_Type = assembly.GetType("UnityEditor.FrameDebuggerWindow");


public static EditorWindow OpenAndEnableFrameDebug()
{
            // 强行开启framedebug
    EditorWindow frameDebuggerWindow = EditorWindow.GetWindow(FrameDebugWindows_Type);
    FrameDebugWindows_Type.GetMethod("EnableIfNeeded", BindingFlags.Instance | BindingFlags.Public).Invoke(frameDebuggerWindow, null);
    return frameDebuggerWindow;
}
```

通过反射的方式，我们获取到了`FrameDebuggerWindow`,并使得它`Enable`

### 获取`FrameDebug`中选中的`Event`数据

这里获取当前`Event`的渲染数据有两种方式，但是这里就说其中一种，另外一种在下面说明！


```c#
var DataEvent_FieldInfo = FrameDebugWindows_Type.GetField("m_CurEventData", BindingFlags.NonPublic | BindingFlags.Instance);
var FrameDebug_FrameDebuggerEventData_Type = assembly.GetType("UnityEditorInternal.FrameDebuggerEventData");


var data = DataEvent_FieldInfo.GetValue(frameDebuggerWindow);
var pi = FrameDebug_FrameDebuggerEventData_Type.GetField("shaderName");
var shadername = pi.GetValue(data) as string;
pi = FrameDebug_FrameDebuggerEventData_Type.GetField("passLightMode");
var passLightMode = pi.GetValue(data) as string;
            
data = DataEvent_FieldInfo.GetValue(frameDebuggerWindow);
pi = FrameDebug_FrameDebuggerEventData_Type.GetField("shaderKeywords");
var keyworlds = pi.GetValue(data) as string;
```

上面是直接反射了`FrameDebuggerWindow`中的`m_CurEventData`字段，因为`FrameDebuggerWindow`的展示中使用的也是这个字段中的内容，所以如果是获取选中的内容，直接反射界面中的内容即可！

> 获取到了`shadername` `passLightMode` `keyworlds`这三个后，就能将其加入到`shadervariantcollection`中了！

## 一次debug的所有数据

完成了上面的步骤后，放置在项目中使用的时候，项目成员不能很好把控哪个才是渲染不正确的，所以如果是需要选中指定的`Event`后在去获取`shadervariant`，那么容易导致有遗漏！于是就有后续——能一次性把当前帧的渲染变体都保存下来！

前文已经说了怎么启动`FrameDebugWindow`和`Enable`了，我们在`FrameDebugWindow`的界面中能看到所有的`Events`和当前是处于哪个`Event Index`

![ ](/images/computer/game/unity/unity优化/framedebug_allevents.png)

方法一：切换`Event Index`后，使用**当前帧【event】数据获取** 的方式去获取数据

这里我们就说说方法二，该获取数据的方法也可以用在 **当前帧【event】数据获取** 中!

### 切换`Event Index`

```c#
var FrameDebuggerUtility_Type = assembly.GetType("UnityEditorInternal.FrameDebuggerUtility");
var FrameDebuggerUtility_limit = FrameDebuggerUtility_Type.GetProperty("limit", BindingFlags.Public | BindingFlags.Static);
var FrameDebuggerWindow_RepaintOnLimitChange = FrameDebugWindows_Type.GetMethod("RepaintOnLimitChange", BindingFlags.Instance | BindingFlags.NonPublic);


public static bool ChangeEventIndex(EditorWindow window, int index)
{

    int i = (int)FrameDebuggerUtility_limit.GetValue(null);
    if (i == index+1)
    {
        return false;
    }

    FrameDebuggerWindow_ChangeFrameEventLimit.Invoke(window, new object[] { index+1 });
    FrameDebuggerWindow_RepaintOnLimitChange.Invoke(window, null);
    return true;
}
```

利用反射，设置`FrameDebuggerUtility`的数据，这里是直接使用`FrameDebuggerWindow:ChangeFrameEventLimit`和`FrameDebuggerWindow:RepaintOnLimitChange`去切换的

### 获取`FrameEventData`的数据

`FrameDebuggerUtility:GetFrameEventData`方法可以获取指定的`EventIndex`数据;但是需要先创建一个`UnityEditorInternal.FrameDebuggerEventData`对象;

```c#
var FrameDebuggerEventData_Type = assembly.GetType("UnityEditorInternal.FrameDebuggerEventData");

var frameEventData = Activator.CreateInstance(FrameDebuggerEventData_Type);
```

通过`FrameDebuggerUtility:GetFrameEventData`获取数据

```c#
var FrameDebuggerUtility_GetFrameEventData = FrameDebuggerUtility_Type.GetMethod("GetFrameEventData", BindingFlags.Static | BindingFlags.Public);
            
var flag = FrameDebuggerUtility_GetFrameEventData.Invoke(null, new object[]{index, frameEventData});
```

获取到的`frameEventData`对象后，就可以获取其内部的`shadername` `passLightMode` `keyworlds`!!

## play状态下的问题

### update的问题

由于Play的状态下，导致`inspector`的更新没有那么频繁，所以需要将界面不断设置为`dirty`才可以！

### 数据获取不到

因为获取数据是异步（可能吧，同步获取我偶尔也会获取不到！）所以最好是自己在`Update`中每次切换`EventIndex`的时候等一会！