---
title: "[Shader]-OldSchool基础着色器"
sticky:
date: 2021-03-06 22:57:25
tags:   
    - Unity
    - Shader
    - 笔记
categories: 学习笔记
keywords:
      - Shader
      - 图形学
      - Unity
description: 简单记录一下很基础的渲染方案
cover: https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307024133.jpg
top_image: https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307024133.jpg
comments:
hide:
---

# 一些命名约定...

常用向量：

- `nDir`：法线方向
- `lDir`：光方向
- `vDir`：观察方向
- `rDir/lrDir`：光反射方向
- `hDir`：（halfway）lDir和vDir的中间角（半角）方向
- `vrDir`：观察方向的反射方向...

所在空间：

- `OS`：ObjectSpace 物体空间
- `WS`：WorldSpace 世界空间
- `VS`：ViewSpace 观察空间
- `CS`：homogeneousClipSpace 齐次裁剪空间
- `TS`：TangentSpace 切线空间
- `TXS`：TextureSpace 纹理空间

例：`nDirWS`-世界空间法线方向

# OldSchool Shader

![image-20210306231559805](https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210306232629.png)

默认在Forward Rendering Path的基础上。

## 法线贴图

对法线贴图(normal mapping)进行采样，需要将存储在纹理空间中的法向量转换到模型空间中，最终转化为世界空间的法线。

``` glsl
VertexOutput vert (VertexInput v) { // 顶点Shader
		VertexOutput o = (VertexOutput)0;
		o.nDirWS = UnityObjectToWorldNormal(v.normal);      // 法线方向
		o.tDirWS = normalize(mul(unity_ObjectToWorld, float4(v.tangent.xyz,0.0)).xyz);  // 切线方向
		o.bDirWS = normalize(cross(o.nDirWS, o.tDirWS) * v.tangent.w);  // 副切线方向
		TRANSFER_VERTEX_TO_FRAGMENT(o)                      // 投影相关 
		return o;
}
```

