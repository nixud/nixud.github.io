---
layout: post
title:  "[翻译]如何通过Geometry Shader来实现草海渲染"
date:   2019-9-17
excerpt: "实际上网上已经有一个翻译过的版本了，但那个版本只翻译了一部分，故重新翻译一下"
tag:
comments: true  
---

>对这篇文章，blog.uwa4d.com已经有了一篇翻译，但是翻译的内容实在太少，实际上原文图片相当多，更为关键的是原文在后面随着教程的深入，**更改了前几步的部分代码**，但blog.uwa4d.com的翻译完全没有提到更改的这一部分；同时也略去了原文中开始工程提供的函数，非常多的原文说明，以及很多重要的示意图。
>
>我好想让那个翻译的人失忆，然后看看他能不能看着自己翻译的东西实现这个shader（躺）。顺便一提，翻译文在最后表示不能在安卓和iOS上运行，**这是教程所用的着色器编译目标版本过高所致**。
>
>因此，我打算重新翻译一下原教程（已经写完的示例代码可以在原文中找到）。

##### 原文地址：https://roystan.net/articles/grass-shader.html

这个教程将会描述如何一步一步的写一个unity的草海shader。这个shader将会接收mesh输入，并且使用**几何着色器**在mesh的每一个顶点上生成一片草叶。为了让草更加真实，草叶会有**随机生成的尺寸和方向**，并且被风所影响。为了控制草的密度，我们使用**曲面细分**增加mesh的顶点数。这些草同样会**接收和产生阴影**。

你可以下载这个链接（https://github.com/IronWarrior/UnityGrassGeometryShader/archive/skeleton.zip）中提供的开始工程来开始这个教程。开始工程中包含了一些教程中会用到的函数。

>如果你认为这个教程对你有所帮助，可以考虑一下进入原文地址**向原作者做出捐赠**。

## 所需技能

要完成这个教程，你需要了解一定的unity知识，以及对shader语义和函数的基本了解。

## 准备工作

下载上文所述的开始工程，在unity中打开。打开`Main`场景，然后用任何你喜欢的可以编辑代码的软件打开`Grass`shader。

这个shader文件中包含一个输出白色的shader，以及一些我们会在教程中用到的函数。你也许会注意到，这些函数以及顶点着色器都被放在了`SubShader`之外的`CGINCLUDE`块里。被放在这个块里的代码会被自动放在shader的所有pass当中。我们在之后**需要写多个pass**，因此这个块里的内容是十分有用的。

我们从写一个可以在我们的mesh上生成三角形的**几何着色器**开始。

## 几何着色器

**几何着色器**是渲染管线中的一个可选部分。这一部分运行在顶点着色器之后（如果有曲面细分着色器，那么就是在曲面细分着色器之后）和片元着色器之前。

