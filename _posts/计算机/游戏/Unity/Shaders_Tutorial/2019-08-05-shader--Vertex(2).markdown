---
layout:     article
title:      "shader--Vertex(2)"
subtitle:   " \"Vertex\""
date:       2019-08-05 17:17:00
author:     "Conerlius"
category: shader
keywords: shader,unity
tags:
    - shader
---

[继上一篇的后续]({% post_url 2019-08-05-shader--Vertex(1) %})
> 上一章我们讲了`Vertex`和`Fragment`那么这两者有什么关系呢？<br>
> 这里就牵扯到了渲染管线，但我们才刚开始，说那么深奥的知识，反而会觉得难！我们就大概说一下就好。<br>
> 首先是计算机应用程序通过IO把Mesh，贴图之类的加载到内存，之后传递给GPU，GPU拿到后，经过一系列处理，提取出需要的数据递交给`Vertex`,`Vertex`处理完后，GPU有处理了一下，然后把数据传入`Fragment`,`Fragment`还给GPU后，GPU处理成了我们看到的图形。（嗯，暂时这么理解吧，讲细的话，以我的能力，你懵我也懵！）大概可以先理解成一下的样子。

![png](/images/computer/game/unity/shader_tutorial/graph.png)


> 还记得上一章中的`float4 vert(float4 v:POSITION) :SV_POSITION`吗？我们发现这里决然只有一个参数，那如果我既想要定点坐标，也想要法线坐标呢？难道传两个参？<br>
> 答案是no,我们抽象一下参数结构出来

```
// 定义系统对vert需要传入的参数(application to vertex)
struct a2v 
{
	float4 v:POSITION;
};  // 最后一定要有一个分号，代表语句已经声明完成
```

> 既然传入Vert能抽象，那么Vert传递到Frag的数据也能抽象了

```
struct v2f
{
	float4 pos:POSITION;
};
```

> 经过两次的抽象后，我们得到一下的代码

```
Shader "WDFramework/VertShader"
{
	// 属性
	Properties{
		_Color("Main Color", Color) = (1,1,1,1)
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
			float4 _Color;
			// 定义系统对vert需要传入的参数
			struct a2v 
			{
				float4 v:POSITION;
			};
			// vert对frag输出的数据
			struct v2f
			{
				float4 pos:POSITION;
			};

			v2f vert(a2v IN) : SV_POSITION
			{
				v2f o;
				// 转换到世界自由空间
				o.pos = UnityObjectToClipPos(IN.v);
				return o;
			}
			// 像素插值输出方法
			float4 frag(v2f o ):SV_Target
			{
				return _Color;
			}
            ENDCG
        }
    }
}
```

> 这下子，如果我们需要多个输入或输出参数的时候，我们只需要在`a2v`或`v2f`里添加定义就行。<br>
> 注:声明的属性必须要声明其数据类型，忘记的可以回顾一下上一篇的`语义`部分。