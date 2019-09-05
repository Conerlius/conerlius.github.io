---
layout:     post
title:      "Unity Animation优化"
subtitle:   " \"Animation优化\""
date:       2019-09-05 18:00:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - Unity;
---

> Unity的动画系统一直比较让人无奈的，谁用谁知道！但是Unity的同时也提供了另外的一套解决方案，可以优化动画带来的部分问题
> * 减少有动作的骨骼数量
>> 一个骨骼(Transform)是有position(x,y,z),rotation(x,y,z,w),scale(x,y,z)3个数据，在unity里，我们可以打开animation观察
>> ![png](/images/Unity/CustomAnimation1.jpg)
>>
>> 一个Position,`x,y,z`三个数据，三条curve那么一个骨骼就有3(position)+4(rotation)+3(scale)=10个curve，少一个带动画的骨骼，那么就直接少了10条curve
> * Curve曲线的剔除
> 
> * 浮点数压缩
> 
> * 恒量曲线剔除
> * 