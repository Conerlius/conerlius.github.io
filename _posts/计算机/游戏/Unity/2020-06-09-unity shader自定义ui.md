---
layout:     article
title:      "unity shader自定义ui"
subtitle:   " \"unity\""
date:       2020-06-09 17:00:00
author:     "Conerlius"
category: unity
keywords: unity,shader
tags:
    - unity
    - shader
---

## 例子使用的shader

```shader
Shader "Unlit/CustomShaderUI"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _MainColor("Main Color", Color)=(0.5, 0.5, 0.5, 1.0)
    }
    SubShader
    {
        Tags { "RenderType"="Opaque"}
        // 简单的pass
        Pass
        {
            HLSLPROGRAM
            #pragma vertex LitPassVertex
            #pragma fragment LitPassFragment
            
            float4x4 unity_ObjectToWorld;
            float4x4 unity_WorldToObject;
            float4 unity_LODFade;

            float4x4 unity_MatrixVP;
            float4x4 unity_MatrixV;
            float4x4 glstate_matrix_projection;
            
            #define UNITY_MATRIX_M unity_ObjectToWorld
            #define UNITY_MATRIX_I_M unity_WorldToObject
            #define UNITY_MATRIX_V unity_MatrixV
            #define UNITY_MATRIX_VP unity_MatrixVP
            #define UNITY_MATRIX_P glstate_matrix_projection
            
            float4 _MainColor;
            
            float4 LitPassVertex(float3 positionOS: POSITION) : SV_POSITION 
            {
                return float4(positionOS, 1.0);
            }
            float4 LitPassFragment() : SV_TARGET 
            {
                return _MainColor;
            }
            ENDHLSL
        }
    }
    // 声明自定义的编辑器
    CustomEditor "Editor.CustomShaderGUI"
}
```

最后一行的`CustomEditor "Editor.CustomShaderGUI"`一定要声明，否则我们编写的`CustomShaderGUI`展示界面不会生效
> 注意，这里需要完整的名称空间

## ShaderGUI

### 创建类**CustomShaderGUI**

> 该类继承了`ShaderGUI`,重载`OnGUI`方法

```c#
using UnityEditor;

namespace Editor
{
	/// <summary>
	/// 自定义的ShaderGUI
	/// </summary>
	public class CustomShaderGUI : ShaderGUI
	{
		/// <summary>
		/// override
		/// </summary>
		public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
		{
			base.OnGUI(materialEditor, properties);
		}
	}
}
```

`OnGUI`方法里有几个对象是比较重要的

- materialEditor

> 它是负责显示和编辑材料的基础编辑器对象。

- properties

> 所有的shader属性

- materialEditor.Target

> 被引用编辑的对象

ShaderGUI的写法和Editor的扩展是一样的！

![png](/images/computer/game/unity/shader自定义ui.png)

下面就直接摆上代码好了。


```c#
using UnityEditor;
using UnityEngine;

namespace Editor
{
	/// <summary>
	/// 自定义的ShaderGUI
	/// </summary>
	public class CustomShaderGUI : ShaderGUI
	{
		MaterialEditor editor;
		Object[] materials;
		MaterialProperty[] properties;
		/// <summary>
		/// override
		/// </summary>
		public override void OnGUI(MaterialEditor materialEditor, MaterialProperty[] properties)
		{
			base.OnGUI(materialEditor, properties);
			editor = materialEditor;
			materials = materialEditor.targets;
			this.properties = properties;
			// 添加按钮
			AddButton1();
			GUILayout.Space(30f);
			GUILayout.Label("label");
			GUILayout.Toggle(true, "开启");
		}
		/// <summary>
		/// 重置颜色按钮
		/// </summary>
		void AddButton1 () {
			if (GUILayout.Button("reset color 1")) {
				// 设置颜色
				// 内置的FindProperty可以查找属性
				FindProperty("_MainColor", properties).colorValue = Color.white;
			}
		}
	}
}
```