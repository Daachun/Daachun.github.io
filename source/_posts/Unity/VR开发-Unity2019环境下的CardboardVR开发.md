---
title: VR开发-Unity2019环境下的Cardboard VR开发
index_img: 
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201028090652.png
tags:
  - VR开发
  - Unity
  - 虚拟现实
  - Android
  - Cardboard
categories:
  - Unity
description: >-
  由于UnityXR框架的出现，Unity2019中的AndroidVR开发变得奇怪起来，这篇文章从环境配置出发，用Unity2019和GoogleXR包一步一步实现一个VR全景程序
abbrlink: 3fd471c4
date: 2020-10-26 17:50:15
---

# 简要介绍

Google的Cardboard SDK可以将安卓手机变为VR显示设备，配合Cardboard（那种插手机的VR眼镜），可以给用户一个低成本的VR体验。

然而，Unity在2020版本又一个大刀阔斧的改革：删除了Build-in的VR支持（如下图）。

![Unity删除了Project Settings里面的XR相关](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201026184346.png)

# 环境配置

## 新建项目和导入插件

这个项目使用的Unity版本是：**Unity 2019.4.13f1c1**

下载Unity现在要通过Unity Hub，并添加Android发布套件。

![Unity2019需要安装Android组件](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201026180201.png)

Android开发环境使用自带的OpenJDK。

我们使用Unity的**Package Manager（包管理器）**来导入SDK。使用这个方法，就**一定要安装Git**。

> - [Git](https://git-scm.com/) must be installed and the `git` executable must be on the `PATH` environment variable. See [Unity's package manager git support](https://docs.unity3d.com/Manual/upm-git.html) docs for more details.

通过包管理器的**Add package from git URL**选项，将URL粘贴，并添加**Google Cardboard XR Plugin for Unity**这个自定义包。

`https://github.com/googlevr/cardboard-xr-plugin.git`

![从github下载自定义包](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201026184914.png)

导入后，找到**Samples > Hello Cardboard**，点击Import导入这个案例包。![导入案例包](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201028092244.png)

接下来我们就可以在案例中`Assets/Samples/Google Cardboard/<version>/Hello Cardboard/Assets/Scenes`找到示例场景了。

## 项目详细配置

> 原文档内容：[Cardboard QuickSstart](https://developers.google.com/cardboard/develop/unity/quickstart)

Unity 2019.3版本之前，我们将GoogleVRForUnity的unitypackage导入项目，并打开Player Settings里面的VR支持，选上Cardboard选项就可以进行开发并打包发布了。但现在这样做就会出错。

项目需要进行详细的配置，才能进行打包发布。这应该是Unity对Android平台支持逐渐强大、领域也逐渐细分的原因，新的XR支持应该也是原因之一。

步骤简述：导入包->设置Build Settings->设置Player Settings->修改一些文件

项目设置步骤：

- 打开**File > Build Settings** 选择安卓平台并切换。将示例场景加入到打包场景中。
- 找到**Project Settings** > **Player** > **Resolution and Presentation**，将 **Default Orientation** 设置为 **Landscape Left**。
- 找到**Project Settings** > **Player** > **Other Settings**：

  1. Graphics APIs中仅留`OpenGLES2`

  2. Scripting Backend中选择`IL2CPP`
  3. Target Architectures中选择`ARMv7`和`ARM64`
  4. Internet Access中选择`Require`

- 记得设置项目的包名（Package Name）
- 找到 **Project Settings > XR Plug-in Management**，勾选`Cardboard XR Plugin`。
- 找到 **Project Settings** > **Player** > **Publishing Settings**：勾选`Custom Main Gradle Template`

![这块用的少](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201026191036.png)

找到这个目录，打开这个文件，在**dependencies**添加以下内容：

```groovy
  implementation 'com.android.support:appcompat-v7:28.0.0'
  implementation 'com.android.support:support-v4:28.0.0'
  implementation 'com.google.android.gms:play-services-vision:15.0.2'
  implementation 'com.google.protobuf:protobuf-lite:3.0.0'
```

例如：

```groovy
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
**DEPS**
  implementation 'com.android.support:appcompat-v7:28.0.0'
  implementation 'com.android.support:support-v4:28.0.0'
  implementation 'com.google.android.gms:play-services-vision:15.0.2'
  implementation 'com.google.protobuf:protobuf-lite:3.0.0'
}
```

检查一下Target API Level，如果设置成`API Level 29`或者`Automatic(highest installed)`落在了29上，那还要进行以下设置：勾选`Custom Main Manifest`，在这个文件内添加`application`的tag：
```xml
<application android:requestLegacyExternalStorage="true" ... >
    ...
</application>
```



至此，Android端Cardboard环境配置完毕。

![发布成功](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201026193212.png)

## 一些需要注意的坑

1. API LEVEL 29以下的就不要加android:requestLegacyExternalStorage="true"标签了。

# 实现全景

## 天空盒

找一张全景图，直接拖到天空盒里就行啦！

这种方式做交互比较麻烦，天空盒很远，比较难对应上位置。

## 法线向内的球

我们通过Blender来简单创建一个法线向内的球，并将全景图贴到这个球上。

1. Blender中Shift+A创建一个经纬球，右键设置为平滑着色。

2. Tab进入编辑模式，将球全选，选择网格 > 法相 > 翻转。或者全选面，使用快捷键`Alt+n`选择翻转。你可以在这里查看法线方向，红色是向内。

   ![视图叠加层](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201026193928.png)

3. 导出fbx模型，导入Unity。

## 简单改改Shader

最近有学一些Shader。针对Unity默认的球，我们可以自己写一个Shader设置设置仅正面剔除。

直接新建一个Standard Surface Shader，别的不用改，加一句剔除正面就可以了。

```shaderlab
Cull Front
```

![是正面剔除](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201027204318.png)

用这个Shader创建一个材质球，放到Unity自带的球上就可以了。

![效果好像还是不太对？内部是正常的](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201027224608.png)

# 实现交互

## 凝视交互

创建一个凝视交互很简单，只要从摄像机发射射线，做一个定时器就可以了。

```c#
    private void Update()
    {
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.forward, out hit, MaxDistance))
        {
            // 新物体
            if(pointedObject != hit.transform.gameObject)
            {
                pointedObject = hit.transform.gameObject;
                // 在这里判断射到新物体时的事件
                // OnPointerEnter
            }
            // if(pointedObject.CompareTag("tag")){ ... }
        }
        else
        {
            pointedObject = null;
			// OnPointerExit
        }
```

创建一个Canvas，设置为Camera Space，创建两个Image，一个是十字，另一个是圆形进度条。作为圆形进度条的Image，Image Type设置为Filled，并将定时器的时间映射到Fill Amount上就可以了。

![设置Image Type](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20201028085457.png)

# 总结

总的来说，Unity2019环境下的Cardboard开发还是不难的，麻烦的地方在配置环境，其他照旧。