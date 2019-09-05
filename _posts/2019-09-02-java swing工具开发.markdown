---
layout:     post
title:      "Java swing 工具开发"
subtitle:   " \"工具开发\""
date:       2019-09-02 11:38:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - Tool;
---

# 开发环境
## idea-2019.2.1
## Java swing

# 创建项目
> 在启动界面`Create New Project`
> ![png](/images/java_swing/java_swing_create_window1.jpg)
> 
> 在创建窗口界面上，注意``1``不要勾上，直接`Next`就可以了
> ![png](/images/java_swing/java_swing_create_window2.jpg)
> 
> 在这里，我们勾选上`Create project from template`，然后`next`
> ![png](/images/java_swing/java_swing_create_window3.jpg)
> 
> 填写上一些必须的名称后，`Finish`
> ![png](/images/java_swing/java_swing_create_window4.jpg)
> 
> ![png](/images/java_swing/java_swing_create_swing1.jpg)
> 创建完成项目工程后，我们现在运行后是一闪而过的，因为我们本没有在`Main.java`文件里写入东西；
> 
> 现在需要创建一个比较合适的swing UI,有了解过java的同学应该知道，java里比较常见的桌面ui开发就是`swing`了。那么现在就开始创建吧!
> 
> 在代码存放的目录下右键，选中`GUI Form`构建GUI的相关文件
> ![png](/images/java_swing/java_swing_create_swing2.jpg)
> 
> ![png](/images/java_swing/java_swing_create_swing3.jpg)
> * 注`1`填写的是GUI配置文件的名称
> * 注`2`填写的是GUI的布局样式
> * 注`3`填写的是GUI绑定的文件
>> GUI的布局样式主要有一下几种
>> * BorderLayout
>> * CardLayout
>> * FlowLayout
>> * FormLayout(JGoodies)
>> * GridBagLayout
>> * GridLayoutManager(Intellij)
> 这里本文使用的是`GridLayoutManager(Intellij)`

# 应用
> 创建完成后，需要怎么样才能把GUI的布局在窗口中展示呢？
> ```
> package com.conerlius.WDTools;
> 
> mport javax.swing.*;
> 
> public class Main {
>
>    public static void main(String[] args) {
>	    // write your code here
>        // 设定为默认展示
>        JFrame.setDefaultLookAndFeelDecorated(true);
>        // 创建frame
>        JFrame frame = new JFrame("Excel2Data");
>        // 设定关闭的方式
>        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
>        // 设定窗口的大小
>        frame.setSize(600,800);
>        // 获取GUI的JPanel，MainView是生成GUI from的绑定class,还需要将mainPanel公开public
>        JPanel panel = new MainView().mainPanel;
>        // 设定为主要的panel
>        frame.setContentPane(panel);
>        // 打包处理
>        frame.pack();
>        // 设置为显示
>        frame.setVisible(true);
>    }
>}
> ```

# 控件
> 其他的空间我就不一一说明了，列举几个比较特殊的使用方式说明一下
## 如何放置控件
> ![gif](/images/java_swing/java_swing_label.gif)
## JPanel
> Java swing的展示面板，一般都是用作大容器使用
>> 前文使用的`JPanel panel = new MainView().mainPanel;`就是一个`JPanel`
## JLabel
> 上图的gif就是一个JLabel的使用方式，现在在这里补充一下JLabel的一些常用字段属性的意义;
> * Field Name : 绑定在java类中的property名称，用于访问和修改的，如果需要`public`，可以去看看绑定的class里将`private`改为`public`
>> ![png](/images/java_swing/java_swing_label2.png)
>> 
>> 第一个框出来的是jlabel的对齐方式
>> 黑色的框是JLabel的大小设置 
## JButton
> 基本属性同JLabel，但是JButton多了一个事件监听；
> 如果需要怼JButton的点击进行监听，操作如下:
>> ![gif](/images/java_swing/java_swing_button.gif)
>
> 选中JButton，右键--> `Create Listener`->`MouseListener`->`mouseClicked`
> 
## JCheckBox
## JList
## JFileChooser