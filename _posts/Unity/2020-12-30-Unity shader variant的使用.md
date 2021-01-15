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
- Shader Variant采集工具

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

`ShaderVariants`作为一个运行时`Untiy`编译Shader变体的符号集合，它能使得`Unity`在目标设备上产生指定的shader变体，通俗来讲，就是产生指定的`shaderprogram`，在这一个过程，`Unity`需要时间，如果变体量比较大的时候，就会消耗大量的启动时间，或者是在需要使用的时候阻塞式生成，（行程卡顿）所以需要`ShaderVariant`去优化这些。

具体的`ShaderVariant`这里就不讲了，百度谷歌很多资料。

#### OnProcessShader处理

`OnProcessShader`是`Unity`提供的一个shader编程回调接口，使得我们能对`Unity`的正在编译的`Shader`变体进行剔除，增加之类的自定义操作；

```c#
/// 一个简单排除Unity内建变体编译的处理器
class BuiltinShaderPreprocessor : IPreprocessShaders {
    static ShaderKeyword[] s_uselessKeywords;
    public int callbackOrder {
        get { return 0; } // 可以指定多个处理器之间回调的顺序
    }
    static BuiltinShaderPreprocessor() {
        s_uselessKeywords = new ShaderKeyword[] {
            new ShaderKeyword( "DIRLIGHTMAP_COMBINED" ),
            new ShaderKeyword( "LIGHTMAP_SHADOW_MIXING" ),
            new ShaderKeyword( "SHADOWS_SCREEN" ),
        };
    }
    public void OnProcessShader( Shader shader, ShaderSnippetData snippet, IList<ShaderCompilerData> data ) {
        for ( int i = data.Count - 1; i >= 0; --i ) {
            for ( int j = 0; j < s_uselessKeywords.Length; ++j ) {
                if ( data[ i ].shaderKeywordSet.IsEnabled( s_uselessKeywords[ j ] ) ) {
                    data.RemoveAt( i );
                    break;
                }
            }
        }
    }
}
```

### Shader Variant采集工具

目前能找到的采集方式在百度，谷歌之类的都能搜索到，大致是一下几个步骤：

1. 创建新场景
2. 使用反射执行`ShaderUtil.ClearCurrentShaderVariantCollection`
3. 选中`mainCamera`,执行`GameObject/Align View to Selected`
4. 遍历所有的Material,注意需要等一帧渲染
5. 遍历指定Material动态添加的keyword
6. 遍历Scene,也需要等几帧渲染
7. 反射执行`ShaderUtil.SaveCurrentShaderVariantCollection`

> 下面是需要注意的

在上面的步骤的时候，网上千篇一律没有提到一个重点，（Unity2020.2.1f1）上述的步骤生成的ShaderVariants并不是正确的（但是如果是第一次就是对的！），需要清理`Library/ShaderCache`目录，再执行上述步骤才能生成正确的！

### shader variant其他优化

说起这个，网上一大堆资料，但是都是需要特定的处理

[link](https://zhuanlan.zhihu.com/p/91820630)

总结一下大致如下：

- shader在书写的时候就要主要处理好`shader_feature`和`multi_compile`，如果是多个项目使用的共同的shader(不同项目需要生效的特性也许会不不一样)，这样的话，就需要每个项目有个特定的成员去处理和修改其keyword了；
- 有些keyword在代码中enable的时候，使用shader_feature的时候，没有被打包编译

基于上面两点，我目前使用的方式，全部使用`multi_compile`或`multi_compile_local`处理所有的keyword，然后unity在打包assetbundle的时候，会唤起前文提到的`OnProcessShader`，我们可以依据`Shader Variant`采集到的变体进行剔除，不使用的就从`IList<ShaderCompilerData> data`列表中remove，这样就使得unity编译的变体大大减少，还能减少打包时间。