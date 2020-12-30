---
title: '[MR开发]appx的打包发布'
cover: >-
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708112959.png
tags:
  - MR开发
  - Hololens
  - 混合现实
categories:
  - Unity
description: 将项目打包成appx，可以使我们通过旁加载的形式发布到Hololens2实机上。
abbrlink: e944771c
date: 2020-07-08 10:17:46
---

# 为什么打包appx？

如果你的Hololens设备没有在手边，不能通过连接usb线或者本地wifi互联部署项目，打包appx后进行传输是你的最优选择。


appx格式（或appxbundle格式），是应用程序安装包格式，类似于apk，不过它用于分发、安装应用程序到UWP平台（Windows通用平台）。你的Win10机器上的“照片”，“天气”等应用都属于UWP应用。UWP应用可以从微软商店获取与下载。

![UWP应用](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708102811.png)


> 顺带一提，UWP应用在开始菜单中有“动态磁贴”功能。
> 
> ![动态磁贴](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708102828.png)




Hololens2中的所有应用都是UWP应用，因此我们需要打包appx来安装部署我们的应用。

# 发布appx

在使用Unity开发Hololens应用过程中，打包appx有两种方式：通过MRTK工具直接打包，或者通过VS打包。


> 我个人比较推荐通过MRTK打包，这样可以略过一个步骤，并且工作流程不需要太多切换。


## 打包前的环境确认

你需要确认你的机器上满足以下开发Hololens应用的条件：
- Windows10专业版
- 已安装Windows10 SDK 18362+
- Visual Studio 2019
- Unity 2018.4x或Unity 2019
- 对应Unity版本的UWP发布组件
- 打开旁加载模式（如果本机使用模拟器）

![开发人员模式](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708104414.png)

具体需求可以在[MRTK插件文档](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/GettingStartedWithTheMRTK.html)和[微软Hololens文档](https://docs.microsoft.com/zh-cn/windows/mixed-reality/mrtk-getting-started)查看。

## 通过MRTK打包

通过MRTK打包发布是十分方便与开心的！只要设置好Player Settings，打开Build Window就可以了！

> 如何导入MRTK包和如何初始化Hololens的MR项目，在这篇文章中不再赘述。

首先，确定项目设置无误，一定要先将平台转成UWP平台。

![转为UWP平台](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708105701.png)

> Hololens2要选择ARM64

主要确定下图中Publishing Settings里的包名，和下方Capabilities中的内容即可。
![20200708105157](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708105157.png)

之后，打开上方菜单栏'Mixed Reality Toolkit -> Utilities -> Build Window'
![Build Window](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708105258.png)

打开'Appx Build Options'选项卡，直接点击右下角'Build APPX'即可！
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708105903.png)

点击'Open APPX Packages Location'你可以看到多次打包的项目目录。

找到刚刚发布版本的文件夹，打开，里面的appx/appxbundle文件就是我们发布的应用程序包。
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708110141.png)

## 通过VS打包

流程和通过VS部署Hololens2程序一样，设置好项目后直接在Build Settings窗口Build即可，可以看[这篇文档](https://docs.microsoft.com/zh-cn/windows/mixed-reality/exporting-and-building-a-unity-visual-studio-solution)。

不同的一点是，我们不通过VS进行远程部署，而是通过VS打包成appx。

打开发布好的VS项目，将上方平台正确设置（一般是“ARM64”和“Release”——根据Build Settings窗口中的设置），点击'项目 -> 发布 -> 创建应用程序包'，选择“旁加载”，根据提示发布，记得**平台只选择ARM64**。

![通过VS打包](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708111123.png)

# 安装appx

如何将appx部署到机器上？

很简单，连接你的Hololens。如果在同一Wifi下，打开设备控制台页面切换至Apps选项卡，找到Deploy apps，选择好appx的安装包，安装即可。

![Device Portal](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708111720.png)


> 
> 连接Hololens设备需要进行配对，一般来说通过浏览器进入设备控制台，忽略证书错误。
> ![证书错误](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200708111617.png)
>
> [这篇文章](https://blog.csdn.net/shanguuncle/article/details/77806731)讲了如何连接设备管理门户，注意，Hololens2需要打开“开发人员模式”。
> 


> 注意：同包名的项目，版本号是判断新旧的唯一标准，如果你想覆盖之前的版本，一定要保证当前版本号高于之前的。

# 扩展阅读

- [将 Vuforia 引擎与 Unity 一起使用](https://docs.microsoft.com/zh-cn/windows/mixed-reality/vuforia-development-overview)
- [Vuforia 文档：在 Unity 中使用 HoloLens 示例](https://library.vuforia.com/articles/Solution/Working-with-the-HoloLens-sample-in-Unity)
