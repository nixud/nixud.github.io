---
layout: post
title:  "Unity-不值一提的shader入门文章（下）"
date:   2018-08-31
excerpt: "一个shader的入门小文章的下篇"
tag:
- unity
comments: true
---

> 这是作者在学习unity时的一些经验，适合刚接触unity的新手。该系列部分文章会涉及C#语言，并不会对其中某些语句和逻辑给出十分详细的讲解。 本文适用于Unity2017以后版本，但如果你的Unity版本过新或过旧（比如你看到这篇文章的时候已经是2028年了），本文章的内容可能会与实际有所出入。

上一次我们介绍了shader代码的前半部分。这一次我们来讨论下代码的后半部分，也是整个代码的核心。
```
Shader "Hidden/BackgroundChange"
{
	Properties
	{
		_MainTex ("PervBG", 2D) = "white" {}
		_SecTex ("NextBG", 2D) = "white" {}
		_Rule ("Rule", 2D) = "white" {}
		_value("Value",Range(0,1.01)) = 0
	}
	SubShader
	{

		Pass
		{
			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			
			#include "UnityCG.cginc"

			struct appdata
			{
				float4 vertex : POSITION;
				float2 uv : TEXCOORD0;
			};

			struct v2f
			{
				float2 uv : TEXCOORD0;
				float4 vertex : SV_POSITION;
			};

			v2f vert (appdata v)
			{
				v2f o;
				o.vertex = UnityObjectToClipPos(v.vertex);
				o.uv = v.uv;
				return o;
			}
			
			sampler2D _MainTex;
			sampler2D _SecTex;
			sampler2D _Rule;
			float _value;
			
			fixed4 frag (v2f i) : SV_Target
			{
			    fixed4 r = tex2D(_Rule, i.uv);
			    r.a=(r.r+r.g+r.b)/3;
			
			    fixed4 col;
			    if(value > r.a)
			        col = tex2D(_MainTex, i.uv);
			    else col = tex2D(_secTex, i.uv);
			
			    return col;
			}
			ENDCG
		}
	}
}
```
我们来看CGPROGRAM内部的部分。#pragma两句表明了它是一个顶点片段着色器。后一句的#include中包含了一些unity中特有的函数。

再下面是两个结构体。对编程稍有了解的同学恐怕都不会对此感到陌生。这里的float4，是变量类型的一种。平时会用到的变量有float4, half4, fixed4, float, half, fixed以及sampler2D,  samplerCUBE, sampler3D。前六个是不同精度的浮点数（flaot精度最大），float4指的就是一个变量中保存了四个float值。后面是三个纹理变量。两个结构体前一个供vert函数使用，后一个供frag函数使用。

在vert函数中我们只是把物体（object）的坐标转换成了剪裁空间中的坐标，然后交给frag函数处理。

中间声明了四个变量，这四个变量是之前Properties中的。如果不声明的话，Properties中的变量将无法直接在下面的函数中使用。

最下面是frag函数。SV_Target描述了该函数的输出值将直接用于渲染。
```
fixed4 r = tex2D(_Rule, i.uv);
r.a=(r.r+r.g+r.b)/3;
			
fixed4 col;
if(value > r.a)
	col = tex2D(_MainTex, i.uv);
else col = tex2D(_secTex, i.uv);
			
return col;
```
第一句把2d纹理_Rule映射到了fix4变量r上。

下面一句，r.a是r的第四个fixed值，表示alpha，透明度。前三个分别为r.r,r.g,r.b,红色绿色和蓝色。那这里的下一句我们可以理解为让r的透明度等于它的红+蓝+绿除以三。为什么呢？在这里颜色的四个值是从0-1。白色是1,1,1而黑色是0,0,0。对于黑白图像来说，它的红绿蓝三个值是相等的。那么这样rule图像中越接近黑色的部分透明度越低，越接近白色的部分透明度越高。

下面声明了fixed4变量col，它就是我们最后函数要输出的值。

if语句中，若value大于上面算出的r.a，那么输出之前的纹理；若小于等于r.a，那么输出新的纹理。因此要实现最终效果，只要用代码调整value的值即可（从0到1或反过来都可以）。

等等，你这么换，value的改变岂不是必须写在Update里嘛？有没有更优雅的方法呢？

有的。

实际上，只需要在C#代码中传入Time.timeSinceLevelLoad，然后再与shader中的_Time之间作比较就可以了。Time.timeSinceLevelLoad和 _Time都为切换场景后总共过的时间（秒数），而Time.timeSinceLevelLoad传入后不会改变，
_Time会一直变化，这样就可以实现切换即更改了。

那么，这就是这两篇文章的全部内容了。由于作者也在学习中，因此文中也许会有错误以及不准确的内容，如果您正好找到了错误，希望您可以与作者联系并指正他的错误。

ps：有一位聚聚提醒我写ifelse会相当消耗性能，但很遗憾的是，因为作者太菜，暂时没有想到如何用其他方法实现。如果您想到了实现的方法，请务必联系作者，他会及时修改。

> 参考资料：
>
> Shader参考：https://docs.unity3d.com/Manual/SL-Reference.html
>
> 如何写顶点片段着色器：https://docs.unity3d.com/Manual/SL-ShaderPrograms.html
>
> Subshdaer参考：https://docs.unity3d.com/Manual/SL-SubShader.html