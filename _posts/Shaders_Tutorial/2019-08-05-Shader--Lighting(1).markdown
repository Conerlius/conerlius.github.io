---
layout:     post
title:      "Shader--Lighting(1)"
subtitle:   " \"Light\""
date:       2019-08-05 17:24:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - Shader
---

> 经过前面的几波操作，我们有了一个比较基本的shader了，但是认真看看，发先，alpha没有，重要的是光照也没有；alpha没有，我们能忍，但光照没有的话，我们就不能忍了！<br>
> 本来是想先讲讲光照模型的，但发现，不少人（比如我）挺讨厌先理论的。所以我们直接以一种光照模型讲！<br>
> 首先要说明的是unity把光照的数据及封装好的函数都放置在`Lighting.cginc`，所以们需要使用光照的时候，别忘了导入该头文件
```
// 引入头文件 一定要放在CGPROGRAM\ENDCG之间
#include "Lighting.cginc"
```

光的反射，相信大家在初中已经学过，我这里就post上图<br>
![png](/images/shader_tutorial/shader-light0.png)<br>
通过上图，我们知道法线和光线的入射角度是必须的，那么这两个数据要怎么获取？
1. Normal
> 法线一般都是来自于模型，当然也可以用其他方式取代（这里就不讲了）,应该还记得上一章的Texture是怎么加进去的么？答案是方式一样。
```
// 定义系统对vert需要传入的参数
struct a2v 
{
	// 模型内顶点坐标,类型参考语义部分
	float4 pos:POSITION;
	// 纹理坐标,类型参考语义部分
    float4 uv:TEXCOORD0;
	// 法线,不是float4
	float3 normal:NORMAL;
};
// vert对frag输出的数据
struct v2f
{
	float4 pos:POSITION;
	//将取到的纹理坐标传递到片元函数
    float4 uv:TEXCOORD0;
	// 法线
	float3 normal:NORMAL;
};
```

> 因为`Lighting.cginc`已经做了数据的记录，所以我们不用像`Texture`那样绑定一下。

2. 入射角度
> 这里我们先说直照光，点光我们后面再另开一章细说！`Lighting.cginc`里为我们提供了大量的光照相关的参数和方法。目前我们用到的就有以下一个，我们现在记住就好：
>> `_WorldSpaceLightPos0` 光源0的数据，其中`_WorldSpaceLightPos0.w`可以用是否为0来判定是不是直照光；`_WorldSpaceLightPos0.xyz`如果是直照光，`.xyz`就是直照光的的方向；目前我们仅考虑直照光，所以我们只用到`_WorldSpaceLightPos0.xyz`这个光照方向就好。

3. 光线颜色
> 也是直照光，点光我们后面再另开一章细说！。目前我们用到的就有以下一个，我们现在记住就好：
>> `_LightColor0` 光源0的数据，`_LightColor0.rgb`就是光源0的颜色；

到此，所有我们需要的数据都有了，那么反光的计算公式呢？光照模型常见的有几个，这里只讲`Lambert模型`
> Disffuse = 直射光颜色*max(0,cosθ) (θ是法线跟光照的夹角)
>> 数学基础就不说了~

```math
diffuse = _LightColor0.rgb * max(0, dot(o.normal, lightDir));
```

> 把这个公式应用到frame里:

```
Shader "WDFramework/VertShader"
{
	// 属性
	Properties{
		// 颜色
		_Color("Main Color", Color) = (1,1,1,1)
		// 主纹理
		_MainTex("Main Texture", 2D) = "white"{}
	}
	
    SubShader
    {
        Tags { "RenderType"="Transparent" }
        LOD 100
		
        Pass
        {
			
            CGPROGRAM
			// 引入头文件 一定要放在CGPROGRAM\ENDCG之间
			#include "Lighting.cginc"
			// 声明指定vertex 和fragment的方法
			#pragma vertex vert
            #pragma fragment frag
			// 颜色
			float4 _Color;
			// 纹理
			sampler2D _MainTex;
			// 定义系统对vert需要传入的参数
			struct a2v 
			{
				// 模型内顶点坐标,类型参考语义部分
				float4 pos:POSITION;
				// 纹理坐标,类型参考语义部分
                float4 uv:TEXCOORD0;
				// 法线,不是float4
				float3 normal:NORMAL;
			};
			// vert对frag输出的数据
			struct v2f
			{
				float4 pos:POSITION;
				//将取到的纹理坐标传递到片元函数
                float4 uv:TEXCOORD0;
				// 法线
				float3 normal:NORMAL;
			};
			// 因为返回的已经不单单是顶点坐标了，我们就去掉SV_POSITION的声明
			v2f vert(a2v IN)// : SV_POSITION
			{
				v2f o;
				// 转换到世界裁剪空间
				o.pos = UnityObjectToClipPos(IN.pos);
				// 从模型空间转到世界空间 float3x3 将一个4*4的矩阵强转成3*3的矩阵
				o.normal = normalize(mul(IN.normal, (float3x3)unity_WorldToObject));
				// 保存uv
				o.uv = IN.uv;
				return o;
			}
			// 像素插值输出方法
			float4 frag(v2f o ):SV_Target
			{
				// 从纹理中回去颜色
				float4 col = tex2D(_MainTex, o.uv)*_Color;
				float3  lightDir = _WorldSpaceLightPos0.xyz;
				//取得反射的颜色
				float3 diffuse = _LightColor0.rgb * max(0, dot(o.normal, lightDir));
				// 转换成4维，方便对color
				float4 diffuse4 = float4(diffuse, 1);
				col = col*diffuse4;
				return col;
			}
            ENDCG
        }
    }
}
```

> 注：在unity Scene的时候调整光的数据，是不会立即生效的，要切到Game才能生效！

![png](/images/shader_tutorial/shader-light1.png)