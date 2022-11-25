---
layout:     article
title:      "使用renderFeature生成屏幕的遮罩"
subtitle:   " \"unity\""
date:       2022-11-25 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

之前有一个使用了`GPUInstance`简单实现类似`For the king`的那种地图遮挡效果的，这种随便百度一下就有了，就不在这里说了；

现在这个是简单使用`RenderFeature`实现的一个地图信息遮挡（因为只是简单出demo体验王法，就简单来）！

## 简单说明一下原理

第一步：使用一张RT,单独绘制场景中代表视野范围的六边形的格子，这张RT就是遮罩Mask;

第二步：正常渲染场景中的所有物件

第三步：在`Post-Process`前，理由Mask对相机正常渲染的Buffer进行Blit



## 阶段性的截图

渲染mask

![png](/images/computer/game/unity/unity_renderfeature_drawscreenhold1.png)

Blit后

![png](/images/computer/game/unity/unity_renderfeature_drawscreenhold2.png)



## 缺陷
这个其实挺多问题的，但是贵在性能好呀,而且实现简单~

- 因为是最后基于屏幕的blit，导致遮挡会直接在屏幕上，对于非垂直视图，裁剪就特别恶心了！
- 有人说类似billboard的方式贴瓦片，但是这样在相机比较水平的时候，那就能直接看到很远了~（同样都有这个问题）
资料

以下是这次使用的代码和shader

```shader
Shader "Custom/DrawViewer"
{
    Properties
    {
        _BaseTex("主纹理", 2D) = "white" {}
    }
    SubShader
    {
        Tags
        {
            "RenderType" = "Opaque"
        }

        Pass
        {
            ZWrite On
            ZTest On
            
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct MeshProperties
            {
                float4x4 mat;
                float status;
            };
            TEXTURE2D(_BaseTex);
            SAMPLER(sampler_BaseTex);
            float4 _BaseTex_ST;
            StructuredBuffer<MeshProperties> _Properties;

            struct Attributes
            {
                float4 positionOS   : POSITION;
                uint instanceID : SV_InstanceID;
                float2 texcoord     : TEXCOORD0;
            };
            struct Varyings
            {
                float4 positionCS   : SV_POSITION;
                float3 uv           : TEXCOORD0;
            };
            Varyings vert(Attributes input)
            {
                Varyings output = (Varyings)0;

                float4 pos = mul(_Properties[input.instanceID].mat, input.positionOS);
                output.positionCS = TransformWorldToHClip(pos);
                output.uv.xy = input.texcoord;
                output.uv.z = _Properties[input.instanceID].status;
                return output;
            }
            half4 frag(Varyings input) : SV_Target
            {
                if (input.uv.z > 0.5)
                    return half4(1,1,0,1);
                return half4(1,0,0, 1);
            }
            ENDHLSL
        }
    }
    FallBack "Hidden/Universal Render Pipeline/FallbackError"
}
Shader "Unlit/FogBlit"
{
    Properties
    {
        DigRT ("Texture", 2D) = "white" {}
        _MainTex ("Texture", 2D) = "white" {}
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 100

        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            TEXTURE2D(DigRT);
            SAMPLER(samplerDigRT);
            float4 DigRT_ST;

            TEXTURE2D(_MainTex);
            SAMPLER(sampler_MainTex);
            float4 _MainTex_ST;

            struct Attributes
            {
                float4 positionHCS   : POSITION;
                float2 uv           : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };
            struct Varyings
            {
                float4  positionCS  : SV_POSITION;
                float2  uv          : TEXCOORD0;
                UNITY_VERTEX_OUTPUT_STEREO
            };
            Varyings vert(Attributes input)
            {
                Varyings output;
                UNITY_SETUP_INSTANCE_ID(input);
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

                output.positionCS = TransformObjectToHClip(input.positionHCS);
                return output;
            }
            half4 frag(Varyings input) : SV_Target
            {
                UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);
                float4 color = SAMPLE_TEXTURE2D_X(DigRT, samplerDigRT, input.uv);
                float4 color2 = SAMPLE_TEXTURE2D_X(_MainTex, sampler_MainTex, input.uv);
                if (color.r < 0.5)
                {
                    return color;
                }
                return color2;
            }
            ENDHLSL
        }
    }
}

```

