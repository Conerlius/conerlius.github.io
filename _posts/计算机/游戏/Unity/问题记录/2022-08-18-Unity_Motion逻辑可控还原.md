---
layout:     article
title:      "Unity Motion逻辑可控还原"
subtitle:   "unity"
date:       2022-08-18 16:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

因为一些项目的原因，需要提取motion的位移，用于做帧移动和后端验证，所以有了这里的内容！

之前就简单试了一下`motion` 和 `curve` 的关联，结果已经post出来了，最后为了不耗费太多时间，决定还是每帧采样animtor的偏移！

## 采样

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using Sirenix.OdinInspector;
using UnityEditor;
using UnityEngine;

/// <summary>
/// 解压motion数据
/// </summary>
[ExecuteInEditMode]
public class ExtractMotion : MonoBehaviour
{
    [LabelText("动画片段名称：")]
    public string AnimationName;

    [LabelText("带Animator的对象")]
    public Animator Target;

    [LabelText("是否要记录Scale")]
    public bool NeedRecordScale = false;

    public bool EnableStep = false;
    private bool isNext = false;
    #region Clip相关

    private float FrameRate;
    private float Total_Time;
    #endregion
    #if UNITY_EDITOR
    [HideInInspector]
    public Animator ExtractModel;
    private int FrameIndex = 0;
    /// <summary>
    /// 是否正在解压
    /// </summary>
    private bool _isExtract = false;

    [Button("Extract")]
    void Extract()
    {
        if (_isExtract)
        {
            return;
        }

        if (!GetMotionInfo())
        {
            return;
        }

        _isExtract = true;
        // 开始解压
        // 实例化
        ExtractModel = Instantiate(Target);
        var transform1 = ExtractModel.transform;
        transform1.position = Vector3.zero;
        transform1.rotation = Quaternion.identity;
        StartRecorderData();
        ExtractModel.Play(AnimationName);
        ExtractModel.Update(0);
        FrameIndex = 0;
        isNext = false;
        // 添加到EditorApplication.update
        EditorApplication.update += this.ExtractUpdate;
    }

    [Button("Next")]
    void Step()
    {
        // 更新动画时间
        isNext = true;
    }

    bool GetMotionInfo()
    {
        bool hasClip = false;
        if (string.IsNullOrEmpty(AnimationName))
            return false;
        if (Target == null)
            return false;
        var clips = Target.runtimeAnimatorController.animationClips;
        foreach (var clip in clips)
        {
            if (clip.name == AnimationName)
            {
                hasClip = true;
                FrameRate = clip.frameRate;
                Total_Time = clip.length;
                break;
            }
        }
        
        return hasClip;
    }

    void ExtractUpdate()
    {
        if (EnableStep)
        {
            if (!isNext)
                return;
        }

        isNext = false;

        // 获取上一帧的数据并记录
        RecorderData();
        // 更新动画时间
        FrameIndex++;
        float cur_time = FrameIndex / FrameRate;
        if (cur_time <= Total_Time)
        {
            // 为了减少累计的差异！
            var deltaTime = cur_time - (FrameIndex - 1) / FrameRate;
            ExtractModel.Update(deltaTime);
        }
        else
        {
            _isExtract = false;
            EditorApplication.update -= this.ExtractUpdate;
            // 清理
            DestroyImmediate(ExtractModel.gameObject);
            ExtractModel = null;
            StopRecorderData();
            
            // 提示
            EditorUtility.DisplayDialog("提示", "提取完成", "确定");
        }
    }

    #region 数据记录

    private Vector3 PreFrame_Positon;
    private Vector3 PreFrame_Rotation;
    private Vector3 PreFrame_Scale;
    private List<byte> Buffer;
    void StartRecorderData()
    {
        var transform1 = ExtractModel.transform;
        PreFrame_Positon = transform1.position;
        PreFrame_Rotation = transform1.eulerAngles;
        PreFrame_Scale = transform1.localScale;
        // 第一个字节说明是否有scale
        Buffer = new List<byte>();
        Buffer.AddRange(BitConverter.GetBytes(NeedRecordScale));
        Buffer.AddRange(BitConverter.GetBytes((int)FrameRate));
    }

