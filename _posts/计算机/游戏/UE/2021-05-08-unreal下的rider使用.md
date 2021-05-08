---
layout:     article
title:      "unreal下的rider使用"
subtitle:   " \"rider\""
date:       2021-05-08 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 纲要

- rider for unreal
- rider配置

### rider for unreal

简单说就是`rider`专门为`unreal`开发的一个`idea`;目前还是处于`beta`阶段。

https://www.jetbrains.com/lp/rider-unreal/

基本上是包含以下几个特点：

- A Fast IDE with native C++ support
- Knowledgeable about Blueprints
- Assists with the reflection mechanism
- Takes care of the Unreal Engine code style
- Profound code analysis & RPC support

### rider配置

- rider for unreal engine下载

https://www.jetbrains.com/rider/unreal/

通过上述的link填好申请后，会收到一个邮件，点击邮件的link激活后，在[rider证书](https://account.jetbrains.com/licenses)登录后就能看到已经激活的app；

![png](/images/computer/game/ue/1.png)

下载安装完成后自行登录账号激活就可以了！

- unreal配置

通过`Editor`-->`Plugins`添加`rider`插件

![png](/images/computer/game/ue/2.png)

![png](/images/computer/game/ue/3.png)

最后通过`Editor`-->`Editor Preference`指定`source code`的编辑器为`rider`：

![png](/images/computer/game/ue/4.png)