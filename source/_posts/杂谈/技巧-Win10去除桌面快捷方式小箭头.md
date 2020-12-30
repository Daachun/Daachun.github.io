---
title: '[技巧]Win10去除桌面快捷方式小箭头'
cover: >-
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200804204956.png
tags:
  - 技巧
  - Win10
  - 资源
categories:
  - 杂谈
description: 推荐使用Dism++，顺便聊聊yoyo鹿鸣
abbrlink: 1fb8f18
date: 2020-08-04 18:48:23
---

# YOYO鹿鸣

最近比较火的[yoyo鹿鸣_Lumi](https://space.bilibili.com/488836173)可算掀开了她的马甲，没错，又是米哈游的阴谋！这个号上的视频质量和播放量都非常高<del>都是lsp</del>。（当然早就有大佬找到mhy注册的yoyo鹿鸣的商标了）

去年有篇米哈游的布料解算的论文发表，这个项目应该算是米哈游的UE4、动捕、布料解算等综合技术的技术展示，质量很高<del>我怀疑跳舞录动作的是妖娆的汉子</del>。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200804185855.png)

[大佬的主页链接](http://geometry.cs.ucl.ac.uk/tuanfeng/)

目前来看是html播放的视频，不过后面应该会加上小游戏一类的交互内容，完整版可以再等等。

目前这个软件跟wallpaper engine、桌面管理软件都有冲突，可以尝鲜一试。

我自己用的是腾讯桌面管理，自带去快捷方式小箭头，因此装机后也没去改这部分的内容，因此，体验人工桌面的时候就会出现**快捷方式小箭头**和**权限管理盾牌**。

![左手剑 右手盾](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200804191928.png)

搜寻资料后在此记录一下比较好的解决方案。

# 为什么不推荐删除IsShortcut

网上找到比较多的方法是**删除注册表IsShortcut键值**的方法来去除快捷方式小箭头，这样会导致很多问题，例如：Win+X菜单打不开，右击计算机图标点管理报错，无法固定到任务栏等。

打开注册表，找到`HKEY_CLASSES_ROOT\Inkfile`，删除`IsShortcut`键值就可以了。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200804192507.png)


容易造成bug，且比较难还原，不推荐。

# 替换成透明Icon

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200804195338.png)

imageres.dll中197这个图标近似于全透明，可以修改注册表的Icon引用，将小箭头或者盾牌图标改成这个197的透明图标。

可以打开注册表位置`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons`在右侧新建**字符串值**，名称为`29`，内容为`C:\WINDOWS\system32\imageres.dll,197`。

盾牌图标想去掉的话，添加一条名为77的键值就可以了，**不建议关闭UAC**。

我们通过批处理来进行设置，新建一个bat文件，并编辑（或者新建txt，编辑好后将后缀设置成bat）。


``` shell
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /d "%systemroot%\system32\imageres.dll,197" /t reg_sz /f
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 77 /d "%systemroot%\system32\imageres.dll,197" /t reg_sz /f
taskkill /f /im explorer.exe
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
pause

```

想恢复的话，将这个位置的注册表键值删除后重启资源管理器就可以了。

``` shell
reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 29 /f
reg delete "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Icons" /v 77 /d "%systemroot%\system32\imageres.dll,197" /t reg_sz /f
taskkill /f /im explorer.exe
del "%userprofile%\AppData\Local\iconcache.db" /f /q
start explorer
pause
```

用**以管理员身份运行**方式运行bat文件就可以了。

# One more thing —— 调整Win10/Win8.1桌面图标默认间距


注册表定位到`HKEY_CURRENT_USER\Control Panel\Desktop\WindowMetrics`，找到`IconSpacing`字符串值。

此处的IconSpacing字符串值代表了桌面图标的水平间距，默认值为：-1125，具体的换算方法为：该值=-15*间距像素值，即-1125代表了桌面图标水平间距为75像素。

如果修改为-1500，就代表着100像素。

找到`IconVerticalSpacing`字符串值。

该字符串值代表着桌面图标的竖直间距，该值和水平间距值要保持一致。

重新打开资源管理器就可以了。


# 推荐软件Dism++

[🔗官网链接](https://www.chuyu.me/zh-Hans/index.html)

Dism++是强大的 Windows 系统优化工具，它把用户使用、调整频率较高的选项集合到一起，让我们可以使用它快速地完成电脑的自定义设置和优化。

直接在“系统优化”项下进行设置就可以了。

![](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200804202441.png)