---
title: '[MR开发]环境配置的深坑'
index_img: 
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708112959.png
tags:
  - MR开发
  - Hololens
  - 混合现实
categories:
  - Unity
description: 环境配置是我们面临的第一个问题
abbrlink: 75960dbe
date: 2020-07-12 17:03:06
---

# 前言

Hololens2开发流程有许多种，官方比较推荐的是通过Unity进行开发、配置和发布，我个人也比较喜欢这个开发路线。

但是，在最开始的**环境配置**这一步，我就遇到了不少困难。

这篇文章主要记录一下配置Hololens2开发环境的**正确步骤**，通过以下顺序来配置环境，能让错误尽量最少。

注意：本篇文章并未涉及Hololens模拟器的安装与配置，相关内容可以参考官方文档。

# 先期准备

## 下载安装包

请先准备以下软件安装包：
- [Windows 10 SDK](https://developer.microsoft.com/zh-cn/windows/downloads/windows-10-sdk/)，10.0.18362.0版本以上。
- [Visual Studio 2019](https://visualstudio.microsoft.com/zh-hans/downloads/)，16.2版本以上。
- [Unity的LTS版本](https://unity3d.com/unity/qa/lts-releases)，推荐2019.4。
- [适用于Unity的MRTK v2开发包](https://github.com/Microsoft/MixedRealityToolkit-Unity/releases)。


## 更新Win10专业版

除此之外，还请将自己的Windows 10系统更新到专业版及以上，具体的更新步骤如下：

1. 找一个可以用在自己机器上的Win10专业版激活码：
   - NXRQM-CXV6P-PBGVJ-293T4-R3KTY
   - MH37W-N47XK-V7XM9-C7227-GCQG9
   - W269N-WFGWX-YVC9B-4J6C9-T83GX
   - 8N67H-M3CY9-QT7C4-2TR7M-TXYCV
   - 2B87N-8KFHP-DKV6R-Y2C8J-PKCKT
2. 打开Win10设置->“更新和安全”->“激活”菜单，点击“更改产品密钥”来尝试一下上面的激活码，如果显示无效可以换一个再试试。
3. 根据弹出的提示进行激活或者升级，这种升级是无损升级，等待重启即可。
4. 安装后具体的激活步骤不再赘述，随便找一个远程kms就可以了。
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712194728.png)

## 打开开发者模式

请打开“更新与安全”->“开发者选项”，选择**开发人员模式**，并打开**设备门户**和**设备发现**。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712195511.png)

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712195538.png)

## 打开虚拟化，开启Hyper-V

请先确定Bios中打开了VT开关！

开始菜单搜索“Hyper-V”，打开“启用或关闭Windows功能”，确保已选择“Hyper-V”。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712195947.png)

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712200006.png)

启用功能后，请重启电脑！


# 安装软件环境

## 1. 安装Win10 SDK

**一定要先安装Win10 SDK，再安装VS！！**

**一定要先安装Win10 SDK，再安装VS！！**

**一定要先安装Win10 SDK，再安装VS！！**

将Win10 SDK安装到C盘或者D盘根目录下Windows Kit文件夹中，**保证安装路径比较简短**。

因为如果按照默认路径安装时，在Unity中进行Build UWP应用会出现“IOException: Win32 IO ......”问题。这就是因为文件在Windows SDK的完整安装路径过长，System.IO.File.Copy无法处理。

![IO错误](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712210758.png)

如果你已经安装过WindosSDK，那么安装路径会无法自定义。需要卸载Visual Studio、所有版本的Windows SDK和模拟器，或者重装系统...

更详细的信息可以看[这里](https://answers.unity.com/questions/1593766/il2cppuwp-strange-error-ioexception-win32-io-retur.html?childToView=1631509#answer-1631509)。

## 2. 安装VS 2019

版本选择社区版就可以了，但是比较重要的是选择哪些开发用的组件。

需要勾选“使用C++的桌面开发”和“通用Windows平台(UWP)开发”这两个组件，.Net可装可不装。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712212232.png)

因为我们还会安装Unity，所以也要勾选上“使用Unity的游戏开发”这个组件，不过就不用勾选右侧的Unity版本了，我们自己装。

## 3. 安装Unity 2019 LTS

安装Unity、注册等步骤不再赘述，需要注意的是，我们要在UnityHub中安装好UWP平台用的Build组件，IL2CPP也推荐安装。
![勾选UWP的发布组件](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200712212333.png)

新建项目后推荐第一时间切换到UWP平台，以防后期素材多了之后切换速度慢。

到这一步，基本环境就安装完毕了。

## 4. 项目中导入MRTK开发包

MRTK可以让我们使用Unity快速开发Hololens应用，并通过其中的Build Window窗口进行打包发布。

打包Appx可以看{% post_link MR开发-appx的打包 这篇文章 %}。

导入MRTK后需要进行项目设置，具体内容可以看[官方教程](https://docs.microsoft.com/zh-cn/windows/mixed-reality/mrlearning-base-ch1)。

# 可能会遇到的坑

Unity打包VS部署Hololens时，报错：找不到WindowsMobile SDK。


这个问题时源自于VS编译的时候是默认UWP相关SDK在C:\Program Files (x86)\Windows Kits中的，而我们为了打包顺利，将SDK装在了短目录。


在使用MRTK工具Build Window打包的时候，也会出现这个报错，这是因为这个工具实际上是帮你用VS打包。


解决方案：

把下载的WindowsMobile SDK从下载的目录

[Windows Kit Root Dir]\10\Extension SDKs\WindowsMobile

拷贝到

C:\Program Files (x86)\Windows Kits\10\Extension SDKs\

个人习惯是10目录下的所有东西都拷贝到对应的10目录下，基本就不会出现这个问题了。

还有些问题可以看看[这篇文章](https://blog.csdn.net/qq_41905133/article/details/88983431)。