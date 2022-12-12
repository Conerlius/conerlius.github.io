---
layout:     article
title:      "Window的简单结构"
subtitle:   "Window的简单结构"
date:       2022-12-12 12:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

效果：
![png](/images/computer/game/unity/editorwindow/create_window.png)

ue是源码，完整的ui框架有开放出来，但是unity不是，开放有限，有时候想搞些事情的时候，因为视图布局之类的，特别不方便，所以我就瞅瞅看看unity有没有办法搞！嘢，还真的有！

Unity内置提供了很多`internal`的类！我这里就直接反射了`ContainWindow`、`SplitView`、`DockArea`这三个！

参考`EditorWindow.CreateNewWindowForEditorWindow`，以下图的形式组装自己想要的ui布局！
![png](/images/computer/game/unity/editorwindow/create_window2.png)

```c#
    [MenuItem("MapEditor/Open #1")]
    static void Open()
    {
        var window = CreateWindow("Demo Window");
    }
    static WindowDemo CreateWindow(string name)
    {
        // 主窗口
        RefContainerWindow mainWindow  = RefContainerWindow.CreateContainerWindow();
        RefSplitView mainPanel = RefSplitView.CreateSplitView();
        mainWindow.rootView = mainPanel.MainObject;
        // 左中右三个方向的布局
        RefDockArea leftArea = RefDockArea.CreateRefDockArea();
        RefDockArea middleArea = RefDockArea.CreateRefDockArea();
        RefDockArea rightArea = RefDockArea.CreateRefDockArea();

        // 为每个区域指定要渲染的editor window
        var leftMenuWindow = ScriptableObject.CreateInstance<MapEditor_LeftMenu>();
        var middleWindow = ScriptableObject.CreateInstance<MapEditor_Middle>();
        var rightMenuWindow = ScriptableObject.CreateInstance<MapEditor_RightMenu>();
        leftArea.AddTab(leftMenuWindow);
        middleArea.AddTab(middleWindow);
        rightArea.AddTab(rightMenuWindow);
        
        // 左中右三个方向的布局一次添加到split view中，默认是水平，有需要可以指定成垂直！
        mainPanel.AddChild(leftArea.MainObject);
        mainPanel.AddChild(middleArea.MainObject);
        mainPanel.AddChild(rightArea.MainObject);
        
        mainWindow.position = new Rect(300,300,600,800);
        // mainPanel.position = new Rect(0.0f, 0.0f, 600, 800);
        mainWindow.Show();
        mainWindow.OnResize();
        mainWindow.UnsavedStateChanged();
        return null;
    }
```

`RefXXX`是我反射unity里的`internal`类,其实还是挺简单的~不过目前基本勉强够用了~
