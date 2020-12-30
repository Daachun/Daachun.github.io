---
title: '[Unity]VRTK的基础配置、实现与UI交互'
index_img: 
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200610210502.png
tags:
  - Unity
  - VR
  - VRTK
  - SteamVR
categories:
  - Unity
keywords:
  - Unity
  - VR
  - VRTK
  - SteamVR
description: 基于Unity2018LTS、SteamVR 1.2.3和VRTK3.3.0的VR开发
abbrlink: 731b117
date: 2020-02-05 18:05:36
top_img:
comments:
top:
toc:
toc_number:
copyright:
mathjax:
katex:
hide:
---

# 简介

VRTK3.3.0是开发VR程序时常用的工具，在SteamVR2.0版本Interaction System出现之前，VRTK提供了通用的VR交互模板，使得我们能够利用VRTK快速开发VR应用。

# 基础配置

相关资源已上传百度网盘。

- [网盘链接](https://pan.baidu.com/s/1JZ7kAO6QI5J-XcDG4jx2kw)
- 提取码：zvx0

## Unity与SDK版本

- 一般来说VRTK对Unity的版本没有太多要求，2017版本以上即可。这里使用的是Unity2018LTS版本。
- VRTK版本使用3.3.0版本，这个版本的VRTK将之前小一百个示例场景整理成7个示例场景：
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/k4Z3wL9Cda8FO2K.png)
- 要兼容VRTK，SteamVR需要老版本1.2.3，这是因为SteamVR2.0重新设计了交互流程，将SteamVR也转换成支持多硬件的插件平台了，VRTK用到的一些API有了修改。

## SDKManager和SDKSetups

场景中创建空物体，命名为“SDKManager”，位置旋转置零，并添加“VRTK_SDKManager”组件。

在SDKManager游戏物体下新建空物体“SDKSetups”，这个空物体下放置各硬件的SDK配置（例如SteamVR的CameraRig）。

使用SteamVR的功能，在SDKSetups下新建空物体“SteamVR”，添加“VRTK_SDKSetup”。将SteamVR的CameraRig拖入到此物体下。组件中“SDK Selection”选择“SteamVR(Standalone:OpenVR)”，点击“Populate Now”按钮即可。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/lZr4A2JEWCuhpQk.png)

同时，在SDKManager中的“Setups”项中点击“Auto Populate”按钮，进行自动配置存在的Setup。
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/Wk4NdGRSZAasgBQ.png)

## VR模拟器

VRTK自带VR模拟器的Setup，可以在Project面板中搜索“VRSimulator_CameraRig”，找到预制体，创建对应的Setup添加到SDKManager中即可。

# 交互基础

配置好SDKManager和SDKSetups后，运行程序还是会报错，这是因为没有在SDKManager中配置左右手柄的引用（Script Aliases）。

## 配置左右手柄

- 这个左右手柄的引用主要用作挂载交互组件以及自定义的和左右手相关的组件。当我们选择不同的SDKSetup运行程序时，这个引用物体会自动成为相应CameraRig的左右手的子物体。
- 在场景中新建空物体“VRTK_Scripts”，并创建“LeftController”和“RightController”子物体，挂载“VRTK_ControllerEvents”。![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/yxEnvet78G5lMOY.png)
- 将这俩空物体分别给SDKManager中“Script Aliases”项赋值即可。

## 配置VRTK中的UI交互

VR环境中，常用的UI交互手段为：

- 注视交互
- 射线交互
- 触碰交互
- 自然交互

先以射线功能为基础，完成VRTK环境下用户与UI交互的需求。

### 添加射线功能

Controller上添加“VRTK_Pointer”组件，用以从手柄发出射线。

其中，“Pointer Activation Settings”项中“Activation Button”项为射线打开的开关按键，默认是触控板按下。

具体按键含义可以对照VRTK的文档以及对应SDK的文档，下图是HTC Vive对应的按键。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/8DrgvZdKFGyJXia.png)

> 注意：这里的VRTK_Pointer组件只是“发出射线并交互”功能的组件，没有“渲染射线”的功能。如果你没有添加“Pointer Renderer”组件的话，一般默认是不渲染出来射线的。
> 
> 在3.3.0版本的VRTK中，你没添加“Pointer Renderer”组件的话，会在运行时自动送你一个“VRTK_Straigh Pinter Renderer”组件，用来以直线渲染射线。
> 
> 还有一种射线的渲染方式是曲线渲染，用在传送上面比较多。可以在“[004 - Locomotion] Teleporting”示例场景查看。

### 与UGUI的交互

Controller上添加“VRTK_UIPointer”组件，使得从手柄发出的射线可以和UGUI进行交互。

场景中新建Canvas，设置模式World Space，缩放更改为0.003，分辨率为3。

添加组件“VRTK_UICanvas”，之后在这个新建的Canvas下创建UGUI即可。

> 记住UIPointer和UICanvas是一对就行。

> 这里有一个坑：如果在添加Canvas之前，添加过VRTK的SDKSwitcher，就会出现无法与UI进行交互的问题。这是因为SDKSwitcher中有一个“Event System”组件，并且是隐藏的。我们应该保证场景中有一个显示的“Event System”组件，这样UGUI才能接收到交互事件。

> ![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/urWq48EdYO3cLbj.png)


### 触碰交互

Controller上添加“VRTK_InteractTouch”，用以开启触碰功能。

Canvas上的“VRTK_UICanvas”组件将“AutoActivateWithinDistance”设置为0.2，这意味着当手柄与UI按钮的距离小于0.2，会自动触发按钮事件。



# 总结

至此，基于VRTK的环境配置和简单UI交互就配置好了，下一篇写一下常用的VRTK组件。