---
layout:     article
title:      "Unity shader variant的使用"
subtitle:   "shader variant"
date:       2020-12-30 13:22:00
author:     "Conerlius"
category: unity
keywords: 
    - unity
    - shader
tags:
    - unity
---

## 纲要

- Shader Variant的出现
- Shader Variant的数量
- Shader Variant的优化

### Shader Variant的出现

先直接丢一个[link](https://blogs.unity3d.com/cn/2018/05/14/stripping-scriptable-shader-variants/)给大家学习


### Shader Variant的数量

- pass variant
- subshader variant
- shader variant
- total variant

一下公式都可以在前面给的文章[link](https://blogs.unity3d.com/cn/2018/05/14/stripping-scriptable-shader-variants/)里找到,这里是问了方便阅览就再copy了一下

#### pass variant

$$
PassVariants=Stages\coprod_{d=1}^{Directives}Keywords_{(d)}
$$

#### subshader variant

$$
SubShaderVariants=\sum_{p=1}^{Passes}\left ( PassVariants \right )
$$

#### shader variant

$$
ShaderVariants=\sum_{s=1}^{SubShaders}\left ( SubShaderVariants \right )
$$

#### total variant

$$
TotalVariants=\sum_{a=1}^{ShaderAssets}\left ( ShaderVariants \right )
$$

整合后的公式是：
$$
TotalVariants=\sum_{a=1}^{ShaderAssets}\sum_{s=1}^{SubShaders}\sum_{p=1}^{Passes}\left ( Stages_{(a,s,p)}\coprod_{d=1}^{Directives_{a,s,p}}Keywords_{(a,s,p,d)} \right )
$$


现在问题来了，既然有了上面的文章，为什么还需要写这么一篇文章呢？是闲么？

前面的link里说了大量的公式，注意事项，但是在真实项目使用中却不是很好用，而且问题还不少，所以才出现了这篇文章去记录（如果有大牛知道其他更加简单便捷的方法，望告知）。

### Shader Variant的优化

- 编译时间优化
- 加载时间优化
- OnProcessShader处理

#### 编译时间优化

`Unity`的`multi_compile keyword1 ……`（或`multi_compile_local`）会产生大量的`variant`的组合，而unity对每个`variant`都进行编译，从而使得编译耗时很长，前面的link有说到！如果想实现剔除一些没有必要的变体，使用`shader_feature`(或`shader_feature_local`)来减少`variant`

使用`multi_compile`的时候，有3.15k个variant

![png](/images/Unity/unity_shadervariant_1.png)

全部更换成使用`shader_feature`的时候，就变成只有11个variant了

![png](/images/Unity/unity_shadervariant_2.png)

这样在unity打包编译shader的时候，Unity就会剔除了不使用的`shader variant`了。

现在先上一下前后的比较图：

修改前:

![png](/images/Unity/unity_shadervariant_3.png)

修改后:

本来是想要截图的，但是发现变体变少了之后，一下子就闪过了，没有来得及截图那么快，结论是减少到了5个`variant`（视具体使用情况而定的！）

#### 加载时间优化

`Unity`提供了一个针对`shader`很有效的Unity运行时编译机制，有了解的同学都知道--`ShaderVariants`;

#### OnProcessShader处理

