---
title: '[笔记]Shader初心者'
cover: >-
  https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20200610210255.png
tags:
  - Unity
  - Shader
  - 特效
categories:
  - 学习笔记
hide: true
abbrlink: '30064800'
date: 2020-04-10 16:59:50
---

# 常见速查

## SubShader Tags

`Tags{"Queue" = "Geometry" "RenderType" = ""}`

### Queue

- Background：1000，最先进行渲染
- Geometry：2000，常用于不透明对象
- AlphaTest：2450，要么完全透明，要么完全不透明。用于利用贴图来实现边缘透明的效果
- Transparent：3000，半透明对象，从后往前渲染
- Overlay：4000，用于叠加效果，最后渲染的东西放在这里

可以这么写：`Tags{ "Queue" = "Geometry+1" }`

注意：在Unity中，渲染队列小于2500的被认为是不透明的物体，从前往后绘制；其他的队列的物体是从后往前绘制的。我们需要尽可能把物体的队列设置为不透明物体的渲染队列，尽量避免重复绘制。

### RenderType

- Opaque
- Transparent
- TransparentCutout
- Background
- Overlay
- TreeOpaque
- TreeTransparentCutout
- TreeBillboard
- Grass
- GrassBillboard

本质上是一种内部约定，用于区别这个Shader要渲染的对象是属于什么类别的。可以自定义。

作用是可以通过`Camera.SetReplacementShader()`来更改最终的渲染效果：

```cs
camera.SetReplacementShader(EffectShader, Tag);
```

EffectShader是需要渲染的Shader，Tag是用来筛选渲染对象的。如果Tag为空，表示全部物体Shader都替换成EffectShader进行渲染。

### UnityCG.cginc

常用的一些帮助函数

``` shaderlab

float3 WorldSpaceViewDir(float4 v) // 输入一个模型空间中的顶点位置(v.vertex)，返回世界空间中从该点刀摄像机的观察方向
float3 UnityWorldSpaceViewDir


```

## Cg/HLSL

### 类型对应


| Properties      | Cg/HLSL                  |
|-----------------|--------------------------|
| Int/float/Range | float/half/fixed 根据需要的精度 |
| Vector/Color    | float4/half4/fixed4      |
| 2D              | sampler2D                |
| 3D              | sampler3D                |
| CUBE            | samplerCUBE              |

### 常见的几种数据类型

