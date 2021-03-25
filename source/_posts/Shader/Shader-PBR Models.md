---
title: "[Shader]PBR Models"
sticky: 
date: 2021-03-26 01:45:06
tags:   
    - Unity
    - Shader
    - 笔记
categories: 学习笔记
keywords:
      - Shader
      - 图形学
      - Unity
description: 简单记录一下PBR模型
top_img: https://Daachun.coding.net/p/blogimg/d/blogimg/git/raw/master/20210326014842.jpg
comments:
hide:
---
## 理解

待补充😳

> Source : https://www.jordanstevenstechart.com/physically-based-rendering

## PBR Test Shader
``` shaderlab
Shader "Daachun/PBR/PBR_01"
{
    Properties
    {
        _Color ("Main Color", Color) = (1,1,1,1)
        _SpecularColor ("Specular Color", Color) = (1,1,1,1)
        _SpecularPower("Specular Power", Range(0,1)) = 1
        _SpecularRange("Specular Gloss",  Range(1,40)) = 0
        _Glossiness("Smoothness",Range(0,1)) = 1
        _Metallic("Metallicness",Range(0,1)) = 0
        _Anisotropic("Anisotropic",  Range(-20,1)) = 0
        _Ior("Ior",  Range(1,4)) = 1.5
        _UnityLightingContribution("Unity Reflection Contribution", Range(0,1)) = 1
        [KeywordEnum(BlinnPhong,Phong,Beckmann,Gaussian,GGX,TrowbridgeReitz,TrowbridgeReitzAnisotropic, Ward)] _NormalDistModel("Normal Distribution Model;", Float) = 0
        [KeywordEnum(AshikhminShirley,AshikhminPremoze,Duer,Neumann,Kelemen,ModifiedKelemen,Cook,Ward,Kurt)]_GeoShadowModel("Geometric Shadow Model;", Float) = 0
        [KeywordEnum(None,Walter,Beckman,GGX,Schlick,SchlickBeckman,SchlickGGX, Implicit)]_SmithGeoShadowModel("Smith Geometric Shadow Model; None if above is Used;", Float) = 0
        [KeywordEnum(Schlick,SchlickIOR, SphericalGaussian)]_FresnelModel("Normal Distribution Model;", Float) = 0
        [Toggle] _ENABLE_NDF ("Normal Distribution Enabled?", Float) = 0
        [Toggle] _ENABLE_G ("Geometric Shadow Enabled?", Float) = 0
        [Toggle] _ENABLE_F ("Fresnel Enabled?", Float) = 0
        [Toggle] _ENABLE_D ("Diffuse Enabled?", Float) = 0
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "Queue"="Geometry"}
        Pass
        {
            Name "FORWORD"
            Tags{
                "LightMode"="ForwardBase"
            }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #define UNITY_PASS_FORWARDBASE
            #include "UnityCG.cginc"
            #include "AutoLight.cginc"
            #include "Lighting.cginc"
            #pragma multi_compile_fwdbase_fullshadows
            #pragma multi_compile _NORMALDISTMODEL_BLINNPHONG _NORMALDISTMODEL_PHONG _NORMALDISTMODEL_BECKMANN _NORMALDISTMODEL_GAUSSIAN _NORMALDISTMODEL_GGX _NORMALDISTMODEL_TROWBRIDGEREITZ _NORMALDISTMODEL_TROWBRIDGEREITZANISOTROPIC _NORMALDISTMODEL_WARD
            #pragma multi_compile _GEOSHADOWMODEL_ASHIKHMINSHIRLEY _GEOSHADOWMODEL_ASHIKHMINPREMOZE _GEOSHADOWMODEL_DUER_GEOSHADOWMODEL_NEUMANN _GEOSHADOWMODEL_KELEMAN _GEOSHADOWMODEL_MODIFIEDKELEMEN _GEOSHADOWMODEL_COOK _GEOSHADOWMODEL_WARD _GEOSHADOWMODEL_KURT 
            #pragma multi_compile _SMITHGEOSHADOWMODEL_NONE _SMITHGEOSHADOWMODEL_WALTER _SMITHGEOSHADOWMODEL_BECKMAN _SMITHGEOSHADOWMODEL_GGX _SMITHGEOSHADOWMODEL_SCHLICK _SMITHGEOSHADOWMODEL_SCHLICKBECKMAN _SMITHGEOSHADOWMODEL_SCHLICKGGX _SMITHGEOSHADOWMODEL_IMPLICIT
            #pragma multi_compile _FRESNELMODEL_SCHLICK _FRESNELMODEL_SCHLICKIOR _FRESNELMODEL_SPHERICALGAUSSIAN
            #pragma multi_compile  _ENABLE_NDF_OFF _ENABLE_NDF_ON
            #pragma multi_compile  _ENABLE_G_OFF _ENABLE_G_ON
            #pragma multi_compile  _ENABLE_F_OFF _ENABLE_F_ON
            #pragma multi_compile  _ENABLE_D_OFF _ENABLE_D_ON
            #pragma target 3.0


            struct VertexInput
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                float2 uv0 : TEXCOORD0;
                float2 uv1 : TEXCOORD1;     // 光照贴图uv
            };

            struct VertexOutput
            {
                float4 pos :SV_POSITION;
                float2 uv0 : TEXCOORD0;
                float2 uv1 : TEXCOORD1;
                // ---
                float3 nDir : TEXCOORD3;
                float3 posWS : TEXCOORD4;
                float3 tDir : TEXCOORD5;
                float3 bDir : TEXCOORD6;
                LIGHTING_COORDS(7,8)        // Unity光照
                UNITY_FOG_COORDS(9)         // Unity雾效
            };

            uniform float4 _Color;
            uniform float4 _SpecularColor;
            uniform float _SpecularPower;
            uniform float _SpecularRange;
            uniform float _Glossiness;
            uniform float _Metallic;
            uniform float _Anisotropic;
            uniform float _Ior;
            uniform float _NormalDistModel;
            uniform float _GeoShadowModel;
            uniform float _FresnelModel;
            uniform float _UnityLightingContribution;


            VertexOutput vert (VertexInput v)
            {
                VertexOutput o = (VertexOutput)0;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv0 = v.uv0;
                o.uv1 = v.uv1;
                o.nDir = UnityObjectToWorldNormal(v.normal);
                o.tDir = normalize(mul(unity_ObjectToWorld, float4(v.tangent.xyz,0.0)).xyz);
                o.bDir = normalize(cross(o.nDir,o.tDir) * v.tangent.w);
                o.posWS = mul(unity_ObjectToWorld, v.vertex);
                UNITY_TRANSFER_FOG(o,o.pos);                    // 雾效
                TRANSFER_VERTEX_TO_FRAGMENT(o);                 // 投影
                return o;
            }

            //------ UnityGI
            // ????
            UnityGI GetUnityGI(float3 lightColor, float3 lDir, float3 nDir, float3 vDir,
            float vrDir, float attenuation, float roughness, float3 worldPos)
            {
                // Unity light Setup ::
                UnityLight light;
                light.color = lightColor;
                light.dir = lDir;
                light.ndotl = max(0.0h, dot(nDir, lDir));
                UnityGIInput d;
                d.light = light;
                d.worldPos = worldPos;
                d.worldViewDir = vDir;
                d.atten = attenuation;
                d.ambient = 0.0h;
                d.boxMax[0] = unity_SpecCube0_BoxMax;
                d.boxMin[0] = unity_SpecCube0_BoxMin;
                d.probePosition[0] = unity_SpecCube0_ProbePosition;
                d.probeHDR[0] = unity_SpecCube0_HDR;
                d.boxMax[1] = unity_SpecCube1_BoxMax;
                d.boxMin[1] = unity_SpecCube1_BoxMin;
                d.probePosition[1] = unity_SpecCube1_ProbePosition;
                d.probeHDR[1] = unity_SpecCube1_HDR;
                Unity_GlossyEnvironmentData ugls_en_data;
                ugls_en_data.roughness = roughness;
                ugls_en_data.reflUVW = vrDir;
                UnityGI gi = UnityGlobalIllumination(d, 1.0h, nDir, ugls_en_data );
                return gi;
            }

            //---------------------------
            //helper functions
            float MixFunction(float i, float j, float x) {
                return  j * x + i * (1.0 - x);
            }
            float2 MixFunction(float2 i, float2 j, float x){
                return  j * x + i * (1.0h - x);
            }
            float3 MixFunction(float3 i, float3 j, float x){
                return  j * x + i * (1.0h - x);
            }
            float MixFunction(float4 i, float4 j, float x){
                return  j * x + i * (1.0h - x);
            }
            float sqr(float x){
                return x*x;
            }
            //------------------------------

            // --- Algorithms Section

            // Blinn-Phong NDF
            // 并不是物理准确的
            float BlinnPhongDNF(float NdotH, float specularpower, float speculargloss)
            {
                float Distribution = pow(NdotH, speculargloss) * specularpower;
                Distribution *= ( 2 + specularpower) / (2 * 3.1415926535);      // 为什么有这么个东西
                return Distribution;
            }
            // Phong NDF
            // 并不是物理准确的，效果模拟比BlinnPhong好一些
            float PhongNDF(float RdotV, float specularpower, float speculargloss)
            {
                float Distribution = pow( RdotV, speculargloss) * specularpower;
                Distribution *= (2 + specularpower) / (2 * 3.1415926535);       // 同上
                return Distribution;
            }
            // Beckmann NDF
            // 加入了roughness
            // Beckmann NDF随着roughness的改变而缓慢变化，直到收紧到一个点(0.000001)。roughness前半段对应粗糙金属，后半段越来越像塑料
            float BeckmannNDF(float roughness, float NdotH)
            {
                float roughnessSqr = roughness * roughness;
                float NdotHSqr = NdotH * NdotH;
                return max(0.000001,(1.0 / (3.1415926535*roughnessSqr*NdotHSqr*NdotHSqr))
                * exp((NdotHSqr-1)/(roughnessSqr*NdotHSqr)));
            }
            // Gaussian NDF
            // 高斯分布
            float GaussianNDF(float roughness, float NdotH)
            {
                float roughnessSqr = roughness * roughness;
                float thetaH = acos(NdotH);
                return exp(-thetaH * thetaH / roughnessSqr);
            }
            // GGX NDF
            float GGXNDF(float roughness, float NdotH)
            {
                float roughnessSqr = roughness* roughness;
                float NdotHSqr = NdotH * NdotH;
                float TanNdotHSqr = (1 - NdotHSqr) / NdotHSqr;
                return (1.0/3.1415926535) * sqr(roughness/(NdotHSqr * (roughnessSqr + TanNdotHSqr)));
            }
            // Trowbridge-Reitz NDF
            // GGX同一篇论文，结果和GGX十分相似。物体边缘的高光比GGX产生的锐利边缘更柔和
            float TrowbridgeReitzNDF(float NdotH, float roughness)
            {
                float roughnessSqr = roughness * roughness;
                float Distribution = NdotH * NdotH * (roughnessSqr - 1.0) + 1.0;
                return roughnessSqr / (3.1415926535 * Distribution * Distribution);
            }
            // Trowbridge-Reitz Anisotropic NDF
            // 各向异性——模仿拉丝金属和其他各向异性表面
            float TrowbridgeReitzAnisotropicNDF(float anisotropic, float NdotH, float HdotX, float HdotY)
            {
                float aspect = sqrt(1.0h - anisotropic * 0.9h);
                float X = max(0.001, sqr(1.0 - _Glossiness) / aspect) * 5;
                float Y = max(0.001, sqr(1.0 - _Glossiness) * aspect) * 5;
                return  1.0 / (3.1415926535 * X*Y * sqr(sqr(HdotX/X) + sqr(HdotY/Y) + NdotH*NdotH));
            }
            // Ward AnisoTropic NDF
            // 高光更柔和，随着光滑度的降低，高光消失得更快
            float WardAnisotropicNDF(float anisotropic, float NdotL, float NdotV, float NdotH, float HdotX, float HdotY)
            {
                float aspect = sqrt(1.0h-anisotropic * 0.9h);
                float X = max(.001, sqr(1.0-_Glossiness)/aspect) * 5;
                float Y = max(.001, sqr(1.0-_Glossiness)*aspect) * 5;
                float exponent = -(sqr(HdotX/X) + sqr(HdotY/Y)) / sqr(NdotH);
                float Distribution = 1.0 / (4.0 * 3.14159265 * X * Y * sqrt(NdotL * NdotV));
                Distribution *= exp(exponent);
                return Distribution;
            }
            // ---Geometric Shadowing Function---
            // Impalicit GSF
            // 隐式模型是集合阴影渲染背后的基本逻辑
            float ImplicitGSF(float NdotL, float NdotV)
            {
                float Gs = (NdotL * NdotV);
                return Gs;
            }
            // Ashikhmin-Shirley GSF
            // 设计用于各向异性NDF的GSF
            // 微表面阴影很微妙
            float AshikhminShirleyGSF(float NdotL, float NdotV, float LdotH)
            {
                float Gs = NdotL * NdotV / (LdotH * max(NdotL, NdotV));
                return Gs;
            }
            // Ashikhmin-Premoze GSF
            // 设计用于各向同性NDF，也很微妙
            float AshikhminPremozeGSF(float NdotL, float NdotV)
            {
                float Gs = NdotL * NdotV / (NdotL + NdotV - NdotL * NdotV);
                return Gs;
            }
            // Duer GSF         ?????
            // 修复了Ward GSF中的高光反射问题。
            // 产生了和Ashihmin-Shirley相似的结果，更适合各向同性BRDF，或者稍微有点各向异性的BRDF
            float DuerGSF (float3 lDir,float3 vDir, float3 nDir,float NdotL, float NdotV){
                float3 LpV = lDir + vDir;
                float Gs = dot(LpV,LpV) * pow(dot(LpV,nDir),-4);
                return Gs;
            }
            // Neumann GSF
            // 另一种针对于各向异性法线描述的GSF。产生了更明显的基于视线与光照方向的几何着色
            float NeumannGSF(float NdotL, float NdotV)
            {
                float Gs = (NdotL * NdotV) / max(NdotL, NdotV);
                return Gs;
            }
            // Kelemen GSF
            // 较为符合能量守恒。几何阴影的比例不是恒定的，而是因查看角度而异。
            // 是Cook-Torrance GSF的近似
            float KelemenGSF(float NdotL, float NdotV, float LdotV, float VdotH)
            {
                float Gs = (NdotL * NdotV) / (VdotH * VdotH);
                return Gs;
            }
            // Modified-Kelemen GSF
            // 修改版的Kelemen函数，通过将原来的做法修改为粗糙度来进行阴影描述
            float ModifiedKelemenGSF(float NdotV, float NdotL, float roughness)
            {
                float c = 0.797884560802865;        // c = sqrt(2 / Pi)
                float k = roughness * roughness * c;
                float gH = NdotV * k + (1 - k);
                return (gH * gH * NdotL);
            }
            // Cook-Torrance GSF
            // 为了解决三种几何衰减的情况而创造出来：
            // 1. 光在没有被干涉的情况下进行反射
            // 2. 反射的光在反射完后被阻挡了
            // 3. 有些光在到达下一个微表面之前被阻挡了
            float CookTorranceGSF(float NdotL, float NdotV, float VdotH, float NdotH)
            {
                float Gs = min(1.0, min(2 * NdotH * NdotV / VdotH, 2 * NdotH * NdotL / VdotH));
                return Gs;
            }
            // Ward GSF
            // 加强版的Implicit GSF。加强NDF适用于突出 视角与平面角度发生改变后各向异性带的表现
            float WardGSF(float NdotL, float NdotV, float VdotH, float NdotH)
            {
                float Gs = pow(NdotL * NdotV, 0.5);
                return Gs;
            }
            // Kurt GSF
            // 各向异性GSF，用于帮助控制基于粗糙度的各向异性表面描述
            // 追求能量守恒，特别是切角线部分
            float KurtGSF(float NdotL, float NdotV, float VdotH, float roughness)
            {
                float Gs = NdotL * NdotV / (VdotH * pow(NdotL * NdotV, roughness));
                return Gs;
            }
            // ---Smith Based GSF---
            // 基于Smith的GSF被广泛认为比其他GSF更精确，并且考虑了正态分布的粗糙度和形状。
            // 这些函数需要处理两个部分
            // GSF(l,v,h) = GSF(L) * GSF(V)

            // Walter et all. GSF:GSF对于BRDF形状的影响非常微小，除了在接近视线切线边缘或者非常粗糙的时候影响很大。不管怎样都要保证能量守恒。
            float WalterEtAllGSF(float NdotL, float NdotV, float alpha)
            {
                float alphaSqr = alpha * alpha;
                float NdotLSqr = NdotL * NdotL;
                float NdotVSqr = NdotV * NdotV;
                float SmithL = 2 / (1 + sqrt(1 + alphaSqr * (1 - NdotLSqr)/ (NdotLSqr)));
                float SmithV = 2 / (1 + sqrt(1 + alphaSqr * (1 - NdotVSqr)/ (NdotVSqr)));
                float Gs = (SmithL * SmithV);
                return Gs;
            }
            // Smith-Beckman GSF
            // 最初是用于和Beckman微表面分布函数进行匹配的，Walter et al提出这也是适用于Phong NDF的GSF
            float BeckmanGSF(float NdotL, float NdotV, float roughness)
            {
                float roughnessSqr = roughness * roughness;
                float NdotLSqr = NdotL * NdotL;
                float NdotVSqr = NdotV * NdotV;
                float calulationL = (NdotL) / (roughnessSqr * sqrt(1 - NdotLSqr));
                float calulationV = (NdotV) / (roughnessSqr * sqrt(1 - NdotVSqr));
                float SmithL = calulationL < 1.6 ? (((3.535 * calulationL) + (2.181 * calulationL * calulationL))/(1 + (2.276 * calulationL) +  (2.577 * calulationL * calulationL))) : 1.0; 
                float SmithV = calulationV < 1.6 ? (((3.535 * calulationV)  + (2.181 * calulationV * calulationV))/(1 + (2.276 * calulationV) + (2.577 * calulationV * calulationV))) : 1.0;
                float Gs =  (SmithL * SmithV);
                return Gs;
            }
            // GGX GSF
            // 对Walter el al 模型的重构
            float GGXGSF(float NdotL, float NdotV, float roughness)
            {
                float roughnessSqr = roughness * roughness;
                float NdotLSqr = NdotL * NdotL;
                float NdotVSqr = NdotV * NdotV;
                float SmithL = (2 * NdotL) / (NdotL + sqrt(roughnessSqr + (1 - roughnessSqr) * NdotLSqr));
                float SmithV = (2 * NdotV) / (NdotV + sqrt(roughnessSqr + (1 - roughnessSqr) * NdotVSqr));
                float Gs = SmithL * SmithV;
                return Gs;
            }
            // Schlick GSF
            // Schlick写的最简单的对Smith GSF的近似模型
            float SchlickGSF(float NdotL, float NdotV, float roughness)
            {
                float roughnessSqr = roughness * roughness;
                float SmithL = (NdotL) / (NdotL * (1 - roughnessSqr) + roughnessSqr);
                float SmithV = (NdotV) / (NdotV * (1 - roughnessSqr) + roughnessSqr);
                float Gs = SmithL * SmithV;
                return Gs;
            }
            // Schlick-Beckman GSF
            // Schlick对Beckman函数的近似。通过对2/Pi的平方根乘以粗糙度，不是预计算的0.797884...
            float SchlickBeckmanGSF(float NdotL, float NdotV, float roughness)
            {
                float roughnessSqr = roughness * roughness;
                float k = roughnessSqr * 0.797884560802865;
                float SmithL = (NdotL) / (NdotL * (1 - k) + k);
                float SmithV = (NdotV) / (NdotV * (1 - k) + k);
                float Gs = SmithL * SmithV;
                return Gs;
            }
            // Schlick-GGX GSF
            // roughness/2
            float SchlickGGXGSF(float NdotL, float NdotV, float roughness)
            {
                float k = roughness/2;
                float SmithL = (NdotL) / (NdotL * (1 - k) + k);
                float SmithV = (NdotV) / (NdotV * (1 - k) + k);
                float Gs = SmithL * SmithV;
                return Gs;
            }

            // --- Fresnel Function ---
            // 菲涅尔函数
            // 考虑漫反射的逆反射，然后考虑BRDF的Fresnel效果。
            // 考虑垂直入射角和锐角方向。用Schlick近似Fresnel：
            // schlick = x + (1 - x) * pow(1 - dotProduct, 5);
            // 近似值：
            // mix(x, 1, pow(1 - dotProduct, 5));
            // 一些GPU中后者计算速度可能会更快。
            //-- 
            // 计算Diffuse：
            // float MixFunction(float i, float j , float x)
            // {
                //     return j * x + i * (1.0 - x);
            // }

            float SchlickFresnel(float i)
            {
                float x = clamp(1.0 - i, 0.0, 1.0);
                float x2 = x * x;
                return x2 * x2 * x;
            }

            float3 FresnelLerp(float3 x, float3 y, float d)
            {
                float t = SchlickFresnel(d);
                return lerp(x, y, t);
            }

            // normal incidence reflection calculation
            float F0(float NdotL, float NdotV, float LdotH, float roughness)
            {
                float FresnelLight = SchlickFresnel(NdotL);
                float FresnelView = SchlickFresnel(NdotV);
                float FresnelDiffuse90 = 0.5 + 2.0 * LdotH * LdotH * roughness;
                return MixFunction(1, FresnelDiffuse90, FresnelLight) * MixFunction(1, FresnelDiffuse90, FresnelView);
            }

            // ---
            // Schlick Fresnel
            // 对于Fresnel方程式的近似
            float3 SchlickFF(float3 SpecularColor, float LdotH)
            {
                return SpecularColor + (1 - SpecularColor) * SchlickFresnel(LdotH);
            }
            // 使用传递的ior折射率，而不是颜色。ior用于表示光穿过表面的速度
            float SchlickIORFF(float ior, float LdotH)
            {
                float f0 = pow(ior - 1, 2) / pow(ior + 1, 2);
                return f0 + (1 - f0) * SchlickFresnel(LdotH);
            }
            // Spherical-Gaussian Fresnel能算出与Schlick Approximation非常相似的结果
            // 差别power值是Spherical Gaussian推导出的
            float SphericalGaussianFF(float LdotH, float SpecularColor)
            {
                float power = ((-5.55473 * LdotH) - 6.98316) * LdotH;
                return SpecularColor + (1 - SpecularColor) * pow(2, power);
            }

            // --- Fragment Section

            fixed4 frag (VertexOutput i) : COLOR
            {
                // -- 准备变量
                float3 nDirWS = normalize(i.nDir);
                float3 lDirWS = normalize(lerp(_WorldSpaceLightPos0.xyz, _WorldSpaceLightPos0.xyz - i.posWS.xyz, _WorldSpaceLightPos0.w));
                float3 lrDirWS = reflect(-lDirWS, nDirWS);
                float3 vDirWS = normalize(_WorldSpaceCameraPos.xyz - i.posWS.xyz);
                float3 vrDirWS = normalize(reflect(-vDirWS, nDirWS));
                float3 hDirWS =  normalize(vDirWS + lDirWS);

                float NdotL = max(0.0, dot(nDirWS, lDirWS));
                float NdotH = max(0.0, dot(nDirWS, hDirWS));
                float NdotV = max(0.0, dot(nDirWS, vDirWS));
                float VdotH = max(0.0, dot(vDirWS, hDirWS));
                float LdotH = max(0.0, dot(lDirWS, hDirWS));
                float LdotV = max(0.0, dot(lDirWS, vDirWS));
                float RdotV = max(0.0, dot(lrDirWS, vDirWS));
                float attenuation = LIGHT_ATTENUATION(i);
                float3 attenColor = attenuation * _LightColor0.rgb;
                // -- 变量准备完毕

                // Unity Scene lighting data
                UnityGI gi =  GetUnityGI(_LightColor0.rgb, lDirWS, nDirWS, vDirWS, vrDirWS, attenuation, 1- _Glossiness, i.posWS.xyz);
                float3 indirectDiffuse = gi.indirect.diffuse.rgb ;
                float3 indirectSpecular = gi.indirect.specular.rgb;

                // remap roughness              JordanStevens经验公式，认为更像物理效果
                float roughness = 1.0 - (_Glossiness * _Glossiness);
                roughness = roughness * roughness;
                // metallic
                float3 diffuseColor = _Color.rgb * (1.0 - _Metallic);
                float f0 = F0(NdotL, NdotV, LdotH, roughness);
                diffuseColor *= f0;
                diffuseColor += indirectDiffuse;

                // Specular
                float3 specColor = lerp(_SpecularColor.rgb, _Color.rgb, _Metallic * 0.5);
                float3 SpecularDistribution = specColor;
                float GeometricShadow = 1;
                float FresnelFunction = specColor;

                // NDF / Specular Distribution
                #ifdef _NORMALDISTMODEL_BLINNPHONG
                    SpecularDistribution *=  BlinnPhongDNF(NdotH, _Glossiness,  max(1,_Glossiness * 40));
                #elif _NORMALDISTMODEL_PHONG
                    SpecularDistribution *=  PhongNDF(RdotV, _Glossiness, max(1,_Glossiness * 40));
                #elif _NORMALDISTMODEL_BECKMANN
                    SpecularDistribution *=  BeckmannNDF(roughness, NdotH);
                #elif _NORMALDISTMODEL_GAUSSIAN
                    SpecularDistribution *=  GaussianNDF(roughness, NdotH);
                #elif _NORMALDISTMODEL_GGX
                    SpecularDistribution *=  GGXNDF(roughness, NdotH);
                #elif _NORMALDISTMODEL_TROWBRIDGEREITZ
                    SpecularDistribution *=  TrowbridgeReitzNDF(NdotH, roughness);
                #elif _NORMALDISTMODEL_TROWBRIDGEREITZANISOTROPIC
                    SpecularDistribution *=  TrowbridgeReitzAnisotropicNDF(_Anisotropic,NdotH, dot(halfDirection, i.tangentDir), dot(halfDirection,  i.bitangentDir));
                #elif _NORMALDISTMODEL_WARD
                    SpecularDistribution *=  WardAnisotropicNDF(_Anisotropic,NdotL, NdotV, NdotH, dot(halfDirection, i.tangentDir), dot(halfDirection,  i.bitangentDir));
                #else
                    SpecularDistribution *=  GGXNDF(roughness, NdotH);
                #endif

                // Geometric Shadowing Function
                #ifdef _SMITHGEOSHADOWMODEL_NONE
                    #ifdef _GEOSHADOWMODEL_ASHIKHMINSHIRLEY
                        GeometricShadow *= AshikhminShirleyGSF (NdotL, NdotV, LdotH);
                    #elif _GEOSHADOWMODEL_ASHIKHMINPREMOZE
                        GeometricShadow *= AshikhminPremozeGSF (NdotL, NdotV);
                    #elif _GEOSHADOWMODEL_DUER
                        GeometricShadow *= DuerGSF (lightDirection, viewDirection, normalDirection, NdotL, NdotV);
                    #elif _GEOSHADOWMODEL_NEUMANN
                        GeometricShadow *= NeumannGSF (NdotL, NdotV);
                    #elif _GEOSHADOWMODEL_KELEMAN
                        GeometricShadow *= KelemenGSF (NdotL, NdotV, LdotH,  VdotH);
                    #elif _GEOSHADOWMODEL_MODIFIEDKELEMEN
                        GeometricShadow *=  ModifiedKelemenGSF (NdotV, NdotL, roughness);
                    #elif _GEOSHADOWMODEL_COOK
                        GeometricShadow *= CookTorranceGSF (NdotL, NdotV, VdotH, NdotH);
                    #elif _GEOSHADOWMODEL_WARD
                        GeometricShadow *= WardGSF (NdotL, NdotV, VdotH, NdotH);
                    #elif _GEOSHADOWMODEL_KURT
                        GeometricShadow *= KurtGSF (NdotL, NdotV, VdotH, roughness);
                    #else
                        GeometricShadow *= ImplicitGSF (NdotL, NdotV);
                    #endif
                    ////SmithModelsBelow
                    ////Gs = F(NdotL) * F(NdotV);
                #elif _SMITHGEOSHADOWMODEL_WALTER
                    GeometricShadow *= WalterEtAllGSF (NdotL, NdotV, roughness);
                #elif _SMITHGEOSHADOWMODEL_BECKMAN
                    GeometricShadow *= BeckmanGSF (NdotL, NdotV, roughness);
                #elif _SMITHGEOSHADOWMODEL_GGX
                    GeometricShadow *= GGXGSF (NdotL, NdotV, roughness);
                #elif _SMITHGEOSHADOWMODEL_SCHLICK
                    GeometricShadow *= SchlickGSF (NdotL, NdotV, roughness);
                #elif _SMITHGEOSHADOWMODEL_SCHLICKBECKMAN
                    GeometricShadow *= SchlickBeckmanGSF (NdotL, NdotV, roughness);
                #elif _SMITHGEOSHADOWMODEL_SCHLICKGGX
                    GeometricShadow *= SchlickGGXGSF (NdotL, NdotV, roughness);
                #elif _SMITHGEOSHADOWMODEL_IMPLICIT
                    GeometricShadow *= ImplicitGSF (NdotL, NdotV);
                #else
                    GeometricShadow *= ImplicitGSF (NdotL, NdotV);
                #endif

                // Fresnel Function
                #ifdef _FRESNELMODEL_SCHLICK
                    FresnelFunction *=  SchlickFF(specColor, LdotH);
                #elif _FRESNELMODEL_SCHLICKIOR
                    FresnelFunction *=  SchlickIORFF(_Ior, LdotH);
                #elif _FRESNELMODEL_SPHERICALGAUSSIAN
                    FresnelFunction *= SphericalGaussianFF(LdotH, specColor);
                #else
                    FresnelFunction *=  SchlickIORFF(_Ior, LdotH);
                #endif

                // test
                #ifdef _ENABLE_NDF_ON
                    return float4(float3(1,1,1)* SpecularDistribution,1);
                #endif
                #ifdef _ENABLE_G_ON 
                    return float4(float3(1,1,1) * GeometricShadow,1) ;
                #endif
                #ifdef _ENABLE_F_ON 
                    return float4(float3(1,1,1)* FresnelFunction,1);
                #endif
                #ifdef _ENABLE_D_ON 
                    return float4(float3(1,1,1)* diffuseColor,1);
                #endif

                // PBR
                // Combining The Algorithms
                float3 specularity = (SpecularDistribution * FresnelFunction * GeometricShadow) / (4 * (  NdotL * NdotV));

                float grazingTerm = saturate(roughness + _Metallic);
                float3 unityIndirectSpecularity =  indirectSpecular * FresnelLerp(specColor,grazingTerm,NdotV) * max(0.15,_Metallic) * (1-roughness*roughness* roughness);

                float3 lightingModel = ((diffuseColor) + specularity + (unityIndirectSpecularity *_UnityLightingContribution));
                lightingModel *= NdotL;
                float4 finalDiffuse = float4(lightingModel * attenColor, 1);
                UNITY_APPLY_FOG(i.fogCoord, finalDiffuse);
                return finalDiffuse;
            }
            ENDCG
        }
    }
    Fallback "Lagacy Shaders/Diffuse"
}

```