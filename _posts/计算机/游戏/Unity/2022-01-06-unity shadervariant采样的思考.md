---
layout:     article
title:      "unity shadervariant采样的思考"
subtitle:   " \"unity\""
date:       2022-01-06 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - shader
---

## 大纲

- 背景
- 思路
- 解决过程
- 目前结论

## 背景

项目使用的shader变体特别多！如果是让TA独立处理太耗人力了，所以想想看看有没有其他的方法可以自动化，但是`CI`的发现很多都需要等unity渲染后反射`ShaderUtil.SaveCurrentShaderVariantCollection`;想抛弃这样的方案看看有没有其他的！

## 思路

对`shader`的要求

1. 自定义的`keyword`必须是`local`的
2. `pass`中必须声明`LightMode`

**想法中的思路1**

1. 获取`material`的`keyword`
2. 剔除`Global keyword`
3. 移除残留的自定义`keyword`后得到后续使用的`keywords`(记为`k1`)
4. 判定`shader`是否包含`DephtOnly``Meta`等，以及是不是`CGProgram`之类，然后建`k1`和这些类型构建`shadervariant`添加到`shadervariantcollection`
5. 针对代码开启的`keyword`，采用以下两种方式：
   1. 手动添加固定的`shadervariant`，在前面的采样执行完成后`merge`到特定的`shadervariantcollection`中
   2. 创建一个已经开启该`keyword`的材质
6. 在`OnProcessShader`的时候，将`IList<ShaderCompilerData>`清空，把`svc`的变体添加进去就可以了~

**想法中的思路2**

1. 获取`material`的`keyword`
2. 剔除`Global keyword`
3. 移除残留的自定义`keyword`后得到后续使用的`keywords`(记为`k1`)
4. 遍历每个`subshader`中每个`pass`
5. 根据`pass`中`lightmode`指定`passtype`，将`keywords`和`passtype`生成特定的`shadervariant`
6. 遍历所有的场景设置，记录场景有的`Global keyword`
7. 将`Global keyword`和`k1`组合出新的`shadervariant`
8. 针对代码开启的`keyword`，采用以下两种方式：
   1. 手动添加固定的`shadervariant`，在前面的采样执行完成后`merge`到特定的`shadervariantcollection`中
   2. 创建一个已经开启该`keyword`的材质
9. 在`OnProcessShader`的时候，将`IList<ShaderCompilerData>`清空，把`svc`的变体添加进去就可以了~

两个步骤的有很大一部分是相同的！

## 解决过程

通过下面的代码可以直接获取到`material`的`keyword`

```c#
var GetShaderGlobalKeywords_MethodInfo = typeof(ShaderUtil).GetMethod("GetShaderGlobalKeywords",
                BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
var GetShaderLocalKeywords_MethodInfo = typeof(ShaderUtil).GetMethod("GetShaderLocalKeywords",
                BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
var global_keywords = new List<string>((string[])GetShaderGlobalKeywords_MethodInfo.Invoke(null, new object[] { shader }));
var local_keywords = new List<string>((string[])GetShaderLocalKeywords_MethodInfo.Invoke(null, new object[] { shader }));

var mat = AssetDatabase.LoadAssetAtPath<Material>(material_path);
var keywords = new List<string>(mat.shaderKeywords);
```

然后就是移除`global`和*无效的自定义keyword*

```c#
// 移除所有的全局keyword
tmp_list = new List<string>(keywords.ToArray());
foreach (var keyword in tmp_list)
{
    if (global_keywords.Contains(keyword))
    {
        keywords.Remove(keyword);
    }
}
// 移除自定义
// 自定义的需要遵守规则，使用_local
var tmp_list = new List<string>(keywords.ToArray());
foreach (var keyword in tmp_list)
{
    if (!local_keywords.Contains(keyword))
    {
        keywords.Remove(keyword);
    }
}
```

在ShaderUtil中能看到`internal static extern bool HasShadowCasterPass([NotNull("ArgumentNullException")] Shader s);`的方法，但是这个只能够给我们判定`shadowcaster`而已，其他的就没有了

再看看`shader.cs`，在代码里能看到`public ShaderTagId FindPassTagValue(int passIndex, ShaderTagId tagName)`

使用一下的代码尝试获取！