    void RecorderData()
    {
        var transform1 = ExtractModel.transform;
        var Position = transform1.position;
        var Rotation = transform1.eulerAngles;
        var Scale = transform1.localScale;
        // 计算delta
        var deltaPostion = Position - PreFrame_Positon;
        var deltaRotation = Rotation - PreFrame_Rotation;
        var deltaScale = Scale - PreFrame_Scale;
        // 写数据
        FileWirteVector3(deltaPostion);
        FileWirteVector3(deltaRotation);
        if (NeedRecordScale)
        {
            FileWirteVector3(deltaScale);
        }
        // 重新赋值
        PreFrame_Positon = Position;
        PreFrame_Rotation = Rotation;
        PreFrame_Scale = Scale;
    }

    void StopRecorderData()
    {
        var fileInfo = new FileInfo(Path.Combine(Application.dataPath, $"Demo/{AnimationName.Trim()}.ini"));
        if (fileInfo.Exists)
            fileInfo.Delete();
        var fileWriter = fileInfo.Create();
        fileWriter.Write(Buffer.ToArray(), 0, Buffer.Count);
        fileWriter.Flush();
        fileWriter.Close();
    }

    void FileWirteVector3(Vector3 v)
    {
        // 如果有需要，可以考虑太小的偏移直接填充0
        Buffer.AddRange(BitConverter.GetBytes(v.x));
        Buffer.AddRange(BitConverter.GetBytes(v.y));
        Buffer.AddRange(BitConverter.GetBytes(v.z));
    }

    #endregion
#endif
}
```

这里就说一下为什么使用的增量值记录，而不直接使用较origin position的值

- 如果是完全的直线移动的话
  这样直接记录原点的位移是没有问题的！
![](/images/computer/game/unity/problems/animation/motion20.png)

- 如果是有曲线呢？

![](/images/computer/game/unity/problems/animation/motion21.png)

这样得到的就是直接闪烁了！而游戏是逐帧行进的，所以每帧叠加增量变化值就可以了做到！

## 还原的简单处理

因为使用的偏移的增量，所有，在做逻辑运动的时候，只需要把两次运动的时长交给采样，把采样到的数据进行累加，就能得到两次时间间的位移增量！

```c#
public void GetDelta(float time, out Vector3 position, out Vector3 rotation, out Vector3 scale, bool IsLerp = false)
{
    // Datas是前面采样记录的每一帧的位移增量！
    position = Vector3.zero;
    rotation = Vector3.zero;
    scale = Vector3.zero;
    while (true)
    {
        if (curIndex / FrameRate > time)
        {
            break;
        }
        var listIndex = curIndex % Datas.Count;
        // 暂时只算直线
        if (IsLerp)
           position += Datas[listIndex].DeltaPosition * (1 - percent);
        else
        {
            position += Datas[listIndex].DeltaPosition;
        }

        curIndex++;
        percent = 0f;
    }

    if (IsLerp && ((curIndex - 1) / FrameRate < time))
    {
        percent = (time - (curIndex - 1) / FrameRate) / FrameRate;
        var listIndex = curIndex % Datas.Count;
        // 不是线性，有抖动！
        position += Datas[listIndex].DeltaPosition * percent;
    }
}
```

问了一下动作的同事，想要不使用曲线不是不行，是难搞，还有就是直线的话，动作没有那么好了！

上一下gif看看结果：

![](/images/computer/game/unity/problems/animation/motion22.gif)

上面的动图能看到的是有一些抖动！！


![](/images/computer/game/unity/problems/animation/motion23.png)

使用的基本都是 `hermite曲线`,而我代码里使用的是直线插值！所以抖动就出现了！
还有就是float精度的问题；如果是用来处理逻辑，为了保证唯一和确定性，定点数因为精度的问题丢失更夸张。
所以用这种方式同时处理渲染和逻辑基本是不可以行的！

> 如果镜头没有那么近的话，是看不出有抖动的，这个方案基本是可行的！

## 结论

有一定程度上的使用性，但是还不是我们想要的，后面再尝试一下渲染和逻辑分两套处理看看怎么样！