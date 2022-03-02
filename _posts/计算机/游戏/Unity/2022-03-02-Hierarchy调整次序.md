---
layout:     article
title:      "Hierarchy调整次序"
subtitle:   " \"unity\""
date:       2022-03-02 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

由于某些原因，我需要替换`Hierarchy`下某个`GameObject`,如果是直接`DestroyImmediate`后再`Instantiate`的话，新创建的`GameObject`是直接放置在最后的（同一个父节点的情况下）

Unity提供了`GetSiblingIndex`函数获取位置

```c#
var index = transform.GetSiblingIndex();
```

如果需要替换原来的位置

```c#
newObj.transform.SetSiblingIndex(index);
```

`Index`就是位置的index,
eg：
```c#
// 置顶
newObj.transform.SetSiblingIndex(0);
```