[源](https://connect.unity.com/p/ling-ji-chu-ru-men-unity-shader-liu)

- float/half/fixed（浮点数）
- integer（整型）
- sampler2D（2D纹理）
- samplerCUBE（3D纹理）

**float**

高精度，32位，通常用于世界坐标下的位置，纹理UV，或涉及复杂函数的标量计算（三角函数、幂运算等）。

**half**

中精度，16位，数值范围[-60000, +60000]，通常用于本地坐标下的位置、方向向量、HDR颜色等。

**fixed**

低精度，11位，数值范围为[-2, +2]，通常用于常规的颜色与贴图，以及低精度间的一些运算变量等。

比较常用的一个规则：除了位置和坐标用float以外，其余的全部用half。原因是因为大部分的现代GPU只支持32位与16位，也就是说只支持float和half，不支持fixed。

**interger**

整型类型，通常用于循环与数组的索引。

**sampler2D、sampler3D与samplerCUBE**

纹理，默认情况下在移动平台纹理会自动转换成低精度的纹理类型。如果需要中精度或者高精度的需要单独声明：

- sampler2D_half    (中精度2D纹理)
- sampler2D_float   (高精度2D纹理)
- sampler3D_half    (中精度3D纹理)
- sampler3D_float   (高精度3D纹理)
- samplerCUBE_half (中精度立方体纹理)
- samplerCUBE_float (高精度立方体纹理)


### UV棋盘格

``` cs

fixed checker(float2 uv)
			{
				float2 repeatUV = uv * 10;
				float2 c = floor(repeatUV) / 2;
				float checker = frac(c.x + c.y) * 2;
				return checker;
			}

```

`2 = floor(2.9)`

`3 = ceil(2.1)`

### HLSL内置函数速查

| Name                          | Syntax                             | Description                                                                                                       |
|-------------------------------|------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| abs                           | abs\(x\)                           | Absolute value \(per component\)\.                                                                                |
| acos                          | acos\(x\)                          | Returns the arccosine of each component of x\.                                                                    |
| all                           | all\(x\)                           | Test if all components of x are nonzero\.                                                                         |
| any                           | any\(x\)                           | Test if any component of x is nonzero\.                                                                           |
| asfloat                       | asfloat\(x\)                       | Convert the input type to a float\.                                                                               |
| asin                          | asin\(x\)                          | Returns the arcsine of each component of x\.                                                                      |
| asint                         | asint\(x\)                         | Convert the input type to an integer\.                                                                            |
| asuint                        | asuint\(x\)                        | Convert the input type to an unsigned integer\.                                                                   |
| atan                          | atan\(x\)                          | Returns the arctangent of x\.                                                                                     |
| atan2                         | atan2\(y, x\)                      | Returns the arctangent of of two values \(x,y\)\.                                                                 |
| ceil                          | ceil\(x\)                          | Returns the smallest integer which is greater than or equal to x\.                                                |
| clamp                         | clamp\(x, min, max\)               | Clamps x to the range \[min, max\]\.                                                                              |
| clip                          | clip\(x\)                          | Discards the current pixel, if any component of x is less than zero\.                                             |
| cos                           | cos\(x\)                           | Returns the cosine of x\.                                                                                         |
| cosh                          | cosh\(x\)                          | Returns the hyperbolic cosine of x\.                                                                              |
| cross                         | cross\(x, y\)                      | Returns the cross product of two 3D vectors\.                                                                     |
| D3DCOLORtoUBYTE4              | D3DCOLORtoUBYTE4\(x\)              | Swizzles and scales components of the 4D vector x to compensate for the lack of UBYTE4 support in some hardware\. |
| ddx                           | ddx\(x\)                           | Returns the partial derivative of x with respect to the screen\-space x\-coordinate\.                             |
| ddy                           | ddy\(x\)                           | Returns the partial derivative of x with respect to the screen\-space y\-coordinate\.                             |
| degrees                       | degrees\(x\)                       | Converts x from radians to degrees\.                                                                              |
| determinant                   | determinant\(m\)                   | Returns the determinant of the square matrix m\.                                                                  |
| distance                      | distance\(x, y\)                   | Returns the distance between two points\.                                                                         |
| dot                           | dot\(x, y\)                        | Returns the dot product of two vectors\.                                                                          |
| exp                           | exp\(x\)                           | Returns the base\-e exponent\.                                                                                    |
| exp2                          | exp2\(x\)                          | Base 2 exponent \(per component\)\.                                                                               |
| faceforward                   | faceforward\(n, i, ng\)            | Returns \-n \* sign\(•\(i, ng\)\)\.                                                                               |
| floor                         | floor\(x\)                         | Returns the greatest integer which is less than or equal to x\.                                                   |
| fmod                          | fmod\(x, y\)                       | Returns the floating point remainder of x/y\.                                                                     |
| frac                          | frac\(x\)                          | Returns the fractional part of x\.                                                                                |
| frexp                         | frexp\(x, exp\)                    | Returns the mantissa and exponent of x\.                                                                          |
| fwidth                        | fwidth\(x\)                        | Returns abs\(ddx\(x\)\) \+ abs\(ddy\(x\)\)                                                                        |
| GetRenderTargetSampleCount    | GetRenderTargetSampleCount\(\)     | Returns the number of render\-target samples\.                                                                    |
| GetRenderTargetSamplePosition | GetRenderTargetSamplePosition\(x\) | Returns a sample position \(x,y\) for a given sample index\.                                                      |
| isfinite                      | isfinite\(x\)                      | Returns true if x is finite, false otherwise\.                                                                    |
| isinf                         | isinf\(x\)                         | Returns true if x is \+INF or \-INF, false otherwise\.                                                            |
| isnan                         | isnan\(x\)                         | Returns true if x is NAN or QNAN, false otherwise\.                                                               |
| ldexp                         | ldexp\(x, exp\)                    | Returns x \* 2exp                                                                                                 |
| length                        | length\(v\)                        | Returns the length of the vector v\.                                                                              |
| lerp                          | lerp\(x, y, s\)                    | Returns x \+ s\(y \- x\)\.                                                                                        |
| lit                           | lit\(n • l, n • h, m\)             | Returns a lighting vector \(ambient, diffuse, specular, 1\)                                                       |
| log                           | log\(x\)                           | Returns the base\-e logarithm of x\.                                                                              |
| log10                         | log10\(x\)                         | Returns the base\-10 logarithm of x\.                                                                             |
| log2                          | log2\(x\)                          | Returns the base\-2 logarithm of x\.                                                                              |
| max                           | max\(x, y\)                        | Selects the greater of x and y\.                                                                                  |
| min                           | min\(x, y\)                        | Selects the lesser of x and y\.                                                                                   |
| modf                          | modf\(x, out ip\)                  | Splits the value x into fractional and integer parts\.                                                            |
| mul                           | mul\(x, y\)                        | Performs matrix multiplication using x and y\.                                                                    |
| noise                         | noise\(x\)                         | Generates a random value using the Perlin\-noise algorithm\.                                                      |
| normalize                     | normalize\(x\)                     | Returns a normalized vector\.                                                                                     |
| pow                           | pow\(x, y\)                        | Returns xy\.                                                                                                      |
| radians                       | radians\(x\)                       | Converts x from degrees to radians\.                                                                              |
| reflect                       | reflect\(i, n\)                    | Returns a reflection vector\.                                                                                     |
| refract                       | refract\(i, n, R\)                 | Returns the refraction vector\.                                                                                   |
| round                         | round\(x\)                         | Rounds x to the nearest integer                                                                                   |
| rsqrt                         | rsqrt\(x\)                         | Returns 1 / sqrt\(x\)                                                                                             |
| saturate                      | saturate\(x\)                      | Clamps x to the range \[0, 1\]                                                                                    |
| sign                          | sign\(x\)                          | Computes the sign of x\.                                                                                          |
| sin                           | sin\(x\)                           | Returns the sine of x                                                                                             |
| sincos                        | sincos\(x, out s, out c\)          | Returns the sine and cosine of x\.                                                                                |
| sinh                          | sinh\(x\)                          | Returns the hyperbolic sine of x                                                                                  |
| smoothstep                    | smoothstep\(min, max, x\)          | Returns a smooth Hermite interpolation between 0 and 1\.                                                          |
| sqrt                          | sqrt\(x\)                          | Square root \(per component\)                                                                                     |
| step                          | step\(a, x\)                       | Returns \(x >= a\) ? 1 : 0                                                                                        |
| tan                           | tan\(x\)                           | Returns the tangent of x                                                                                          |
| tanh                          | tanh\(x\)                          | Returns the hyperbolic tangent of x                                                                               |
| tex1D                         | tex1D\(s, t\)                      | 1D texture lookup\.                                                                                               |
| tex1Dbias                     | tex1Dbias\(s, t\)                  | 1D texture lookup with bias\.                                                                                     |
| tex1Dgrad                     | tex1Dgrad\(s, t, ddx, ddy\)        | 1D texture lookup with a gradient\.                                                                               |
| tex1Dlod                      | tex1Dlod\(s, t\)                   | 1D texture lookup with LOD\.                                                                                      |
| tex1Dproj                     | tex1Dproj\(s, t\)                  | 1D texture lookup with projective divide\.                                                                        |
| tex2D                         | tex2D\(s, t\)                      | 2D texture lookup\.                                                                                               |
| tex2Dbias                     | tex2Dbias\(s, t\)                  | 2D texture lookup with bias\.                                                                                     |
| tex2Dgrad                     | tex2Dgrad\(s, t, ddx, ddy\)        | 2D texture lookup with a gradient\.                                                                               |
| tex2Dlod                      | tex2Dlod\(s, t\)                   | 2D texture lookup with LOD\.                                                                                      |
| tex2Dproj                     | tex2Dproj\(s, t\)                  | 2D texture lookup with projective divide\.                                                                        |
| tex3D                         | tex3D\(s, t\)                      | 3D texture lookup\.                                                                                               |
| tex3Dbias                     | tex3Dbias\(s, t\)                  | 3D texture lookup with bias\.                                                                                     |
| tex3Dgrad                     | tex3Dgrad\(s, t, ddx, ddy\)        | 3D texture lookup with a gradient\.                                                                               |
| tex3Dlod                      | tex3Dlod\(s, t\)                   | 3D texture lookup with LOD\.                                                                                      |
| tex3Dproj                     | tex3Dproj\(s, t\)                  | 3D texture lookup with projective divide\.                                                                        |
| texCUBE                       | texCUBE\(s, t\)                    | Cube texture lookup\.                                                                                             |
| texCUBEbias                   | texCUBEbias\(s, t\)                | Cube texture lookup with bias\.                                                                                   |
| texCUBEgrad                   | texCUBEgrad\(s, t, ddx, ddy\)      | Cube texture lookup with a gradient\.                                                                             |
| texCUBElod                    | tex3Dlod\(s, t\)                   | Cube texture lookup with LOD\.                                                                                    |
| texCUBEproj                   | texCUBEproj\(s, t\)                | Cube texture lookup with projective divide\.                                                                      |
| transpose                     | transpose\(m\)                     | Returns the transpose of the matrix m\.                                                                           |
| trunc                         | trunc\(x\)                         | Truncates floating\-point value\(s\) to integer value\(s\)                                                        |

### appdata语义
``` shaderlab
struct appdata
	{
		float4 vertex : POSITION;		//顶点
		float4 tangent : TANGENT;		//切线
		float3 normal : NORMAL;			//法线
		float4 texcoord : TEXCOORD0;	        //UV1
		float4 texcoord1 : TEXCOORD1;	        //UV2
		float4 texcoord2 : TEXCOORD2;	        //UV3
		float4 texcoord3 : TEXCOORD3;	        //UV4
		fixed4 color : COLOR;			//顶点色
	};
```