简单的feature代码

```c#
using System;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class DrawFullScreenViewerFeature : ScriptableRendererFeature
{
    /// <summary>
    /// 画视野使用的材质
    /// </summary>
    public Material material;
    /// <summary>
    /// 最后合成阶段的材质
    /// </summary>
    public Material blitMaterial;
    /// <summary>
    /// 每个视野格子的mesh
    /// </summary>
    public Mesh mesh;
    /// <summary>
    /// pass
    /// </summary>
    private DrawFullScreenViewerPass _fullScreenViewerPass;
    private BlitFullScreenViewerPass _blitFullScreenViewerPass;
    public RenderPassEvent passEvent = RenderPassEvent.BeforeRenderingShadows; 
    public RenderPassEvent blitPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
    /// <summary>
    /// 临时rt
    /// </summary>
    [NonSerialized]
    private RenderTexture rt1 = null;
    /// <summary>
    /// 临时rt
    /// </summary>
    [NonSerialized]
    private RenderTexture rt2 = null;
    public override void Create()
    {
        _fullScreenViewerPass = new DrawFullScreenViewerPass(material, mesh);
        _blitFullScreenViewerPass = new BlitFullScreenViewerPass(blitMaterial);
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        var width = renderingData.cameraData.camera.pixelWidth;
        var heigh = renderingData.cameraData.camera.pixelHeight;
        if (rt1 == null)
        {
            rt1 = RenderTexture.GetTemporary(width, heigh);
            rt2 = RenderTexture.GetTemporary(width, heigh);
        }

        if (material != null && mesh != null)
        {
            _fullScreenViewerPass.SetRTs(rt1);
            _blitFullScreenViewerPass.SetRTs(rt1, rt2);
            _fullScreenViewerPass.renderPassEvent = passEvent;
            _blitFullScreenViewerPass.renderPassEvent = blitPassEvent;
            renderer.EnqueuePass(_fullScreenViewerPass);
            renderer.EnqueuePass(_blitFullScreenViewerPass);
        }
    }
}

public class DrawFullScreenViewerPass : ScriptableRenderPass
{
    private Material _material;
    private Mesh _mesh;
    private RenderTexture _rt;
    public DrawFullScreenViewerPass(Material material, Mesh mesh)
    {
        _material = material;
        _mesh = mesh;
    }

    public void SetRTs(RenderTexture rt)
    {
        _rt = rt;
        FogDataMgr.Instance.Configure(_mesh, _material);
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        CommandBuffer cmd = CommandBufferPool.Get("DrawFullScreenViewer");
        // 执行draw
        FogDataMgr.Instance.DigView(cmd, _rt);
        context.ExecuteCommandBuffer(cmd);
        cmd.Clear();
        cmd.SetRenderTarget(renderingData.cameraData.renderer.cameraColorTargetHandle);
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
    public override void FrameCleanup(CommandBuffer cmd)
    {
        FogDataMgr.Instance.FrameCleanup(cmd);
    }
}

public class BlitFullScreenViewerPass : ScriptableRenderPass
{
    private readonly Material _material;
    private RenderTexture _rt1;
    private RenderTexture _rt2;
    public BlitFullScreenViewerPass(Material mat)
    {
        _material = mat;
    }
    
    public void SetRTs(RenderTexture rt1, RenderTexture rt2)
    {
        _rt1 = rt1;
        _rt2 = rt2;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        CommandBuffer cmd = CommandBufferPool.Get("BlitFullScreenViewerPass");
        FogDataMgr.Instance.Blit(cmd, _material, ref _rt1, ref _rt2, ref renderingData);
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
}
```
FogDataMgr.cs

