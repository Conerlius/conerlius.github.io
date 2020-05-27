---
layout:     article
title:      "unity shader的inspacetor展示"
subtitle:   " \"unity\""
date:       2020-05-27 15:00:00
author:     "Conerlius"
category: unity
keywords: unity,shader
tags:
    - unity
---

## 官方的文档

https://docs.unity3d.com/ScriptReference/MaterialPropertyDrawer.html

## 例子

```shader
Shader "CustomRP/AttrShader"
{
    Properties
    {
        // 图集
        _MainTex("Texture", 2D) = "white" {}
        // 在inspector界面中隐藏
        [HideInInspector] _MainTex2("Hide Texture", 2D) = "white" {}
        // 没有`Tilling`和`Offset`
        [NoScaleOffset] _MainTex3("No Scale/Offset Texture", 2D) = "white" {}
        // 前置，通常为 MaterialPropertyBlock使用，隐藏
        [PerRendererData] _MainTex4("PerRenderer Texture", 2D) = "white" {}
        
        _Color("Color", Color) = (1,0,0,1)
        // hdr
        [HDR] _HDRColor("HDR Color", Color) = (1,0,0,1)

        _Vector("Vector", Vector) = (0,0,0,0)
         
        // 文本标题，一般使用进行归类(目测只能使用英文)
        [Header(A group of things)]
         
        // 使用默认的"_INVERT_ON"作为关键词
        [Toggle] _Invert("Auto keyword toggle", Float) = 0
         
        // 使用默认的"ENABLE_FANCY"作为关键词
        [Toggle(ENABLE_FANCY)] _Fancy("Keyword toggle", Float) = 0

        // Blend模式 
        [Enum(UnityEngine.Rendering.BlendMode)] _Blend("Blend mode Enum", Float) = 1

        [Enum(One,1,SrcAlpha,5)] _Blend2("Blend mode subset", Float) = 1
        // 指定的关键词选择，转换成`_Overlay_None`、`_Overlay_Add`、`_Overlay_Multiply`
        [KeywordEnum(None, Add, Multiply)] _Overlay("Keyword Enum", Float) = 0

        // 非线性的滑动条
        [PowerSlider(3.0)] _Shininess("Power Slider", Range(0.01, 1)) = 0.08
         
        // 指定范围的滑动条
        [IntRange] _Alpha("Int Range", Range(0, 255)) = 100
         
        // 和上面一条属性在inspactor界面上的垂直距离（此处是默认距离）
        [Space] _Prop1("Small amount of space", Float) = 0
         
        // 和上面一条属性在inspactor界面上的垂直距离（此处是50）
        [Space(50)] _Prop2("Large amount of space", Float) = 0
    }
    SubShader
    {
        Tags {
				"LightMode" = "CustomLit"
			}
        Pass
        {
            HLSLPROGRAM
            #pragma target 3.0
            #pragma vertex LitPassVertex
			#pragma fragment LitPassFragment
			#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"
           
            CBUFFER_START(UnityPerDraw)
            float4x4 unity_ObjectToWorld;
            float4x4 unity_WorldToObject;
            float4 unity_LODFade;
            real4 unity_WorldTransformParams;
            CBUFFER_END
            
            
            float4x4 unity_MatrixVP;
            float4x4 unity_MatrixV;
            float4x4 glstate_matrix_projection;
            
            #define UNITY_MATRIX_M unity_ObjectToWorld
            #define UNITY_MATRIX_I_M unity_WorldToObject
            #define UNITY_MATRIX_V unity_MatrixV
            #define UNITY_MATRIX_VP unity_MatrixVP
            #define UNITY_MATRIX_P glstate_matrix_projection
            
            #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/SpaceTransforms.hlsl"
            
            float4 LitPassVertex (float3 pos : POSITION) : SV_POSITION {
                return float4(TransformObjectToWorld(pos), 1);
            }

            float4 LitPassFragment (): SV_TARGET {
                return float4(1,1,1,1);
            }
            ENDHLSL
        }
    }
}

```