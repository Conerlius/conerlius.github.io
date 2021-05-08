---
layout:     article
title:      "Unity Animation优化"
subtitle:   " \"Animation优化\""
date:       2019-09-05 18:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

[TOC]

> Unity的动画系统一直比较让人无奈的，谁用谁知道！但是Unity的同时也提供了另外的一套解决方案，可以优化动画带来的部分问题
> * 减少有动作的骨骼数量
>> 一个骨骼(Transform)是有position(x,y,z),rotation(x,y,z,w),scale(x,y,z)3个数据，在unity里，我们可以打开animation观察
>> ![png](/images/computer/game/unity/CustomAnimation1.jpg)
>>
>> 一个Position,`x,y,z`三个数据，三条curve那么一个骨骼就有3(position)+4(rotation)+3(scale)=10个curve，少一个带动画的骨骼，那么就直接少了10条curve
> * Curve曲线的剔除
>> 比如剔除无用的`Scale`曲线(看具体而定，不要盲目剔除)
>> ```
>> [MenuItem("Assets/CustomAnimation/剔除所有Scale曲线")]
>> static void CullCurves()
>> {
>>     // 需要直接选中AnimationClip
>>     var animation_go = Selection.activeObject;
>>     if (animation_go.GetType() == typeof(AnimationClip))
>>     {
>>         // clip
>>         var clip = animation_go as AnimationClip;
>>         // 获取Animation的所有Curve
>>         var binds = AnimationUtility.GetCurveBindings(clip);
>>         foreach (var bind in binds)
>>         {
>>             // 通常名称都是m_LocalScale.(x/y/z),如果是就置空
>>             if (bind.propertyName.Contains("Scale"))
>>                 AnimationUtility.SetEditorCurve(clip, bind, null);
>>         }
>>         EditorUtility.SetDirty(clip);
>>         // 重新保存
>>         AssetDatabase.SaveAssets();
>>     }
>> }
>> ```
> 剔除`Scale`前
> ![png](/images/computer/game/unity/CustomAnimation2.jpg)
> 
> 剔除`Scale`后
> ![png](/images/computer/game/unity/CustomAnimation3.jpg)
> 
> 前后差了0.8K
> 
> * 浮点数压缩
> 顾名思义，就是对animation曲线上的高精度浮点进行压缩低精度
>> ```
>> [MenuItem("Assets/CustomAnimation/优化曲线精度")]
>> static void OptionalFloatCurves()
>> {
>>     // 保留两位小数，看需求定义
>>     var floatFormat = "f2";
>>     // 需要直接选中AnimationClip
>>     var animation_go = Selection.activeObject;
>>     if (animation_go.GetType() == typeof(AnimationClip))
>>     {
>>         // clip
>>         var clip = animation_go as AnimationClip;
>>         // 获取Animation的所有Curve
>>         var binds = AnimationUtility.GetCurveBindings(clip);
>>         foreach (var bind in binds)
>>         {
>>             var curve = AnimationUtility.GetEditorCurve(clip, bind);
>>             var keys = curve.keys;
>>             for (int index=0; index< keys.Length; index++)
>>             {
>>                 var keyframe = keys[index];
>>                 keyframe.value = float.Parse(keyframe.value.ToString(floatFormat));
>>                 keyframe.inTangent = float.Parse(keyframe.inTangent.ToString(floatFormat));
>>                 keyframe.outTangent = float.Parse(keyframe.outTangent.ToString(floatFormat));
>> 
>>                 keys[index] = keyframe;
>>             }
>>             // struct 需要重新指定
>>             curve.keys = keys;
>>             // 重新指定
>>             AnimationUtility.SetEditorCurve(clip, bind, curve);
>>         }
>>         EditorUtility.SetDirty(clip);
>>         // 重新保存
>>         AssetDatabase.SaveAssets();
>>     }
>> }
>> ```
>
> 在上一章的剔除`Scale`的基础上再次压缩精度后
> ![png](/images/computer/game/unity/CustomAnimation4.jpg)
> 直接优化了8.9K
> * 恒量(变化小的关键帧)
> 比较曲线的输入和输出变化，如果是基本稳定，即为常量
>> ```
>> [MenuItem("Assets/CustomAnimation/优化小变化曲线")]
>> static void OptionalConstantCurves()
>> {
>>     // 需要直接选中AnimationClip
>>     var animation_go = Selection.activeObject;
>>     if (animation_go.GetType() == typeof(AnimationClip))
>>     {
>>         // clip
>>         var clip = animation_go as AnimationClip;
>>         // 获取Animation的所有Curve
>>         var binds = AnimationUtility.GetCurveBindings(clip);
>>         foreach (var bind in binds)
>>         {
>>             var curve = AnimationUtility.GetEditorCurve(clip, bind);
>>             var keys = curve.keys;
>>             // 移除常量曲线中的冗余帧
>>             int beginIdx = -1;
>>             // 要移除的帧
>>             List<int> removeIdxs = new List<int>();
>>             for (int index=0; index< keys.Length; index++)
>>             {
>>                 var keyframe = keys[index];
>>                 // 是否是异常量关键帧或者无限接近0
>>                 if ((float.IsInfinity(keyframe.inTangent) && float.IsInfinity(keyframe.outTangent))
>>                     || (Mathf.Abs(keyframe.inTangent) <= float.Epsilon && Mathf.Abs(keyframe.outTangent) <= float.Epsilon))
>>                 {
>>                     removeIdxs.Add(index);
>>                 }
>>             }
>>             // 移除
>>             for (int r = removeIdxs.Count - 1; r >= 0; --r)
>>             {
>>                 curve.RemoveKey(removeIdxs[r]);
>>             }
>>             // 如果curve的关键帧已经没有了，直接移除空的curve
>>             if (curve.length == 0)
>>             {
>>                 AnimationUtility.SetEditorCurve(clip, bind, null);
>>             }
>>             else
>>             {
>>                 // 重新指定
>>                 AnimationUtility.SetEditorCurve(clip, bind, curve);
>>             }
>>         }
>>         EditorUtility.SetDirty(clip);
>>         // 重新保存
>>         AssetDatabase.SaveAssets();
>>     }
>> }
>> ```
>> ![jpg](/images/computer/game/unity/CustomAnimation5.jpg)
>>
>> 在精度压缩的基础上再次剔除常量的变化曲线，内存上直接少了1.1K