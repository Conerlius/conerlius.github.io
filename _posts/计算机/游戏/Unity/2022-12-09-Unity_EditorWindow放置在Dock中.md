---
layout:     article
title:      "EditorWindow放置在Dock中"
subtitle:   " \"unity\""
date:       2022-12-09 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

## 需求说明

平时创建`EditorWindow`的时候，很多时候都是悬浮在`Game`上的，但是现在我需要一个类似`SceneView`的`EditorWindow`，所以最好也能在创建的时候直接放置在`Game`视图旁边（也就是`Dock`）！

## 解决方案

```c#
Type gameViewType = Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.GameView");
var sceneView = SceneView.CreateWindow<SceneView>(gameViewType);
sceneView.Show();
sceneView.Focus();
```

通过反射获取`Game`视图，然后在构建的`EditorWindow`里将dock参数传递进去！

其底层使用了一套相当完善的框架，目前大多数都是internal的，所以如果先要自己定义，反射的东西还挺多的。目前我自己使用的就是基于放射创建的。

以下是目前使用的简单创建代码

```c#
        // Type gameViewType = Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.GameView");
        // EditorWindow gameWindow = EditorWindow.GetWindow(gameViewType);
        // var win = SceneView.CreateWindow<MapEditorView>(gameViewType);
        var win = SceneView.CreateWindow<MapEditorView>();

        var ContainerWindowType = Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.ContainerWindow");
        var propty = ContainerWindowType.GetProperty("windows", BindingFlags.Static | BindingFlags.Public);
        var array = (Array)propty?.GetValue(null);
        
        var rootViewProperty = ContainerWindowType.GetProperty("rootView", BindingFlags.Instance | BindingFlags.Public);
        var ViewType = Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.View");
        var allChildrenProperty = ViewType.GetProperty("allChildren", BindingFlags.Instance | BindingFlags.Public);
        
        var DockAreaType = Assembly.GetAssembly(typeof(Editor)).GetType("UnityEditor.DockArea");
        var ms = DockAreaType.GetMethod("AddTab", new Type[]{typeof(EditorWindow), typeof(bool)});
        for (int i = 0; i < array.Length; i++)
        {
            Debug.Log(array.GetValue(i));
            var viewObj = rootViewProperty.GetValue(array.GetValue(i));
            var allView = (Array)allChildrenProperty.GetValue(viewObj);
            for (int j = 0; j < allView.Length; j++)
            {
                Debug.Log(allView.GetValue(j));
                if (allView.GetValue(j).GetType() == DockAreaType)
                {
                    ms.Invoke(allView.GetValue(j), new Object[] { win, true });
                }
            }
        }
        // var win = SceneView.CreateWindow<MapEditorView>();
        win.Show(true);
        win.titleContent = new GUIContent("MapEditor");
        win.Focus();
```