```c#
using System.Collections.Generic;
using Unity.Mathematics;
using UnityEngine;
using UnityEngine.Rendering;
using UnityEngine.Rendering.Universal;

public class FogDataMgr
{
    struct MeshProperties
    {
        public float4x4 mat;
        public float status;
    };
    
    private static FogDataMgr _instance = null;

    public static FogDataMgr Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new FogDataMgr();
            }

            return _instance;
        }
    }

    private Dictionary<ulong, int> ViewerGridsStatus = new Dictionary<ulong, int>();
    private ComputeBuffer argsBuffer;
    private ComputeBuffer meshPropertiesBuffer;
    private Material _fogMaterial;
    private Mesh _fogMesh;
    private readonly string PropertyName = "_Properties";
    private uint _width;
    private bool _canDraw = false;
    private uint[] args;

    public void Config(uint width, uint heigh)
    {
        _width = width;
        ViewerGridsStatus.Clear();
        _canDraw = false;
        args = new uint[5] { 0, 0, 0, 0, 0 };
        if (argsBuffer!= null)
            argsBuffer.Dispose();
        argsBuffer = new ComputeBuffer(1, args.Length * sizeof(uint), ComputeBufferType.IndirectArguments);
        argsBuffer.SetData(args);
    }

    public void AddViewerPoint(Vector2Int point, int status)
    {
        ulong key = (ulong)point.x * _width + (ulong)point.y;
        if (!ViewerGridsStatus.ContainsKey(key))
            ViewerGridsStatus.Add(key, status);
    }

    public void Apply()
    {
        CreateBuffer();
        _canDraw = true;
    }

    private void CreateBuffer()
    {
        args[1] = (uint)ViewerGridsStatus.Count;
        if (argsBuffer!= null)
            argsBuffer.Dispose();
        argsBuffer = new ComputeBuffer(1, args.Length * sizeof(uint), ComputeBufferType.IndirectArguments);
        argsBuffer.SetData(args);

        MeshProperties[] properties = new MeshProperties[ViewerGridsStatus.Count];
        int index = 0;
        foreach (var (key, value) in ViewerGridsStatus)
        {
            Vector3 position = new Vector3(key/_width, 0, key%_width);
            Quaternion rotation = Quaternion.Euler(Vector3.zero);
            Vector3 scale = Vector3.one;
     
            properties[index].mat = Matrix4x4.TRS(position, rotation, scale);
            properties[index].status = value;
            index++;
        }
        int size = System.Runtime.InteropServices.Marshal.SizeOf(typeof(MeshProperties));
        if (meshPropertiesBuffer!= null)
            meshPropertiesBuffer.Dispose();
        meshPropertiesBuffer = new ComputeBuffer(ViewerGridsStatus.Count, size);
        meshPropertiesBuffer.SetData(properties);
        _fogMaterial.SetBuffer(PropertyName, meshPropertiesBuffer);
    }
    
    public void DigView(CommandBuffer cmd, RenderTexture rt)
    {
        cmd.SetRenderTarget(rt);
        cmd.ClearRenderTarget(true, true, Color.clear, 1);
        if (_canDraw)
            cmd.DrawMeshInstancedIndirect(_fogMesh, 0, _fogMaterial, -1, argsBuffer);
    }

    public void FrameCleanup(CommandBuffer cmd)
    {
    }

    public void Blit(CommandBuffer cmd, Material _material, ref RenderTexture rt, ref RenderTexture rt2, ref RenderingData renderingData)
    {
        _material.SetTexture("DigRT", rt);
        cmd.Blit(renderingData.cameraData.renderer.cameraColorTargetHandle, rt2);
        cmd.Blit(rt2, renderingData.cameraData.renderer.cameraColorTargetHandle, _material, 0);

    }

    public void Configure(Mesh mesh, Material material)
    {
        _fogMaterial = material;
        _fogMesh = mesh;
        args = new uint[5] { 0, 0, 0, 0, 0 };
        args[0] = _fogMesh.GetIndexCount(0);
        args[2] = _fogMesh.GetIndexStart(0);
        args[3] = _fogMesh.GetBaseVertex(0);
    }
}
```