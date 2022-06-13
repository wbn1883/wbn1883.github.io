---
title: 'Anti-aliasing grid 抗锯齿网格'
date: 2022-04-18 13:07:58
tags: [Unity,shader]
published: true
hideInList: false
feature: 
isTop: false
mathjax: true
---

最近项目需要用到网格，来自fwidth()的ddx ddy抗锯齿线条
![](https://wbn1883.github.io//post-images/1650258960665.png)
这里贴一下shader:

# $\int(l,v)$ = $\frac{DFG}{4(n \cdot l)(n \cdot v)}$

```swift
Shader "Unlit/AAGrid"
{
    Properties
    {
        _GridColour ("Grid Colour", color) = (1, 1, 1, 1)
        _BaseColour ("Base Colour", color) = (1, 1, 1, 0)
        _GridSpacing ("Grid Spacing", float) = 0.1
        _LineThickness ("Line Thickness", float) = 1
    }
    SubShader
    {
        Tags { "RenderType" = "Transparent" "Queue" = "Transparent" }
        LOD 100

        Blend SrcAlpha OneMinusSrcAlpha
        ZWrite Off
        Cull Off
        Offset -1, -1
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            CBUFFER_START(UnityPerMaterial)
                fixed4 _GridColour;
                fixed4 _BaseColour;
                float _GridSpacing;
                float _LineThickness;
            CBUFFER_END
            v2f vert(appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = (mul(unity_ObjectToWorld, v.vertex).xz + float2(_GridSpacing * 0.5, _GridSpacing * 0.5)) / _GridSpacing;

                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                float2 wrapped = frac(i.uv) - 0.5f;
                float2 range = abs(wrapped);
                float2 pixelRange = range / fwidth(i.uv);
                float lineWeight = saturate(min(pixelRange.x, pixelRange.y) - _LineThickness);

                return lerp(_GridColour, _BaseColour, lineWeight);
            }
            ENDCG
        }
    }
}
```
***

### **关于fwidth解释：**
fwidth(v） = abs(ddx(v)) + abs(ddy(v))

ddx(v) = 该像素点右边的v值 - 该像素点的v值

ddy(v) = 该像素点下面的v值 - 该像素点的v值

ddx(v) ddy(v) 在计算出结果的同时，也会存储该像素的v值，这样它周围的像素才能获取到该点的v值

ddx(float3(1,2,3)) = float3(0,0,0) //因为使用该shader的所有像素 输出的记录值都是 float3(1,2,3)那么差值就为float3(0,0,0)

fwidth(Position) 就是计算出 该像素与相邻两个像素的位置的差值

fwidth(Normal) 就是计算出 该像素与相邻两个像素的法线的差值

fwidth(Color) 就是计算出 该像素与相邻两个像素的Color的差值

fwidth(UV) 就是计算出 该像素与相邻两个像素的UV的差值

fwidth ddx ddy 都是有重载形式的。即你输入什么类型的参数 就是什么返回值:

float fwidth(float v); float ddx(float v); float ddy(float v);

float2 fwidth(float2 v); floa2t ddx(float v); float2 ddy(float v);

float3 fwidth(float3 v); float3 ddx(float v); float ddy(float v);
...


