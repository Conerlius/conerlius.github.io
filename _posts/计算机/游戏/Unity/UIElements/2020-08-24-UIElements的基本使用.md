---
layout:     article
title:      "UIelements的基本使用"
subtitle:   " \"unity\""
date:       2020-08-24 18:00:00
author:     "Conerlius"
category: unity
tags:
    - unity
    - uielements
---

本文章是实践为主的，所以顺序上和unity的文档有一定错乱！

## `UXML`文件

`UIElements`的使用是基于`UXML`文件的，在构建一个属于自己的`UIElements`前，我们需要先了解`UXML`的格式（基本的，更加详细请移步unity文档）。

### 文件格式

xml的固定开头
```xml
<?xml version="1.0" encoding="utf-8"?>
<UXML
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="UnityEngine.UIElements"
    xmlns:engine="UnityEngine.UIElements"
    xmlns:editor="UnityEditor.UIElements"
    xsi:noNamespaceSchemaLocation="../../UIElementsSchema/UIElements.xsd"
>
    <Label text="Hello World! From UXML" />
</UXML>
```

> 注意：
> 
> 如果没有`xmlns="UnityEngine.UIElements"`那么就一定需要`engine`的名称空间

`UXML`文件可以通过unity直接创建的！

> `Create`->`UI Toolkit`->`Editor Window` 菜单构建！

## 展示

`UIElements`提供了编辑直接预览的功能！

### Preview

在选中自定义的`UXML`文件后，unity的右下角（即unity提供的资源预览窗口可以直接看到生成的ui！）

![png](/images/computer/game/unity/uielements/1.png)

### C#加载

`UIElements`作为资源存放到unity工程，所以需要使用的时候，需要从`DCC`中加载到内存。

```c#
var template = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("path/to/file.uxml");
// 或者从模板中加载,常使用的是第一种！
var template = EditorGUIUtility.Load("path/to/file.uxml") as VisualTreeAsset;
```

加载完成后，将模板克隆一份到展示的窗口就能展示了。

```c#
public class MyWindow : EditorWindow  {
    [MenuItem ("Window/My Window")]
    public static void  ShowWindow () {
        EditorWindow w = EditorWindow.GetWindow(typeof(MyWindow));
        VisualTreeAsset uiAsset = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("Assets/GameAssets/First.uxml");
        VisualElement ui = uiAsset.CloneTree("");
        w.rootVisualElement.Add(ui);
    }
}
```

效果图

![png](/images/computer/game/unity/uielements/2.png)

## `UIElements`的各种类型

### Visual Tree

ui的树状结构，每个节点都包含布局、渲染顺序、事件响应等。

### VisualElement

`VisualElement`是`Visual Tree`的基本节点。包含节点样式、布局、事件等。



## USS样式

### uss的格式

```css
selector {
  property1:value;
  property2:value;
}
```

eg:

```css
Button {
  width: 200px;
}
```

更多的uss字段可以自行到官方网站上查看，这里就不说了！

### uss文件的引用

- C#动态引用
- UXML的静态引用

#### C#动态引用

`uss`文件和`uxml`文件差不多，都是作为资源，所有也需要加载！

```c#
var style = AssetDatabase.LoadAssetAtPath<StyleSheet>("path/to/file.uss");
// 或者从模板中加载,常使用的是第一种！
var style = EditorGUIUtility.Load("path/to/file.uss") as StyleSheet;
```

加载完成后，只需要将该`sytle`添加到ui的样式集合里就可以生效了！

```c#
VisualTreeAsset uiAsset = AssetDatabase.LoadAssetAtPath<VisualTreeAsset>("Assets/GameAssets/First.uxml");
var style = AssetDatabase.LoadAssetAtPath<StyleSheet>("Assets/GameAssets/firststyle.uss");
VisualElement ui = uiAsset.CloneTree("");
ui.styleSheets.Add(style);
editorWindow.rootVisualElement.Add(ui);
```

#### `UXML`的静态引用

全局生效：

```xml
<engine:UXML ...>
    <engine:VisualElement class="root">
        <Style src="styles.uss" />
		<!-- 这里写入需要采用此样式的控件 -->
		<!-- 不在这个有效范围的话，样式不生效 -->
    </engine:VisualElement>
</engine:UXML>
```

单个`Element`生效

```xml
<engine:UXML ...>
    <engine:VisualElement class="root">
        <Style src="styles.uss" />
			<!-- 这里使用Box作为例子说明 -->
			<Box class="style_name">
			</Box>
    </engine:VisualElement>
</engine:UXML>
```

demo:

firststyles.uss(***和`First.uxml`同一个目录下***)

```css
root {
    width: 200px;
    height: 200px;
    background-color: red;
}
Button {
    width: 50px;
}
.BoxButton {
    width: 250px;
}
```

First.uxml

```xml
<?xml version="1.0" encoding="utf-8"?>
<UXML
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="UnityEngine.UIElements"
        xsi:noNamespaceSchemaLocation="../UIElementsSchema/UIElements.xsd"
        xsi:schemaLocation="UnityEngine.UIElements ../UIElementsSchema/UnityEngine.UIElements.xsd">
    <VisualElement class="root">
        <Style src="firststyles.uss"/>
        <Label text="Select something to remove from your suitcase:"/>
        <Box>
            <Toggle name="boots" label="Boots" value="false"/>
            <Toggle name="helmet" label="Helmet" value="false"/>
            <Toggle name="cloak" label="Cloak of invisibility" value="false"/>
        </Box>
        <Box class="BoxButton">
            <Button name="cancel" text="Cancel"/>
            <Button name="ok" text="OK"/>
        </Box>
        <Button name="cancel1" text="testA"/>
    </VisualElement>
</UXML>
```

![png](/images/computer/game/unity/uielements/3.png)



## `UIElement`在C#代码的使用

### 代码创建

`UIElement`允许代码构建`VisualElement`, 下面以`Lable`为例，其他空间类似，具体参考官方文档。

```c#
VisualElement label = new Label("Hello World! From C#");
root.Add(label);
```

### 控件查询

```c#
root.Query<Button>("foo").First();
root.Query("foo").Children<Button>().ForEach(//do stuff);
```

### 其他

- 为组件动态指定样式

  在实际开发中，会因为逻辑的变更而对组件进行样式之类的修改。

  ```
  s
  ```

  

- 动态添加组件

- 动态删除或隐藏组件



## 事件行为

`UIElement`的事件传递如下:

![png](/images/computer/game/unity/uielements/4.png)

`UIElement`提供了一下两个常用的，和一个不怎么使用的。

- 注册
- 取消注册
- 其他

### 注册

```c#
// Register a callback on a mouse down event
myElement.RegisterCallback<MouseDownEvent>(MyCallback);
// Same, but also send some user data to the callback
myElement.RegisterCallback<MouseDownEvent, MyType>(MyCallbackWithData, myData);
```

### 取消注册

```c#
btn.UnregisterCallback<MouseDownEvent>(MyCallback);
```

> 在测试的时候，`MouseDownEvent`点击无效（unity 2020.1.2 alpha），使用`MouseUpEvent`正常，或者鼠标右键也正常！