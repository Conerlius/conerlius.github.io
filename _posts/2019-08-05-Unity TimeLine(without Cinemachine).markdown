---
layout:     post
title:      "Unity TimeLine(without Cinemachine)"
subtitle:   " \"Unity TimeLine(without Cinemachine)\""
date:       2019-08-05 17:28:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - Unity;
---

# 什么是TimeLine
> Timeline是Unity2017推出的电影序列工具，有助于设计师们更加方便地编辑影片中的动作、声音、事件、视频等等。Timeline无需编写代码，所有操作仅需通过“拖拽”即可完成，从而让设计师们可以更加专注于剧情与故事讲述，加快制作流程。
> 
> 一般情况下创建完成后的Timeline是下图差不多的:
> 
> ![png](/images/2017_1_feature_timeline.png)

# Timeline的基本使用
## Timeline的开启及Track添加
> 由于我使用的是unity 2019.1.4，所以unity里已经集成了`timeline`，如果用的是其他版本的unity的同学可以，在`PackageManager`自行导入`Unity Timeline`
> 
> 导入完成`Timeline`后，通过`Window`->`Sequencing`->`Timeline`来打开`Timeline`的编辑窗口
>> ![png](/images/Timeline_window.png)
> 
> 按照窗口的提示`to start creating a timeline, select a GameObject`，我创建一个空的`Scene`,在场景中添加一个名为`TimelineObj`的`EmptyObject`
> ![png](/images/timeline_selectObj.png)
> 点击创建资源后，unity将自动对`TimelineObj`的Object添加`Playable Director`组件，同时我们在`Timeline`的视窗中看到`Timeline`的初始样子
> ![png](/images/timeline_init.png)
> 
## Track的种类
> 我们可以通过`Timeline`视窗的左上角的`Add+`按钮添加各式各样的Track，`Cinemachine Track`需要导入`Cinemachine`才能有;
>
> 下面我们来大概讲讲各种Track的使用

## Activation Track
> 该Track是用来控制GameObject的显示和隐藏的;
> 
> 通过`Add+`按钮添加一个`Activation Track`, <br>
> 在右侧的视图窗口可以可以控制该组件的生效开始时间和时长，在这里我们把`Active`的开始时间往后移动，
> 绑定GameObject到`Activation Track`
> ![gif](/images/Unity_timeline_Activation_track.gif)
> 启动后（或者直接播放，拖动timeline的时间线），我们就能看到timeline运行工的效果了。
## Animation Track
> 动画片段的控制器，先说一下怎么添加`Animation Track`吧。<br>
> 通过`Add+`按钮添加一个`Animation Track`,<br>
> `Animation Track`同`Activation Track`一样，也需要绑定对象，但是`Animation Track`对绑定的对象有一定的要求，绑定的对象必须要有`Animation Controller`组件。绑定的方式和`Activation Track`一样。<br>
> `Track`添加完成后，我们就需要往`Track`里添加`clip`了。<br>
> ![gif](/images/Unity_timeline_Animation_track.gif)
> 上图里展示了clip的两种添加方式!<br>
> 如果绑定的对象animation Controller也有自己的一套完备的动作体系，那么`Track`的clip就会覆盖对象本身的动作。<br>
> 我们可以通过视图把clip拉伸，从而产生具体的clip的重复执行次数，也可以拆分clip等。<br>
> ![gif](/images/Unity_timeline_Animation_track2.gif)
> 我们看到在clip前后的时候，对象的动作貌似被锁定了，`Track`没有clip的情况下，如果希望重新交给Animation Controller来控制，
> ![png](/images/Unity_timeline_Animation_track3.png)<br>
> 那么就选择相应的clip，在inspector里将其前后的hold改成none就可以了，其他参数的说明，就不在这里一一说了。
## Audio Track
> 添加方式和对象的绑定方式都和`Animation Track`、`Activation Track`一样，但是`Audio Track`的对象是必须有`Audio Source`组件。<br>
> clip的添加也同`Animation Track`，基本的操作也差不多，由于`Audio Track`使用的还是`Audio Source`，所以还是能和`Audio Mixer`一起使用的
## Control Track
> 目前我能发现的就只是用于挂载新物件，可以在剧情的时候在对象上添加一个道具（武器之类的）
> ![png](/images/Unity_timeline_Animation_track4.gif)
## Playable Track
> 这个可能是我们使用最多的了！脚本控制!

***最近再看opengl,这个贴子有空再补***

## Signal Track
> 
## Cinemachine Track
> 使用基本同animation,其他的将在`Cinemachine`中讲述