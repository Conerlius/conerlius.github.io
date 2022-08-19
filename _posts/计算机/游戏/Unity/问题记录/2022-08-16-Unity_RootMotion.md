---
layout:     article
title:      "Unity RootMotion简单尝试记录"
subtitle:   "unity"
date:       2022-08-16 16:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

找了一大圈资料，基本都是各种copy，讲的基本都是unity怎么使用~为此，我还特意去看了ue的源码，想着对比看看！

这里本来是想要说明工作的整个过程的，但是因为结果不理想，我也不敢说这是在说原理。

本文只是在对unity root motion的一个简单的尝试记录！

## Root Motion 工作原理

官方文档的说明

https://docs.unity3d.com/cn/current/Manual/RootMotion.html


下面就是一套瞎操作！

## 资源导入及简单设置

模型导入的时候使用的是`Humanoid`

![png](/images/computer/game/unity/problems/animation/motion1.png)

动画导入也要选择`Humanoid`

![png](/images/computer/game/unity/problems/animation/motion2.png)
Animation页签
![png](/images/computer/game/unity/problems/animation/motion3.png)

在`Root Motion Node`中使用的是`< None >`的时候，界面就和网上google或者官方的界面样式一致了！
like this! https://docs.unity3d.com/cn/current/Manual/RootMotion.html

## Apply Root Motion 在Root Motion Node为none的测试

![png](/images/computer/game/unity/problems/animation/motion4.png)

直接运动到帧末的时候，GameObject在Z轴的位置是 $ -3.99 $ ！

展开一下动作资源,在使用`Root Motion Node`为`none`时只有`Root T`和`Root Q`,这两个curve的最后数值！（获取的代码在文后）

![png](/images/computer/game/unity/problems/animation/motion5.png)

居然有差异！！！

## 排查差异

![png](/images/computer/game/unity/problems/animation/motion6.png)

把关于`Aniomation`的优化全关掉！

运行得到的结果没有变化！！

> 这个看情况的，如果本来和动作同学设定的值变成较大，关掉这些优化后，还是有变化的。

咨询了一下动作的同学，得到设置的值

![png](/images/computer/game/unity/problems/animation/motion7.png)

这一下子就懵逼了我。

https://forum.unity.com/threads/mecanim-root-motion-for-generic-rig-quadruped.169891/
https://forum.unity.com/threads/is-there-a-way-to-extract-rootmotion-curves-from-a-clip.610579/
https://forum.unity.com/threads/make-root-motion-in-animation-clip.250723/


![png](/images/computer/game/unity/problems/animation/motion8.png)

问题是我在Animation中没有看到Motion!!!

## Apply Root Motion 在Root Motion Node为Root的测试

尝试将`Root Motion Node`为`Root`
![png](/images/computer/game/unity/problems/animation/motion9.png)
然后再Animation中就能找到`Motion Curve`
![png](/images/computer/game/unity/problems/animation/motion10.png)
尝试获取一下`Root T`和`Root Q`(或者`Motion T`和`Motion Q`)的`Curve`，然后读取一下数据，也测试一下`Apply Root Motion`的执行结果！

`Apply Root Motion`的执行结果：
![png](/images/computer/game/unity/problems/animation/motion11.png)

Curve读取的结果：

![png](/images/computer/game/unity/problems/animation/motion12.png)

展开Curve看看

![png](/images/computer/game/unity/problems/animation/motion13.png)

基本就是-3.7呀！！怎么Unity的`Apply Root Motion`成了-4呢？而且3dmax中设置也是-4!!!(`Apply Root Motion`的结果更加逼近3dmax)

自己在Unity中搞一个Animaion试试

![png](/images/computer/game/unity/problems/animation/motion14.png)

![png](/images/computer/game/unity/problems/animation/motion15.png)

赞，我设置的10呀！！

去掉Apply Root Motion
![png](/images/computer/game/unity/problems/animation/motion16.png)

这就一样了！！！

晕了，等其他大佬分享吧！现在资料有点少~或者以后看UE的源码再分析一下。

```c#
if (Clip.hasMotionCurves)
{
    var bindings = AnimationUtility.GetCurveBindings(Clip);
    // 也可以改成Root测试！
    var pName = "Motion";
    foreach (var bind in bindings)
    {
        if (bind.propertyName == $"{pName}T.x")
        {
            curve_x = AnimationUtility.GetEditorCurve(Clip, bind);
            x = curve_x.Evaluate(7);
        }
        if (bind.propertyName==$"{pName}T.y")
        {
            curve_y = AnimationUtility.GetEditorCurve(Clip, bind);
            y = curve_y.Evaluate(7f);
        }
        if (bind.propertyName==$"{pName}T.z")
        {
            curve_z = AnimationUtility.GetEditorCurve(Clip, bind);
            z = curve_z.Evaluate(7f);
        }
        if (bind.propertyName==$"{pName}Q.x")
        {
            curve_Qx = AnimationUtility.GetEditorCurve(Clip, bind);
            qx = curve_Qx.Evaluate(7f);
        }
        if (bind.propertyName==$"{pName}Q.y")
        {
            curve_Qy = AnimationUtility.GetEditorCurve(Clip, bind);
            qy = curve_Qy.Evaluate(7f);
        }
        if (bind.propertyName==$"{pName}Q.z")
        {
            curve_Qz = AnimationUtility.GetEditorCurve(Clip, bind);
            qz = curve_Qz.Evaluate(7f);
        }
        if (bind.propertyName==$"{pName}Q.w")
        {
            curve_Qw = AnimationUtility.GetEditorCurve(Clip, bind);
            qw = curve_Qw.Evaluate(7f);
        }
    }
}
```