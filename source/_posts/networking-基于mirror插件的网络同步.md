---
title: networking-基于mirror插件的网络同步
sticky: 1
date: 2021-03-11 11:48:53
tags:
	- Unity
	- 网络同步
categories:
	- 学习笔记
keywords:
	- Unity
	- 网络同步
description: 使用mirror插件进行网络同步
top_img:
comments:
hide: true
---

| 写法                  | 解释                                                         |
| --------------------- | ------------------------------------------------------------ |
| title                 | 【必需】文章标题                                             |
| date                  | 【必需】文章创建日期                                         |
| updated               | 【可选】文章更新日期                                         |
| tags                  | 【可选】文章标籤                                             |
| categories            | 【可选】文章分类                                             |
| keywords              | 【可选】文章关键字                                           |
| description           | 【可选】文章描述                                             |
| top_img               | 【可选】文章顶部图片                                         |
| cover                 | 【可选】文章缩略图(如果没有设置top_img,文章页顶部将显示缩略图，可设为false/图片地址/留空) |
| sticky                | 【可选】支持文章置顶功能。你可以直接在文章的front-matter区域里添加sticky: 1属性来把这篇文章置顶。数值越大，置顶的优先级越大。 |
| hide                  | 【可选】是否隐藏文章 \# hexo-hide-posts                      |
| comments              | 【可选】显示文章评论模块(默认 true)                          |
| toc                   | 【可选】显示文章TOC(默认为设置中toc的enable配置)             |
| toc_number            | 【可选】显示toc_number(默认为设置中toc的number配置)          |
| copyright             | 【可选】显示文章版权模块(默认为设置中post_copyright的enable配置) |
| copyright_author      | 【可选】文章版权模块的文章作者                               |
| copyright_author_href | 【可选】文章版权模块的文章作者链接                           |
| copyright_url         | 【可选】文章版权模块的文章连结链接                           |
| copyright_info        | 【可选】文章版权模块的版权声明文字                           |
| mathjax               | 【可选】显示mathjax(当设置mathjax的per_page: false时，才需要配置，默认 false) |
| katex                 | 【可选】显示katex(当设置katex的per_page: false时，才需要配置，默认 false) |
| aplayer               | 【可选】在需要的页面加载aplayer的js和css,请参考文章下面的音乐 配置 |
| highlight_shrink      | 【可选】配置代码框是否展开(true/false)(默认为设置中highlight_shrink的配置) |
| aside                 | 【可选】显示侧边栏 (默认 true)                               |



---

# Mirror插件

Mirror插件地址：https://assetstore.unity.com/packages/tools/network/mirror-129321

官网：https://mirror-networking.com/

文档：https://mirror-networking.gitbook.io/docs

视频系列教程：https://www.youtube.com/playlist?list=PLkx8oFug638oBYF5EOwsSS-gOVBXj1dkP



Mirror提供了 **服务端Host/客户端Client** 和 **服务器Server/客户端Client** 两种连接方式。前者服务端既当作服务器也当作服务端，后者的服务器进当作服务器使用。

# 简单的Demo

## 主要组件

### NetworkManager

网络管理器，即开即用。可以将Player的Prefab放到`Player Prefab`项中去，这样玩家连接时会自动生成`spawn`这个prefab到场景中。

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210311115646.png" alt="image-20210311115645984" style="zoom:50%;" />

同时添加`NetworkManagerHUD`组件，可以在屏幕左上角显示简单的控制台，用于开启服务器，连接端口。

### 

# VR中同步需要考虑的事情