`tDirWS`指的是切线方向，``bDirWS`是副切线方向，需要构建TBN矩阵来进行空间转化。

``` glsl
float4 frag(VertexOutput i) : COLOR {   // 片元Shader 输出rgba
        // 向量准备
		float3 nDirTS = UnpackNormal(tex2D(_NormTex,i.uv0)).rgb;    // 切线空间法线方向
		float3x3 TBN = float3x3(i.tDirWS, i.bDirWS, i.nDirWS);      // TBN矩阵，用于将法线从切线空间转到世界空间
		float3 nDirWS = normalize(mul(nDirTS, TBN));                // 世界空间法线方向
}
```

以上代码中，`VertexOutput`中的`nDirWS`实际上是模型自带法线转为世界空间的方向，通过TBN矩阵将法线贴图中的法线方向转到世界空间。

## 光源

### 漫反射

Lambert很简单：法线方向和光方向点乘，取大于零的数据。

``` glsl
float lambert = max(0.0, ndotl);
```

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307004127.gif" alt="Lambert" style="zoom: 50%;" />

Half-Lambert在上方基础上，进行了Remap，将[0,1]的计算结果置换到了[0.5,1]之间。

``` glsl
float halflambert = max(0.0, ndotl) * 0.5 + 0.5;
```



<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307004155.gif" alt="HalfLambert" style="zoom:50%;" />

可以看到，半兰伯特更淡一点讨好眼睛。当然其中的0.5值不是固定不变的，可以根据需求修改。

固有色和贴图，乘上去就行了。

``` glsl
float4 var_MainTex = tex2D(_MainTex, i.uv0);	// 纹理采样
float3 baseCol = var_MainTex * _MainCol;
// 固有色结果为 baseCol * lambert
```



### 高光

常用Phong光照模型和Blinn-Phong光照模型，都比较简单，只考虑物体对直接光照的反射作用，认为环境光是常量，没有考虑物体之间相互的反射光。

> [Blinnphong到BRDF之间的联系和区别是什么？](http://games-cn.org/forums/topic/gainianyihuoblinnphongdaobrdfzhijiandelianxihequbieshishenme/)
>
> BSDF/BRDF是一种描述光线打到表面后散射现象，从而描述物体的表面性质的物理概念。
> Phong/Blinn-Phong是一种**基于经验**的BSDF模型，之后学习的比如Diffuse(Matte),Glass,Mirror等等是一些，基于物理的BSDF模型。
> Phong/Blinn-Phong模型对物体表面的建模不基于物理，所以可能导致能量不守恒等问题。
>
> - 双向反射分布函数(Bidirectional Reflectance Distribution Function,BRDF)
> - 双向散射分布函数（Bidirectional scattering distribution function, BSDF）

Phong光照模型需要`vDirWS`和`rDirWS`，其中光反射方向的计算，需要用`reflect()`方法。具体原理是光照到物体表面，根据法线方向进行反射。Phong模型计算光反射方向和视角方向的重叠角度，角度越小，高光越强。注意，`lDirWS`实际上是从模型到光源方向，进行反射计算要反向。

``` glsl
float3 lDirWS = _WorldSpaceLightPos0.xyz;
float3 vDirWS = normalize(_WorldSpaceCameraPos.xyz - i.posWS.xyz);  // 世界空间视线方向
float3 lrDirWS = reflect(-lDirWS, nDirWS);
float vdotr = dot(vDirWS, lrDirWS);
...
float phong = pow(max(0.0, vdotr), specPow);
```

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307011058.gif" alt="Phong" style="zoom:50%;" />

Blinn-Phong光照模型计算的是法线方向和半角方向的重合程度，角度越小，高光越强。

``` glsl
float3 hDirWS = normalize(vDirWS + lDirWS);	// 计算半角方向
float3 ndoth = dot(nDirWS,hDirWS);
...
float blinnPhong = pow(max(0.0,ndoth),_SpecularPow);
```

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307011050.gif" alt="BlinnPhong" style="zoom:50%;" />

### Unity中的阴影

暂且不考虑自定义阴影，直接引用Unity给的。主要注意**引用cginc**,结构体内声明格式，以及在顶点片元着色器中的函数。

``` glsl
...
// 追加投影相关文件
#include "AutoLight.cginc"
#include "Lighting.cginc"
...
```

``` glsl
struct VertexOutput {
		...
		float3 bDirWS : TEXCOORD4;  // 世界空间副切线方向
		LIGHTING_COORDS(5,6)        // 投影相关（参数给TEXCOORD编号，后延2位）
         ...
};
```

``` glsl
VertexOutput vert (VertexInput v) { // 顶点Shader
    	...
		TRANSFER_VERTEX_TO_FRAGMENT(o);		// 投影相关 
		...
}
```

``` glsl
float4 frag(VertexOutput i) : COLOR {   // 片元Shader 输出rgba
    	...
    	float shadow = LIGHT_ATTENUATION(i);	// 投影相关 
    	...
}
```

### 组合

直接光照相关组合

``` glsl
float3 dirLighting = (baseCol * lambert + specCol * phong) * _LightColor0 * shadow;
// _LightColor0为光照颜色
// 漫反射颜色和模型乘，高光颜色和高光模型乘
```

## 环境光

### 环境色

一个简单的环境色影响。粗略的将环境光分为上部、侧面和底部的颜色，将颜色叠加上去。取值部分需要注意一点，取模型上部，**从世界空间法线方向的y轴取**。

``` glsl
/// 环境光照
float upMask = max(0.0, nDirWS.y);      // 取得向上部分遮罩
float downMask = max(0.0, -nDirWS.y);   // 取得向下部分遮罩
float sideMask = 1.0-upMask-downMask;     // 侧面部分遮罩
float3 envCol = _EnvUpCol * upMask +
                _EnvSideCol * sideMask +
                _EnvDownCol * downMask;
