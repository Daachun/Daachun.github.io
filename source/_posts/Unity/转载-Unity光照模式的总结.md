---
title: '[转载]Unity光照模式的总结'
tags:
  - Unity
  - 转载
  - 渲染
categories:
  - Unity
keywords:
  - Unity
  - 转载
  - 渲染
description: Unity光照模式简单总结，作者MousouCrow是《地下城与勇士》同人游戏DFQ的开发者，蛮厉害的！
cover: >-
  https://connect-cdn-prd-cn.unitychina.cn/20190816/p/images/13c122f9-0ea1-4ef8-8cbf-93c010d5b89f____36.jpg
top: false
copyright: false
abbrlink: 683a0a1e
date: 2020-01-19 22:49:08
top_img:
comments:
toc:
toc_number:
mathjax:
katex:
hide:
---

> 本篇文章转载自[知乎-MusouCrow'BLOG-Unity光照模式的总结](https://zhuanlan.zhihu.com/p/102410358)


# 前言

近日在琢磨Demo应该选择怎样的光照模式，遂做了个试验：对比在同一场合下，各种模式的情况。故以此文记录之（版本为2019.2、平台为Standalone、渲染管线为Builtin）。

# 环境光

环境光(Ambient)严格来说并不是一种光照，它只是单纯的为所有显示元素上色罢了。可以理解为2D游戏便是有个(255, 255, 255)环境光。可于(Window → Rendering → Lighting Settings)下的Environment Lighting进行设置。

![环境光](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-129d1fa63e98971d0a65aabc6657b472_hd.jpg)

环境光是无论如何都需要的，一般用于决定画面的底色。下图便是用了白色的效果：

![白色的环境光](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-ec2ba57002343e76338cd668845aadc7_hd.jpg)

看到上图便能理解我说的「环境光严格来说并不是一种光照」，毕竟连影子都没有，整个画面显得很单薄。但事实上在早期、以及现在一些不依赖光影的游戏是有这种做法的。它们一般会采用类似2D游戏的做法，在素材层面解决各种显示效果问题。对于不依赖光影、强调美术的绝对控制的游戏，使用纯环境光是个方案。

　　顺带一提，在Environment Lighting设置下的Gradient与Skybox模式有着不一样的效果，属于更高级的环境光实现。

# 实时光

　实时光(Realtime)顾名思义，就是每时每刻都在进行的光照。在Light组件的Mode属性设置为Realtime即是。实时光的优缺点很明显，如下：

优点

- 游戏时可随时改变光照的状态，即刻产生反应
- 随取随用，无需烘焙
- 光照效果最好

缺点

- 在正向渲染(Forward Rendering)下，画面同时出现多个光照时，开销较大
- 为了节能，某些设备、设置下，光照的数量有限

实时光一般就是开箱即用到的光照，效果如下：

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-0c28c7abe4b4d0dfb45979b29443a4fa_hd.jpg)

可以看得出，各项消耗指标都比纯环境光要高，而该场景只有三项光照（平行光1个、聚光灯2个）。故一般游戏都不会如此奢侈，会采用各种手段来达到相同的效果。

　　而以上却还不是效果的极致，还差个全局光照(Global Illumination)呢。刚才所见的光照只是「直接的光照」罢了，它只会考虑到照到了谁便处理谁，没有从全局的角度去考虑。在开启全局光照后，除了直接光照之外，还会产生物件之间相互反射的间接光照。效果如下：

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-7d97a673af4351ffa501b37cb7148f26_hd.jpg)

从画面效果来看变得更为深邃了，墙壁与地板都有了反射后的光渍，而各项指标实际上与局部并无不同（疑似）。使用它的前提是要在上文的Lighting Settings下开启Realtime Global Illumination，并且为静态物件做好Static标记。具体实现细节请自行查阅官方文档，在此不表。

　　当然，这并不代表全局光照优于局部光照，就比如有些游戏的画面风格并不喜欢那些全局光照带来的光渍。还是要看想要怎样的美术效果。

# 烘焙光

烘焙光(Baked)可谓实时光的反面：根据光照信息预先渲染成贴图，最后盖到场景上。这个「根据光照信息预先渲染成贴图」的过程，是为烘焙。而烘焙的类型、算法、设置有着多样化的选择，直接影响烘焙的时长、效果、贴图大小与数量。也因烘焙的特性，只适用于静态物件（标记为Static的对象）。优缺点如下：

优点

- 部分渲染元素（取决于烘焙类型）没有实时运行的开销
- 属于全局光照，拥有间接光

缺点

- 光照属性不能运行时修改
- 动态物件不受影响
- 烘焙耗时

烘焙类型主要分三种，效果如下：

- Subtractive: 全烘焙
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-be2760564cb5b2f44d9e01fb8d4e7564_hd.jpg)

- Shadowmask: 烘焙阴影与间接光
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-7649567c407cfaa5d3b8fa917a374b32_hd.jpg)

- Backed Indirect: 只烘焙间接光
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/v2-2dc290b8a1738638e1a0538b2fdf23ed_hd.jpg)

可以看到效果是一个比一个好，但性能却是一个比一个耗。并且可以看出，由于烘焙设置的问题，效果是不如实时光的。而通过设置达到最优的话，烘焙时长则又是个问题了，鱼和熊掌不可兼得啊（砸钱便能我全都要）。

　　对于Subtractive，只需把Light组件的Mode属性设置为**Baked**即可。对于其余两种，实际上是一种实时光+烘焙光的混合方案，则需设置为**Mixed**。由于动态对象不受烘焙光影响的特性，Subtractive下的胶囊仔直接跟纯环境光时一个样了。解决方案有很多种，如采用Mixed方案（静态物件烘焙光、动态物件实时光）、[Light Probes](https://docs.unity3d.com/Manual/LightProbes.html)等。

　　顺带一提，关于Shadowmask，在阴影设置中可调为**Distance Shadowmask**。如此将取决于阴影距离的设置，在阴影距离内的阴影，将采用实时阴影，距离之外的则是烘焙阴影。也算是一种提升品质的方式吧。

　　烘焙光在业界的应用相当广泛，其中Subtractive式烘焙在早期游戏与现代手游可谓家常便饭，妥善使用Light Probes也能达到不俗的效果。

# 后记

以上只是本人粗略的实验与记录，实际上光照的内容浩如烟海，远非本篇所能涵盖。在光照方面本人也只能算是初学者，有所不对还请海涵，并欢迎指教。

---

> 简单在这里做个转载，对于光照系统的总结可以查看[Unity官方文档的光照部分](https://docs.unity3d.com/Manual/LightingOverview.html)。
> 
> Unity官方也有相关文章与演讲，有空搬过来。