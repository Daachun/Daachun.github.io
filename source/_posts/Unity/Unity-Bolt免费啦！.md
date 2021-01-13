---
title: '[Unity]Bolt免费啦！'
cover: >-
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723103138.png
tags:
  - Unity
  - 编程
  - 资源
categories:
  - Unity
description: Unity可视化编程工具！
abbrlink: 30993b50
date: 2020-07-23 10:29:17
---

{% note info %}
这篇文章并不会详细的写Bolt的使用方法和节点介绍。
{% endnote %}

# 早上起来天变了

今天早上起床，看了眼手机，QQ上各种技术群里面都在传递着同一个消息——Bolt免费了！打开AssetStore的主页，先映入眼帘的就是上方黄色的条幅：
![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723104307.png)

打开Bolt界面，一个大大的“FREE”拍在了我的脸上。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723104356.png)

对这款可视化编程工具我也早有耳闻，但面对较高的售价之前也没有入手。之前听说开发商变成了Unity，群里就在讨论Unity买下来这个估计是为了把它免费出来，到今天果然免费了。

你问我香不香呢，那肯定是香的，不过对最近买的人可有点不友好。

![不过可以退款](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723104548.png)

# 可视化编程

我们都知道Unreal Engine 4的蓝图（Blueprints）是一个很棒的流（flow）式设计的可视化编程插件，常用UE4的大佬可以只用蓝图做一个完成度非常高的作品出来。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723105203.png)

这种可视化编程的方式，比面对<del>枯燥的</del>代码直接编程要直观的多，对于新手十分友好，也因此可视化编程也会出现在各种幼儿教育中。

# Bolt和Playmaker

什么是“流”式设计的可视化变成插件呢？我把它总结成以下几个特点：
- 通过graph来创建脚本，一个graph就是一个脚本；
- graph中有节点，每个节点代表一项命令；
- 顺序执行，执行完上一个节点后，执行下一个节点（当然，也会有循环）；
- 数据可以从一个节点输入给另一个节点。


在Bolt中，节点就是单元（“Unit”），**实际上就是Unity的各种API命令**，Bolt号称支持所有Unity内置命令，而且可以添加第三方插件的自定义类。可以说这就是Unity版的“蓝图”。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723105925.png)

确实，里面还有逻辑判断的节点...

提到“Unity的蓝图”，我们也不得不提起PlayMaker这个老伙计。这个基于状态机的插件还是比较好理解的，现成的Action库也很多，有一定函数库、代码库积累，效率会比较高。PM的缺点在于代码重用比较困难，可维护性可能会低一些，对数据操作的直接支持比较差，更别说实现设计模式了。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200723111933.png)

两者的对比下来，Bolt更偏向于编写C#脚本的可视化工具，PM可以通过可视化的状态机来快速实现功能。

> 我个人还是认为连连看是写Shader的大杀器，编程的话感觉会更麻烦。
> 当然这是因为我不会写Shader Orz

# 我怎么看？

我认为Bolt是一个不错的程序入门的起点，同时，也是一个很好的原型制作工具。与其说给程序用，不如说给策划岗用更好一些——将设计交还给设计者，这也是Unity作为一个“全能工具”而不是游戏引擎想要做到的。

总的来说，如果你已经过了对Unity的API“从0到1”的理解阶段，直接写一个C#脚本可能是更方便的方法...

<del>不过看DOTS还是要提上日程</del>

# 相关阅读

- [Bolt文档](https://docs.unity3d.com/bolt/1.4/manual/index.html)
- [AssetStore-Bolt](https://assetstore.unity.com/packages/tools/visual-scripting/bolt-163802)
- [UE4蓝图](https://docs.unrealengine.com/zh-CN/Resources/Showcases/BlueprintExamples/index.html)