![image](https://docs.microsoft.com/en-us/windows/desktop/direct3d11/images/d3d11-pipeline-stages.jpg)
>DX11图形渲染管线，在这张图中片元着色器被叫做Pixel Shader。

几何着色器接受单个图元作为输入，但是却可以生成0个，1个或更多图元。我们将使用一个接受顶点输入然后输出一个三角形的几何着色器来表示一根草。

```
// Add inside the CGINCLUDE block.
struct geometryOutput
{
	float4 pos : SV_POSITION;
};

[maxvertexcount(3)]
void geo(triangle float4 IN[3] : SV_POSITION, inout TriangleStream<geometryOutput> triStream)
{
}

…

// Add inside the SubShader Pass, just below the #pragma fragment frag line.
#pragma geometry geo
```

上面的代码声明了一个叫做`geo`的几何着色器，它有两个参数。第一个参数`triangle float4 IN[3]`，表示它会接受一个三角形输入拓扑（有三个点）作为输入。第二个参数`TriangleStream`表示几何着色器会输出一个三角形，它的各个顶点的信息被保存在结构体`geometryOutput`中。

>译者注：这里原作者给出了为什么不可以只接收一个点而不是三角形，感兴趣的话可以去康康，我不翻了（摸肚子

此外，几何着色器声明上方有一行`[maxvertexcount(3)]`，这一行语句告诉GPU这个着色器最多输出3个顶点。为了使`SubShader`使用这个顶点着色器，我们把它声明在Pass中。

这个几何着色器现在还空空如也，添加下面的代码来让它输出一个三角形。

```
geometryOutput o;

o.pos = float4(0.5, 0, 0, 1);
triStream.Append(o);

o.pos = float4(-0.5, 0, 0, 1);
triStream.Append(o);

o.pos = float4(0, 1, 0, 1);
triStream.Append(o);
```

![image2](https://roystan.net/media/tutorials/grass/grass-screen-space.gif)

现在有一个问题：这个三角形被渲染到了屏幕空间中，因此它不会跟随摄像机的移动而移动。为了解决这个问题，我们应当直接让顶点着色器输出裁切空间中的坐标。我们把代码改成这个样子：

```
// Update the return call in the vertex shader.
// 删除下面紧接着的一句，并改为return vertex;
return UnityObjectToClipPos(vertex);
return vertex;

…

// Update each assignment of o.pos in the geometry shader.
o.pos = UnityObjectToClipPos(float4(0.5, 0, 0, 1));

…

o.pos = UnityObjectToClipPos(float4(-0.5, 0, 0, 1));

…

o.pos = UnityObjectToClipPos(float4(0, 1, 0, 1));
```

![image](https://roystan.net/media/tutorials/grass/grass-single-blade.png)

这样一来，我们的三角形就已经生成好了。然而它只有孤零零的一个。这是因为每一个顶点都在固定的位置生成了一个三角形，我们的三角形位置是固定的，它不会跟随每个顶点的位置而改变。

我们应当修改一下我们的着色器，在顶点附近加入不同偏移量，来表示三角形。

```
// Add to the top of the geometry shader.
float3 pos = IN[0];

…

// Update each assignment of o.pos.
o.pos = UnityObjectToClipPos(pos + float3(0.5, 0, 0));

…

o.pos = UnityObjectToClipPos(pos + float3(-0.5, 0, 0));

…

o.pos = UnityObjectToClipPos(pos + float3(0, 1, 0));
```

![image](https://roystan.net/media/tutorials/grass/grass-multiple-blades.png)

>作者在这里解释了一下为什么有的顶点不会生成三角形，我也不翻了，有兴趣的话阔以看看w

那么现在我们的三角形似乎已经很正常了。但我们继续之前，请把场景中的`GrassPlane`的勾去掉变为inactive状态，把`GrassBall`变为active状态。如果我们希望它可以在所有种类的面上都表现的很好，那么在各种不同形状的mesh上测试是十分必要的。

![image](https://roystan.net/media/tutorials/grass/grass-sphere.png)

我们发现，三角形的生成方向依旧固定朝上，而不是朝向球体表面的切线方向。为了解决这个问题，我们将把草叶构建在**切线空间**当中。

## 切线空间

理想状况下，我们会希望在完全不考虑表面的倾角的情况下，为我们的草叶加上随机的宽度、高度、曲率和旋转角度。简而言之，为了解决问题，我们将在原先的空间建立草叶，并且把它转换到一个以表面法线和切线为参考系的空间中。这个坐标空间被叫做**切线空间**。

![image](https://roystan.net/media/tutorials/grass/tangent-space.png)
>在切线空间，XYZ三条轴的方向是由表面的法线决定的。

和任何其他空间一样，我们通过三个向量来定义顶点的切线空间：right、forward和up。我们可以使用这三个向量构筑一个矩阵，来把草叶从切线空间中移动到本地空间。

现在，我们要通过添加输入的内容来得到right和up向量。

```
// Add to the CGINCLUDE block.
struct vertexInput
{
	float4 vertex : POSITION;
	float3 normal : NORMAL;
	float4 tangent : TANGENT;
};

struct vertexOutput
{
	float4 vertex : SV_POSITION;
	float3 normal : NORMAL;
	float4 tangent : TANGENT;
};

…

// Modify the vertex shader.
vertexOutput vert(vertexInput v)
{
	vertexOutput o;
	o.vertex = v.vertex;
	o.normal = v.normal;
	o.tangent = v.tangent;
	return o;
}

…

// Modify the input for the geometry shader. Note that the SV_POSITION semantic is removed.
void geo(triangle vertexOutput IN[3], inout TriangleStream<geometryOutput> triStream)

…

// Modify the existing line declaring pos.
float3 pos = IN[0].vertex;
```

而第三个变量我们可以通过前两个变量的向量积来得到。
>译者吐槽：行吧，来到了线性代数环节了......

```
// Place in the geometry shader, below the line declaring float3 pos.		
float3 vNormal = IN[0].normal;
float4 vTangent = IN[0].tangent;
float3 vBinormal = cross(vNormal, vTangent) * vTangent.w;
```

>这里作者解释了一下为什么向量积结果要乘一个vTangent.w，不翻了（

集齐了所有三个向量之后，我们可以构建一个在切线空间和本地空间之间转换的矩阵。在我们使用`UnityObjectToClipPos`把顶点坐标转换到本地空间前，我们需要把顶点坐标同这个矩阵相乘。

```
// Add below the lines declaring the three vectors.
float3x3 tangentToLocal = float3x3(
	vTangent.x, vBinormal.x, vNormal.x,
	vTangent.y, vBinormal.y, vNormal.y,
	vTangent.z, vBinormal.z, vNormal.z
	);
```

在乘矩阵之前，我们需要把处理顶点输出的各种操作放到一个函数里，来避免一遍又一遍的写一些同样的代码。这通常被叫做DRY原则（dont repeat youself）。

```
// Add to the CGINCLUDE block.
geometryOutput VertexOutput(float3 pos)
{
	geometryOutput o;
	o.pos = UnityObjectToClipPos(pos);
	return o;
}

…

// Remove the following from the geometry shader.
/*删除注释掉的这一部分
geometryOutput o;

o.pos = UnityObjectToClipPos(pos + float3(0.5, 0, 0));
triStream.Append(o);

o.pos = UnityObjectToClipPos(pos + float3(-0.5, 0, 0));
triStream.Append(o);

o.pos = UnityObjectToClipPos(pos + float3(0, 1, 0));
triStream.Append(o);
*/

// ...and replace it with the code below.
triStream.Append(VertexOutput(pos + float3(0.5, 0, 0)));
triStream.Append(VertexOutput(pos + float3(-0.5, 0, 0)));
triStream.Append(VertexOutput(pos + float3(0, 1, 0)));
```

之后为顶点乘上矩阵。

```
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(0.5, 0, 0))));
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(-0.5, 0, 0))));
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(0, 1, 0))));
```

![image](https://roystan.net/media/tutorials/grass/grass-sphere-y.png)

这样看起来就差不多了，但是还并不是完全正确。问题在于之前我们的“上”方向在Y轴上，而在切线空间中，它在Z方向上。因此要把代码改成下面的样子：

```
// Modify the position of the third vertex being emitted.
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(0, 0, 1))));
```

![image](https://roystan.net/media/tutorials/grass/grass-sphere-z.png)

## 草叶的外观

为了使三角形看上去更像一片草，我们需要加一些颜色和变化。

#### 颜色渐变

我们的目标是允许美术人员定义两种颜色————底部一种，顶部一种，然后在中间加入这两种颜色的渐变。这些颜色已经在shader文件中有了定义，分别被叫做`_TopColor`，`_BottomColor`。为了让叶片拥有颜色，我们需要提供给片元着色器UV坐标。

```
// Add to the geometryOutput struct.
float2 uv : TEXCOORD0;

…

// Modify the VertexOutput function signature.
geometryOutput VertexOutput(float3 pos, float2 uv)

…

// Add to VertexOutput, just below the line assigning o.pos.
o.uv = uv;

…

// Modify the existing lines in the geometry shader.
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(0.5, 0, 0)), float2(0, 0)));
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(-0.5, 0, 0)), float2(1, 0)));
triStream.Append(VertexOutput(pos + mul(tangentToLocal, float3(0, 0, 1)), float2(0.5, 1)));
```

我们分别给予三个点恰当的UV值，底部两个一左一右，顶点在中间而且最高。

![image](https://roystan.net/media/tutorials/grass/grass-uv.png)

现在我们可以通过UV坐标加入颜色，然后通过`lerp`函数对颜色进行插值。片元着色器同样要接收`geometryOutput`作为输入，而不是只有一个用于表示位置的`float4`。

```
// Modify the function signature of the fragment shader.
float4 frag (geometryOutput i, fixed facing : VFACE) : SV_Target

…

// Replace the existing return call.
// 删除紧接着的一行，并用下面的一行代替它。
return float4(1, 1, 1, 1);

return lerp(_BottomColor, _TopColor, i.uv.y);
```

![image](https://roystan.net/media/tutorials/grass/grass-gradient.png)

#### 随机的前倾

如果所有的草叶都笔直指向天空，那么看起来将会十分奇怪。这样的草叶也许可以表现人造的假草，但表现现实中的草还远远不够。我们将会写一个可以让草叶在x轴旋转的矩阵，来让它们前倾。

```
// Add as a new property.
_BendRotationRandom("Bend Rotation Random", Range(0, 1)) = 0.2

…

// Add to the CGINCLUDE block.
float _BendRotationRandom;

…

// Add to the geometry shader, below the line declaring facingRotationMatrix.
float3x3 bendRotationMatrix = AngleAxis3x3(rand(pos.zzx) * _BendRotationRandom * UNITY_PI * 0.5, float3(-1, 0, 0));
```

这里再一次使用了叶片位置作为随机种子。时间的不同会创建一个独一无二的值。这里还是用了乘`UNITY_PI`再乘0.5来保证旋转角度在0-90以内。

同样，把这个矩阵也乘进去。

```
// Modify the existing line.
float3x3 transformationMatrix = mul(mul(tangentToLocal, facingRotationMatrix), bendRotationMatrix);
```

#### 叶片宽度与高度

叶片的尺寸目前有一单位宽，一单位高。我们将会加入一个可以控制它们的属性，同时加入一些随机的值来让不同的草不太一样。

```
// Add as new properties.
_BladeWidth("Blade Width", Float) = 0.05
_BladeWidthRandom("Blade Width Random", Float) = 0.02
_BladeHeight("Blade Height", Float) = 0.5
_BladeHeightRandom("Blade Height Random", Float) = 0.3

…

// Add to the CGINCLUDE block.
float _BladeHeight;
float _BladeHeightRandom;	
float _BladeWidth;
float _BladeWidthRandom;

…

// Add to the geometry shader, above the triStream.Append calls.
float height = (rand(pos.zyx) * 2 - 1) * _BladeHeightRandom + _BladeHeight;
float width = (rand(pos.xzy) * 2 - 1) * _BladeWidthRandom + _BladeWidth;

…

// Modify the existing positions with our new height and width.
triStream.Append(VertexOutput(pos + mul(transformationMatrix, float3(width, 0, 0)), float2(0, 0)));
triStream.Append(VertexOutput(pos + mul(transformationMatrix, float3(-width, 0, 0)), float2(1, 0)));
triStream.Append(VertexOutput(pos + mul(transformationMatrix, float3(0, 0, height)), float2(0.5, 1)));
```

![image](https://roystan.net/media/tutorials/grass/grass-thin.png)

现在的三角形比起之前已经很接近真实的草了，但是草的数目还是太少了。这是因为模型mesh的顶点数太少了。

其中一个解决的方法是用unity或者什么3d建模软件创建一个新的mesh。但是这不足以让我们动态的更改草的密度。因此，窝们将使用**曲面细分**来细分输入的mesh。

## 曲面细分

>作者大概介绍了下曲面细分是怎么回事和我们在这里怎么用，但我相信这篇文章的读者都知道，不翻了

首先，我们要引入`CustomTessellation.cginc`。这个文件里包含了曲面细分部分的实现。

```
// Add inside the CGINCLUDE block, below the other #include statements.
#include "Shaders/CustomTessellation.cginc"
```

实际上在这个文件里已经有了`vertexInput`和`vertexOutput`以及顶点着色器了。因此我们shader代码里的这部分可以删掉。

即删掉下面代码框中的所有内容：

```
struct vertexInput
{
	float4 vertex : POSITION;
	float3 normal : NORMAL;
	float4 tangent : TANGENT;
};

struct vertexOutput
{
	float4 vertex : SV_POSITION;
	float3 normal : NORMAL;
	float4 tangent : TANGENT;
};

vertexOutput vert(vertexInput v)
{
	vertexOutput o;
	o.vertex = v.vertex;
	o.normal = v.normal;
	o.tangent = v.tangent;
	return o;
}
```

现在我们可以给草叶shader添加hull shader和domain shader（细分控制着色器和细分计算着色器）的声明了。我们顺便加上一个叫做`_TessellationUniform`的属性来控制曲面细分的程度。和这个变量有关的代码已经在`CustomTessellation.cginc`里写好了。

```
// Add as a new property.			
_TessellationUniform("Tessellation Uniform", Range(1, 64)) = 1

…

// Add below the other #pragma statements in the SubShader Pass.
#pragma hull hull
#pragma domain domain
```

现在通过调节TessellationUniform可以很方便地控制草叶密度了。

![image](https://roystan.net/media/tutorials/grass/grass-tess.gif)

## 风

我们将对变形纹理取样来表示出风。这个纹理和法线图有一点点像，但它只有两个通道（红和绿，妹有蓝）。我们讲使用这两个通道表示风的X和Y方向。

![image](https://roystan.net/media/tutorials/WaterDistortion.png)

在对纹理取样之前，我们首先需要构建一个uv坐标，然后将纹理缩放和变换，最后再加上时间的影响。

```
// Add as new properties.
_WindDistortionMap("Wind Distortion Map", 2D) = "white" {}
_WindFrequency("Wind Frequency", Vector) = (0.05, 0.05, 0, 0)

…

// Add to the CGINCLUDE block.
sampler2D _WindDistortionMap;
float4 _WindDistortionMap_ST;

float2 _WindFrequency;

…

// Add to the geometry shader, just above the line declaring the transformationMatrix.
float2 uv = pos.xz * _WindDistortionMap_ST.xy + _WindDistortionMap_ST.zw + _WindFrequency * _Time.y;
```