```c#
var shader = mat.shader;
ShaderTagId m_ShaderTagId = new ShaderTagId("DepthOnly");
var shaderTagId = shader.FindPassTagValue(passIndex, m_ShaderTagId);
```

但是发现无论是直接从mat中获取还是`Instance`实例到`hierachy`中，都是不行；

又尝试了一下改`shadertagId`

```c#
ShaderTagId m_ShaderTagId = new ShaderTagId("LightMode");
```

也是不可以！

到这里就只能放弃了这个方法！

***现在开始尝试一下另一种方案***

`subshader`的获取

```c#
MethodInfo GetShaderSubshaderCount_MethodInfo = typeof(ShaderUtil).GetMethod("GetShaderSubshaderCount",
                BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
MethodInfo GetShaderTotalPassCount_MethodInfo = typeof(ShaderUtil).GetMethod("GetShaderTotalPassCount",
                BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
int subShaderCount = (int)GetShaderSubshaderCount_MethodInfo.Invoke(null, new object[] { shader });
int passCount = (int)GetShaderTotalPassCount_MethodInfo.Invoke(null, new object[] { shader, subIndex });

```

遍历所有的`pass`

```c#
for (int subIndex = 0; subIndex < subShaderCount; subIndex++)
{
    int passCount = (int)GetShaderTotalPassCount_MethodInfo.Invoke(null, new object[] { shader, subIndex });
    for (int passIndex = 0; passIndex < passCount; passIndex++)
    {
        // code here
    }
}
```

又到了获取`pass`的`passType`的时候了！

在这里我目前能想到的只能是正则了~

首先获取`pass`的`line number`

```c#
MethodInfo GetShaderPassSourceCode_MethodInfo = typeof(ShaderUtil).GetMethod("GetShaderPassSourceCode",
                BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic);
string source = GetShaderPassSourceCode_MethodInfo.Invoke(null, new object[] { shader, subIndex, passIndex }) as string;
```

`source`一般是类似以下的形式（注意，这里的`source`并不是`Unity shader`书写的代码！）

> #line 99 ""\n其他代码

使用正则提取行！

```c#
Regex lineRegex = new Regex("#line ([0-9]*)");
var match = lineRegex.Match(source);
int lineNum = System.Convert.ToInt32(match.Groups[1].Value)
```

会有多个匹配的，但是一般我们只需要第一行的！

然后我们查找`lineNum`最近且在其前面的`LightMode`；这里又需要解释`Shader`源文件了~

```c#
var shader_path = AssetDatabase.GetAssetPath(shader);
string[] shader_source = File.ReadAllLines(shader_path);

Regex lgohtmode_regex = new Regex("#line (\\d{1,4}) #[ \\t]*[{\\t\"= \\w]*\"LightMode\"[ \\s]*=[ \\s]*\"(.*)\"");
StringBuilder sb = new StringBuilder();
for (int index = 0; index < shaderSource.Length; index++)
{
    sb.AppendLine($"#line {index} #{shaderSource[index]}");
}
int min_match_line = 99999;
string lightMode = "Nothing";
var matches = lgohtmode_regex.Matches(sb.ToString());
for (int index = 0; index < matches.Count; index++)
{
    int line = System.Convert.ToInt32(matches[index].Groups[Value);
    if (line_num - line > 0 && min_match_line >= line_num -line)
    {
        lightMode = matches[index].Groups[2].Value;
        min_match_line = line_num - line;
    }
}
```

到此我们就获取到了`LightMode`了！

然后尝试构建`shadervariant`

```c#
switch (lightMode_string)
{
    case "ShadowCaster":
    {
        var shadervariant = new ShaderVariantCollection.ShaderVariant(shader, PassType.ShadowCaster, keywords.ToArray());
        svc.Add(shadervariant);
    }
        break;
}
```

但是就这么个例子中！执行的时候就报错了！！

> ArgumentException: passed shader keyword variant not found in shader XXXshader pass type 8

到此就需要判定我们之前提出的`k1`中有哪些是`ShadowCaster`中没有！但是找了一轮，没有发现有什么接口和方法可以搞定的；

> 也许继续用正则可以解释出来，这个后面有时间再试试！

## 目前的结论

有点麻烦，这两个方案到此就都放弃了！