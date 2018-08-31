---
layout: post
title:  "Unity-不值一提的shader入门文章（上）"
date:   2018-08-30
excerpt: "一个shader的入门小文章"
tag:
- blog
- unity
comments: true
---

> 这是作者在学习unity时的一些经验，适合刚接触unity的新手。该系列部分文章会涉及C#语言，并不会对其中某些语句和逻辑给出十分详细的讲解。 本文适用于Unity2017以后版本，但如果你的Unity版本过新或过旧（比如你看到这篇文章的时候已经是2028年了），本文章的内容可能会与实际有所出入。

这篇文章将要介绍如何使用shader进行日式rpg中的转场。一些日式rpg或galgame中，时常会用到一种挺有趣的背景切换方式。这种背景切换方式需要用到一种叫rule图像的东西，具体长这个样子：
![image](https://s1.ax1x.com/2018/08/29/PXrjoT.png)
*rpg maker xp中的rule图像(在rmxp的汉化版本中被称为渐变图形)*

令人着迷的是，根据指定的rule图像不同，两张背景之间会有不同的变换方式，变换的规则是新的背景从黑色最深的部分开始显示，随时间流逝向黑色较浅的部分延伸，直到完全填满旧的背景。我的语文水平（比较）蹩脚，也许通过文字叙述难于理解，但你可以下载一个rmxp试试（rpg maker后面的系列似乎移除了rule图像，改为直接渐变）。这种更换背景的方式也许有些复古，但目前还是有很多gal在采用这种方式，更何况你也许能通过它对shader有一个最最基本的了解。

~~当然话说回来，如果想做galgame或者rpg，为什么一定要选择unity呢？~~

那么让我们进入正题。shader翻译成中文是着色器，作用可以简单的理解为给屏幕中的物体画上颜色。shader可以直接控制你的GPU对物体进行控制，可以说是非常酷炫了（个鬼啦）。要进一步了解shader，还必须要进行计算机图形学的学习（诶？）。但如果你之前完全没了解过，也不要慌，因为这篇文章本身的难度还是十分简单的（其实就是作者自己也不懂多少），并不会涉及很多图形学的概念，仅仅是供没有了解过的同学了解一下而已（真的哦）。

一个好消息，同时也是一个坏消息，是编写shader有shader专门的语言（shading language，着色语言），而且语法和C#完全不同（这破文章顶上只提了C#啊卧槽）。重新学习的成本是有的，但好消息在于这个东西接触的人比较少，而且由于用途单一，比较简单，没有class的概念。对于shader的核心部分，unity选用了HLSL/Cg语言。

> If you are familliar with C or one of the many languages derived from C, then Cg programs tend to be **short and understandable.** --<The Cg Tutorial>

shader分为三种：

- 表面着色器(surface shaders)
- 顶点片段着色器(vertex and fragment shaders)
- 固定功能着色器(fixed function shaders)

surface shaders需要为你的被着色的物体指定一个光照模型，但正如你所知，2d游戏的背景并不需要光照模型。因此我们使用vertex and fragment shaders来实现。

以下是代码部分：


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
简单来讲，它的核心部分就是CGPROGRAM到ENDCG之间的部分。

最上面一行，是shader的分组和名称。它可以帮助你在选择shader的时候找到它。

第二行是Properties，这一行是显示改shader的可输入项（你可以在Inspector里看到它或者用代码更改）。比如下面这一行：
```
_MainTex ("PervBG", 2D) = "white" {}
```
_MainTex是接收的值在unity中可供识别的名字以及在Subshader花括号内可以声明的变量，它被称为**name。**

"PervBG"是它在Inspector里可以看到的名字，被称作**display name**。这两个名字可以自己来起，但name前面必须加必须加一条美丽（？）的下划线。

再后面的2d是接收的类型，等号后面是默认值。可以接收的类型有：
- Range (min, max) （一个可变的数，有调整范围，默认值"defaulttexture" {}）
- Float （浮点数，默认值格式同上）
- Int   （整数，默认值格式同上）
- Color （颜色，默认值格式(number,number,number,number)）
- Vector （向量，默认值格式同上）
- 2D （2d纹理，默认值格式"defaulttexture" {}）
- Cube （方体纹理，默认值格式同上）
- 3D （3d纹理，默认值格式同上）

下一节我们来讨论下面的Subshader部分。

> 参考资料：
>
> Shader参考：https://docs.unity3d.com/Manual/SL-Reference.html
>
> 如何写顶点片段着色器：https://docs.unity3d.com/Manual/SL-ShaderPrograms.html
>
> Properties参考：https://docs.unity3d.com/Manual/SL-Properties.html