---
layout:     article
title:      "Unity的GetComponent非空问题"
subtitle:   " \"==\""
date:       2019-09-26 15:00:00
author:     "Conerlius"
category: unity
keywords: unity,getcomponent,gameobject
tags:
    - unity
---

## 基本的原因

Unity对部分内置的Component（如rigidbody、animator等），在GetComponent时，如果组件不存在，Unity不会返回null，而是返回一个会判断为null的object（类似一个gameObject被Destroy后，会判断为null一样，可以参考这篇文章：[Custom == operator, should we keep it?](https://blogs.unity3d.com/cn/2014/05/16/custom-operator-should-we-keep-it/)。这是因为Unity重载了UnityEngine.Object的==运算符。

```
// 假设gameObject没有Rigidbody组件
var com = gameObject.GetComponent<Rigidbody>();
if (com == null) {
    // ok，因为Unity重载了运算符
}

object obj = com;
if (obj == null) {
    // 失败，此时会采用object自己的==判定
}
```