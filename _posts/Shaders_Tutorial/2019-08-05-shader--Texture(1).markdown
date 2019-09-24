---
layout:     post
title:      "shader--Texture(1)"
subtitle:   " \"Texture\""
date:       2019-08-05 17:20:00
author:     "Conerlius"
category: shader
keywords: shader,unity
tags:
    - shader
---

前面两章说了一些基本的内容，我们做Shader，怎么可能烧的了贴图纹理呢？所以这章我们来讲讲纹理的使用。<br>
> 首先我们像之前声明Color那样去声明和绑定一个纹理<br>

```
// 属性
Properties{
	// 颜色
	_Color("Main Color", Color) = (1,1,1,1)
	// 主纹理
	_MainTex("Main Texture", 2D) = "white"{}
}
...
// 颜色
float4 _Color;
// 纹理
sampler2D _MainTex;
```
> 如此的时候，我们已经绑定完成了，那么我们要怎么处理`Vert`和`Frag`呢？<br>
> 这里说一下`Vert`和`Frag`到底是干嘛的吧<br>
>> `Vertext`是顶点着色器，GPU把进过处理后的顶点数据逐一传进`Vertex`做编程处理。
>> `Fragment`是像素，是GPU对顶点插值、裁剪、投影等等一系列后转换的像素着色器。

> 在说一个小理论，不深入；模型的顶点数据和纹理是的映射对应关系是通过`UV`的(UV是啥，自己百度了)；

> 了解`Vertex`和`Fragment`后，我们想想第一章的时候我们是怎么把顶点左边传进
`Vert`的？<br>
> 在`a2v`里声明,这样GPU就会把uv信息告诉`Vert`了

```
// 定义系统对vert需要传入的参数
struct a2v 
{
	// 模型内顶点坐标,类型参考语义部分
	float4 pos:POSITION;
	// 纹理坐标,类型参考语义部分
    float4 uv:TEXCOORD0;
};
```
> 注意一点，模型的数据只有在vert里能拿到，在`Frag`里不能直接取的，但是呢，我们可以通过`a2v`来获取，存放到`v2f`里，这样`Frag`就可以间接取到了！

```
// vert对frag输出的数据
struct v2f
{
	float4 pos:POSITION;
	//将取到的纹理坐标传递到片元函数
    float4 uv:TEXCOORD0;
};
```

> 我们来写写`Vert`吧

```
// 因为返回的已经不单单是顶点坐标了，我们就去掉SV_POSITION的声明
v2f vert(a2v IN)// : SV_POSITION
{
	v2f o;
	// 转换到世界裁剪空间
	o.pos = UnityObjectToClipPos(IN.pos);
	// 保存uv
	o.uv = IN.uv;
	return o;
}
```

> Fragment

```
// 像素插值输出方法
float4 frag(v2f o ):SV_Target
{
	return _Color;
}
```

> 现在我们在`v2f o`里可以取得到UV，那么我们如果去获取该像素点上的纹理颜色呢？Unity为我们提供了api：`tex2D(Texture, UV)`；直接使用返回后我们得到的Shader和结果如下！

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
        Tags { "RenderType"="Opaque" }
        LOD 100
		
        Pass
        {
            CGPROGRAM
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
			};
			// vert对frag输出的数据
			struct v2f
			{
				float4 pos:POSITION;
				//将取到的纹理坐标传递到片元函数
                float4 uv:TEXCOORD0;
			};
			// 因为返回的已经不单单是顶点坐标了，我们就去掉SV_POSITION的声明
			v2f vert(a2v IN)// : SV_POSITION
			{
				v2f o;
				// 转换到世界裁剪空间
				o.pos = UnityObjectToClipPos(IN.pos);
				// 保存uv
				o.uv = IN.uv;
				return o;
			}
			// 像素插值输出方法
			float4 frag(v2f o ):SV_Target
			{
				// 从纹理中回去颜色
				float4 col = tex2D(_MainTex, o.uv);
				return col;
			}
            ENDCG
        }
    }
}
```

> 预览图
![png](/images/shader_tutorial/shader_texture_1.png)

> 试着玩一下，发现Main Color无效了！<br>
> 查看一下，`_Color`居然没有用了！那怎么使之生效有能和texture混合呢？
> 简单粗暴的方式就`float4 col = tex2D(_MainTex, o.uv)*_Color;`,这样就直接生效了。
> 这里还有个问题，那就是透明度居然不生效！！！<br>
> 这个我们留着后面讲
