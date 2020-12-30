---
title: 我喜欢的音乐
date: 2020-06-11 12:31:00
index_img: https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200611124501.jpg
type: 'music'
comments: false
description:
top_img:
mathjax:
katex:
---


<font color=#0c74d6 size=3 face="黑体">Daachun喜欢的音乐</font>

{% meting "34194010" "netease" "playlist" %}


# Aplayer 参数

| 名称              | 默认值                                    | 描述                                        |
|-----------------|----------------------------------------|-------------------------------------------|
| container       | document\.querySelector\('\.aplayer'\) | 播放器容器元素                                   |
| fixed           | false                                  | 开启吸底模式, 详情                                |
| mini            | false                                  | 开启迷你模式, 详情                                |
| autoplay        | false                                  | 音频自动播放                                    |
| theme           | '\#b7daff'                             | 主题色                                       |
| loop            | 'all'                                  | 音频循环播放, 可选值: 'all', 'one', 'none'         |
| order           | 'list'                                 | 音频循环顺序, 可选值: 'list', 'random'             |
| preload         | 'auto'                                 | 预加载，可选值: 'none', 'metadata', 'auto'       |
| volume          | 0\.7                                   | 默认音量，请注意播放器会记忆用户设置，用户手动设置音量后默认音量即失效       |
| audio           | \-                                     | 音频信息, 应该是一个对象或对象数组                        |
| audio\.name     | \-                                     | 音频名称                                      |
| audio\.artist   | \-                                     | 音频艺术家                                     |
| audio\.url      | \-                                     | 音频链接                                      |
| audio\.cover    | \-                                     | 音频封面                                      |
| audio\.lrc      | \-                                     | 详情                                        |
| audio\.theme    | \-                                     | 切换到此音频时的主题色，比上面的 theme 优先级高               |
| audio\.type     | 'auto'                                 | 可选值: 'auto', 'hls', 'normal' 或其他自定义类型, 详情 |
| customAudioType | \-                                     | 自定义类型，详情                                  |
| mutex           | true                                   | 互斥，阻止多个播放器同时播放，当前播放器播放时暂停其他播放器            |
| lrcType         | 0                                      | 详情                                        |
| listFolded      | false                                  | 列表默认折叠                                    |
| listMaxHeight   | \-                                     | 列表最大高度                                    |
| storageName     | 'aplayer\-setting'                     | 存储播放器设置的 localStorage key                 |


# Meting参数

| 选项                      | 默认       | 描述                                     |
|-------------------------|----------|----------------------------------------|
| id\(编号\)                | require  | 歌曲ID /播放列表ID /专辑ID /搜索关键字              |
| server\(平台\)            | require  | 音乐平台：netease，tencent，kugou，xiami，baidu |
| type（类型）                | require  | song，playlist，album，search，artist      |
| auto（支持类种 类）            | options  | 音乐链接，支持：netease，tencent，xiami          |
| fixed（固定模式）             | false    | 启用固定模式，默认false                         |
| mini（迷你模式）              | false    | 启用迷你模式,默认false                         |
| autoplay（自动播放）          | false    | 音频自动播放，默认false                         |
| theme\(主题颜色\)           | \#2980b9 | 默认\#2980b9                             |
| loop（循环）                | all      | 播放器循环播放，值：“all”，one”，“none”            |
| order\(顺序\)             | list     | 播放器播放顺序，值：“list”，“random”              |
| preload\(加载\)           | auto     | 值：“none”，“metadata”，“'auto”            |
| volume（声量）              | 0\.7     | 默认音量，请注意播放器会记住用户设置，用户自己设置音量后默认音量将不起作用  |
| mutex（限制）               | true     | 防止同时播放多个玩家，在该玩家开始播放时暂停其他玩家             |
| lrc\-type（歌词）           | 0        | 歌词显示                                   |
| list\-folded（列表折叠）      | false    | 指示列表是否应该首先折叠                           |
| list\-max\-height（最大高度） | 340px    | 列出最大高度                                 |
| storage\-name（储存名称）     | metingjs | 存储播放器设置的localStorage键                  |
