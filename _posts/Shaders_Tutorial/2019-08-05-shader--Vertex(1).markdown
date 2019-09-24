---
layout:     post
title:      "shader--Vertex(1)"
subtitle:   " \"Vertex\""
date:       2019-08-05 17:14:00
author:     "Conerlius"
category: shader
keywords: shader,unity
tags:
    - shader
---

## 声明
> 这里先讲Vertex/Fragment，至于ShaderLab，有时间在整理。<br>
> 本人是自己乱学的shader，属野鸡流，没有什么系统地学，但应该比较适合还懵懵的新人吧！<br>
> 直接正文

## 最简单的Shader
> 这里不说渲染管线的流程，什么几何阶段都不理，直接上手`Shader`;
```
Shader "WDFramework/VertShader"
{
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
			// float4 v:POSITION=>直接指定告诉编译器参数的意义
			float4 vert(float4 v:POSITION) :SV_POSITION
			{
				// 转换到世界自由空间
				return UnityObjectToClipPos(v);
			}
			// 像素插值输出方法
			float4 frag():SV_Target
			{
				return fixed4(0,1,1,1);
			}
            ENDCG
        }
    }
}
```
> 这个shader简单吧？但是是不是有好多参数或定义不明白？下面我们一个一个来盘点;<br>
> 在写Shader的时候，我们总需要有一个模板吧？
```
// 指定Shader路径和名字
Shader "WDFramework/Shader Temp"
{
    // 属性,主要是为了在unity的面板上展示和用于给CPU交互的！
    // 简单说现在可以理解为是供其他访问的
    Properties 
    {
        // 声明属性
    }
    // SubShader可以写很多个 GPU自上而下执行
    SubShader
    {
        //至少含有一个Pass
        Pass {
            //在这里编写Shader代码 HLSLPROGRAM
            CGPROGRAM
            //使用CG语言编写Shader代码
            ENDCG
        }
    }
    // 如果上面SubShader都不支持 则执行默认的Shader效果
    FallBack  "VertexLit" 
}
```
### Properties
> Properties的格式是
```
_Color("Color",Color)=(1,1,1,1)
// 变量名称(在unity面板上现实的名称, 变量类型)=变量的默认值
```
比较常用的有一下几个
```
// 颜色
_Color("Color",Color)=(1,1,1,1)
// 矩阵
_Vector("Vector",Vector)=(1,2,3,4)
// 整数
_Int("Int",Int)=2
// 浮点， 值是不用带f的
_Float("Float",Float)=12.3
// 浮点范围类型
_Range("Range",Range(1.0,10.0))=1.0
// 图片纹理类型， white是默认值图的颜色
_2D("Texture",2D)="white"{}
// 立方体纹理
_Cute("Cute",Cube)="red"{}
//3D纹理
_3D("Texture",3D)="black"{}
```
### CG
> 本文不讲HLSLPROGRAM, 这就直接说说CG的一些常用格式;CG都必须写在`CGPROGRAM`和`ENDCG`之间;<br>
```
// 声明指定vertex 和fragment的方法
#pragma vertex vert
#pragma fragment frag
```
> * Vertex
> 
>> ```
>> float4 vert(float4 v:POSITION) :SV_POSITION
>> {
>>      // 转换到世界自由空间
>>      return UnityObjectToClipPos(v);
>>}
>> ```
>> `float4 v:POSITION`这是指定调用`vert`方法的时候，告诉编译器参数是`POSITION`,`POSITION`是什么，怎么来的，后面`语义`再讲，现在只需要知道这样声明后，GPU调用`vert`的时候，会把渲染提的定点数据传进来;<br>
>> `SV_POSITION`是剪裁坐标语义,告诉系统，`vert`返回的是剪裁坐标

> * Fragment
> 
>> ```
>> float4 frag():SV_Target
>> {
>>      // 图元的颜色
>>      return fixed4(0,1,1,1);
>>}
>> ```
>> `SV_Target`是剪裁坐标语义,告诉系统，`frag`返回的是颜色
### 语义
> 从应用程序传递到顶点函数的语义有哪些(application to vertex,通常我们就命名成a2v) 
> 1. POSITION顶点坐标(模型空间下的) 
> 2. NORMAL法线(模型空间下) 
> 3. TARGET切线(模型空间) 
> 4. TEXCOORD0~n纹理坐标 
> 5. COLOR顶点颜色

---

>从顶点函数传递给片元函数的时候可以使用的语义 (vertex to fragment,通常我们就命名成v2f) 
> 1. SV_POSITION剪裁空间中的顶点坐标(一般是系统直接使用) 
> 2. COLOR0可以传递一组值 四个值 
> 3. COLOR1可以传递一组值 四个值 
> 4. TEXCOORD0~7传递纹理坐标

---

> 片元函数传递给系统<br>
> SV_TARGET颜色值，显示到屏幕上的颜色

前面讲了那么多，我们改一下前面的那个简单的demo;demo上的输出的颜色是固定的，我们改成我们能调整的；
> 为了能在unity面板上调整，就要用刚刚说到的`Properties`
```
// 属性
Properties{
	_Color("Main Color", Color) = (1,1,1,1)
}
```
> 位置参考一下前面说的模板的位置
> 创建个Material,把刚刚写的shader放上去（不会的自己百度了）,这个时候我们在unity Inspect看到一下样子
![png](/images/shader_tutorial/shader_vertex_1.png)
> 我们调整一下MainColor，发现Mesh并没有任何变化？为什么？<br>
> 原来我们只是声明，并没有参与到shader的计算;前文已经说过，`Properties`是开放外部用的，那么shader内部要使用的话，就需要再次在shader里声明（或理解成绑定）；
```
//float4就是四个值 _Color要跟上面属性名字保持一致
float4 _Color;
```
> 写到哪里？模板没有说呀！这个是一个CG，所以我们要写到`CGPROGRAM`和`ENDCG`间；既然说的绑定的问题，那么其他的属性类型应该绑定什么数据类型呢？一下是unity常见的几个
```
float4 _Color;
float4 _Vector;
float _Int;
float _Range;
sampler2D _2D;
samplerCube _Cube;
sampler3D _3D;
```
> 注：float还有几种类似的替代<br>
> float、half和fixed类型区别
```

float也可以用half和fixed代替 
float 32位来存储 
half 16位来存储 -6万~+6万 
fixed 11位来存储 一般都是用fixed
```
> 属性已经绑定了，那么我们就把改颜色输出到物体上，改动一下`frag`;
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
			// float4 v:POSITION=>直接指定告诉编译器参数的意义
			float4 vert(float4 v:POSITION) :SV_POSITION
			{
				// 转换到世界自由空间
				return UnityObjectToClipPos(v);
			}
			// 像素插值输出方法
			float4 frag():SV_Target
			{
			    // 直接输出
				return _Color;
			}
            ENDCG
        }
    }
}
```
效果如下
![png](/images/shader_tutorial/shader_vertex_2.png)