---
title: 相册
type:
comments: false
top_img: 
date: 2020-01-17 21:50:58
description: 相册页面
mathjax:
katex:

# 图库写法
# <div class="gallery-group-main">
# {% galleryGroup '壁纸' '收藏的一些壁纸' '/Gallery/wallpaper' https://i.loli.net/2019/11/10/T7Mu8Aod3egmC4Q.png %}
# {% galleryGroup '漫威' '关于漫威的图片' '/Gallery/marvel' https://i.loli.net/2019/12/25/8t97aVlp4hgyBGu.jpg %}
# {% galleryGroup 'OH MY GIRL' '关于OH MY GIRL的图片' '/Gallery/ohmygirl' https://i.loli.net/2019/12/25/hOqbQ3BIwa6KWpo.jpg %}
# </div>



# 相册内写法：
# {% gallery %}
# markdown 图片格式
# {% endgallery %}
---

<div class="gallery-group-main">


{% galleryGroup '磊子哥的奇妙冒险' '黄山奇遇' '/Gallery/leiadventure' https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200614110342.jpg %}

{% galleryGroup '主题壁纸' '博客主题相关图' '/Gallery/theme' https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200610210615.jpg %}

</div>


{% gallery %}


{% endgallery %}