float3 envDiff = baseCol * envCol * _EnvDiffInt;
```

### 菲涅尔反射

菲涅尔反射（Fresnel reflection）用来描述光在不同折射率的介质之间的行为。这里可以简单想象一个水面，直看水可以看到水面以下，但远看水面反射较强。

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307014505.png" alt="image-20210307014504350" style="zoom:50%;" />

计算和视角方向有关，需要求`vdotn`即`vDirWS`和`nDirWS`的点乘。要突出边缘变化，边缘值应为1，中间为0，所以需要用1减去`vdotn`。`_FresnelPow`控制强度。

``` glsl
float fresnel = pow(max(0.0, 1.0 - vdotn), _FresnelPow);
```

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307015017.gif" alt="fresnel" style="zoom:50%;" />

### Matcap

MatCap是Material Capture，材质捕获。使用特定材质球的贴图，作为当前材质的**视图空间**环境贴图，从而实现具有均匀表面着色的反射材质物体的显示。是一个低计算成本的环境反射的方式。

这相当于将光照结果直接画在图上，需要的时候直接采样取出就可以了。这种Shader有一定的局限性，它可以说是固定光照条件下，从特定方向，特定角度的光照表现结果。优点自然是不用提供光照，不用计算。缺点是不适合相机频繁旋转和角度调节。

注意取样时范围应为一个圆，uv需要针对视角做处理。

``` glsl
...
float3 nDirVS = mul(UNITY_MATRIX_V,float4(nDirWS, 0.0));
float3 vDirWS = normalize(_WorldSpaceCameraPos.xyz - i.posWS.xyz);
float2 matcapUV = nDirVS.rg * 0.5 + 0.5;	// 将[-1,1]的方形取值范围，转为半径0.5的圆形
float nDotv = dot(nDirWS, vDirWS);
float3 matcap = tex2D(_Matcap, matcapUV);	// 正确取样的颜色
...
```

从这张帖图上采样结果：

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307015719.png" alt="matcap" style="zoom:33%;" />

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307015745.gif" alt="matcapgif" style="zoom:50%;" />

### Cubemap

天空盒。处理天空盒资源时要注意exr和ldr的处理，还有将分辨率remap到方形。一般来说需要在photoshop等美术软件中进行处理。

这里面有个概念Mipmap，一个模型身上会有贴图，当我们对这个贴图使用了MipMap技术之后，那么在游戏运行中这个模型的贴图会根据摄像机距离模型的远近而调整不同的不同质量的贴图显示。可以类比LOD。

``` glsl
float3 vrDirWS = reflect(-vDirWS, nDirWS);	// 视线反射方向
float cubemapMip = lerp(_CubemapMip, 1.0, var_SpecTex.a); 	// 贴图越亮越光滑
float3 var_Cubemap = texCUBElod(_Cubemap, float4(vrDirWS, cubemapMip)).rgb;
```

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307020919.gif" alt="cubemap" style="zoom:66%;" />

### 环境光遮蔽

熟悉的AO（Ambient Occlusion），或者叫环境闭塞等等...指物体自身的结构会对环境光造成遮挡，从而影响环境光在不同部位的强度。

这个现象可以抬头看房间两面墙的接缝处，可以看到直角边缘比墙要暗。直接采样AO图乘上去就行了。SSAO一类的东西后面再研究。

<img src="https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210307022200.gif" alt="AO" style="zoom:67%;" />

### 组合

ao最后进行遮蔽就可以。

``` glsl
float3 envLighting = (envDiff + envSpec) * occlusion;
```

---

## 代码备忘

``` shaderlab
Shader "Daachun/L10/OldShcoolPro" {
    Properties {    // 材质面板参数
        [Header(Texture)]
            _MainTex        ("RGB:基础颜色 A:环境遮罩", 2D)         = "white"{}
            _NormTex        ("RGB:法线贴图", 2D)                    = "bump"{}
            _SpecTex        ("RGB:高光颜色 A:高光次幂", 2D)         = "gray"{}
            _EmitTex        ("RGB:环境贴图", 2D)                    = "black"{}
            _Cubemap        ("RGB:环境贴图", Cube)                  = "_Skybox"{}
        [Header(Diffuse)]
            _MainCol        ("基本色",Color)                        = (0.5, 0.5, 0.5, 1.0)
            _EnvDiffInt     ("环境漫反射强度", Range(0,1))          = 0.2
            _EnvUpCol       ("环境色天顶颜色", Color)               = (1.0, 1.0, 1.0, 1.0)
            _EnvSideCol     ("环境色水平颜色", Color)               = (0.5, 0.5, 0.5, 1.0)
            _EnvDownCol     ("环境色地表颜色", Color)               = (0.0, 0.0, 0.0, 1.0)
        [Header(Specular)]
            _SpecPow        ("高光次幂", Range(1, 90))              = 30
            _EnvSpecInt     ("环境镜面反射强度", Range(0, 5))       = 0.2
            _FresnelPow     ("菲涅尔次幂", Range(0, 5))             = 1
            _CubemapMip     ("环境球Mip", Range(1, 7))              = 1
        [Header(Emission)]
            _EmitInt        ("自发光强度", Range(1, 10))            = 1

    }
    SubShader {
        Tags {
            "RenderType"="Opaque"
        }
        Pass {
            Name "FORWARD"
            Tags {
                "LightMode"="ForwardBase"
            }
            
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            // 追加投影相关文件
            #include "AutoLight.cginc"
            #include "Lighting.cginc"
            #pragma multi_compile_fwdbase_fullshadows
            #pragma target 3.0

            // 输入参数
            // Texture
            uniform sampler2D _MainTex;
            uniform sampler2D _NormTex;
            uniform sampler2D _SpecTex;
            uniform sampler2D _EmitTex;
            uniform samplerCUBE _Cubemap;
            // Diffuse
            uniform float3 _MainCol;
            uniform float _EnvDiffInt;
            uniform float3 _EnvUpCol;
            uniform float3 _EnvSideCol;
            uniform float3 _EnvDownCol;
            // Specular
            uniform float _SpecPow;
            uniform float _FresnelPow;
            uniform float _EnvSpecInt;
            uniform float _CubemapMip;
            // Emission
            uniform float _EmitInt;

            struct VertexInput {
                float4 vertex : POSITION;   // 顶点信息
                float3 uv0 : TEXCOORD0;     // UV信息
                float4 normal : NORMAL;     // 法线信息
                float4 tangent : TANGENT;   // 切线信息

            };
            struct VertexOutput {
                float4 pos : SV_POSITION;   // 顶点Shader输出结构
                float2 uv0 : TEXCOORD0;     // UV0
                float4 posWS : TEXCOORD1;   // 世界空间顶点位置
                float3 nDirWS : TEXCOORD2;  // 世界空间法线方向
                float3 tDirWS : TEXCOORD3;  // 世界空间切线方向
                float3 bDirWS : TEXCOORD4;  // 世界空间副切线方向
                LIGHTING_COORDS(5,6)        // 投影相关

            };
            VertexOutput vert (VertexInput v) { // 顶点Shader
                VertexOutput o = (VertexOutput)0;
                o.pos = UnityObjectToClipPos( v.vertex );           // 顶点位置
                o.uv0 = v.uv0;                                      // 传递UV信息
                o.posWS = mul(unity_ObjectToWorld, v.vertex);       // 顶点位置
                o.nDirWS = UnityObjectToWorldNormal(v.normal);      // 法线方向
                o.tDirWS = normalize(mul(unity_ObjectToWorld, float4(v.tangent.xyz,0.0)).xyz);  // 切线方向
                o.bDirWS = normalize(cross(o.nDirWS, o.tDirWS) * v.tangent.w);  // 副切线方向
                TRANSFER_VERTEX_TO_FRAGMENT(o)                      // 投影相关 
                return o;
            }
            float4 frag(VertexOutput i) : COLOR {   // 片元Shader 输出rgba
                // 向量准备
                float3 nDirTS = UnpackNormal(tex2D(_NormTex,i.uv0)).rgb;    // 切线空间法线方向
                float3x3 TBN = float3x3(i.tDirWS, i.bDirWS, i.nDirWS);      // TBN矩阵，用于将法线从切线空间转到世界空间
                float3 nDirWS = normalize(mul(nDirTS, TBN));                // 世界空间法线方向
                float3 vDirWS = normalize(_WorldSpaceCameraPos.xyz - i.posWS.xyz);  // 世界空间视线方向
                float3 vrDirWS = reflect(-vDirWS, nDirWS);                  // 视线反射方向
                float3 lDirWS = _WorldSpaceLightPos0.xyz;                   // 光方向
                float3 lrDirWS = reflect(-lDirWS, nDirWS);                  // 光方向的反射方向
                // 中间量准备
                float ndotl = dot(nDirWS, lDirWS);
                float vdotr = dot(vDirWS, lrDirWS);
                float vdotn = dot(vDirWS, nDirWS);
                // 纹理采样
                float4 var_MainTex = tex2D(_MainTex, i.uv0);
                float4 var_SpecTex = tex2D(_SpecTex, i.uv0);
                float3 var_EmitTex = tex2D(_EmitTex, i.uv0).rgb;
                float cubemapMip = lerp(_CubemapMip, 1.0, var_SpecTex.a);                   // 高光次幂贴图越亮越光滑
                float3 var_Cubemap = texCUBElod(_Cubemap, float4(vrDirWS, cubemapMip)).rgb;
                // 光照模型
                /// 直接光照
                float3 baseCol = var_MainTex.rgb * _MainCol;
                float lambert = max(0.0, ndotl);
                float specCol = var_SpecTex.rgb;
                float specPow = lerp(1, _SpecPow, var_SpecTex.a);
                float phong = pow(max(0.0, vdotr), specPow);
                float shadow = LIGHT_ATTENUATION(i);
                float3 dirLighting = (baseCol * lambert + specCol * phong) * _LightColor0 * shadow;
                /// 环境光照
                float upMask = max(0.0, nDirWS.y);      // 取得向上部分遮罩
                float downMask = max(0.0, -nDirWS.y);   // 取得向下部分遮罩
                float sideMask = 1.0-upMask-downMask;     // 侧面部分遮罩
                float3 envCol = _EnvUpCol * upMask +
                                _EnvSideCol * sideMask +
                                _EnvDownCol * downMask;
                float3 envDiff = baseCol * envCol * _EnvDiffInt;
                float fresnel = pow(max(0.0, 1.0 - vdotn), _FresnelPow);   // 菲涅尔
                float3 envSpec = var_Cubemap * fresnel * _EnvSpecInt;
                float occlusion = var_MainTex.a;
                float3 envLighting = (envDiff + envSpec) * occlusion;
                /// 自发光
                float3 emission = var_EmitTex * _EmitInt;
                // 返回值
                float3 finalRGB = dirLighting + envLighting + emission;
                return fixed4(finalRGB, 1.0);
            }
            ENDCG
        }
    }
    FallBack "Diffuse"
}
```

# 学习资料

- [庄懂TA入门](https://space.bilibili.com/6373917/video)
- [GAMES101:现代计算机图形学入门](https://games-cn.org/intro-graphics/)
- Unity Shader 入门精要

