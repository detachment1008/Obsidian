# 3.1 Unity Shader 概述

## 3.1.1 材质和 Unity Shader

在 Unity 中，我们需要配合使用材质和 Unity Shader 才能达到需要的效果

一个常见的流程：
1. 创建一个材质
2. 创建一个 Unity Shader，并把它赋给上一步中创建的材质
3. 把材质赋给要渲染的对象
4. 在材质面板中调整 Unity Shader 的属性，以得到满意的结果

## 3.1.2 Unity 中的材质

Unity 中的材质需要结合一个 GameObject 的 Mesh 或者 Particle System 组件来工作。它决定了我们的游戏看起来是什么样的

默认状态下，新建的材质将使用 Unity 内置的 Standard Shader，这是一种基于物理渲染的着色器

## 3.1.3 Unity 中的 Shader

默认提供了五种 Shader：
1. Standard Surface Shader：基于物理的标准光照模型
2. Unlit Shader：不含光照的基本的顶点/片元着色器
3. Image Effect Shader：为我们实现各种屏幕后处理效果提供了一个基本模板
4. Compute Shader：这类 Shader 利用 GPU 的并行性来进行一些与常规渲染流水线无关的计算
5. Ray Tracing Shader：光线追踪 Shader

# 3.2 Unity Shader 的基础：ShaderLab

Unity Shader 是 Unity 为开发者提供的高层级的渲染抽象层。

ShaderLab 是 Unity 提供的编写 Unity Shader 的一种说明性语言。它定义了要显示一个材质所需的所有东西，而不仅仅是着色器代码

基础结构：
```
Shader "ShaderName"
{
	Properties
	{
		// 属性
	}
	SubShader
	{
		// 显卡A使用的子着色器
	}
	SubShader
	{
		// 显卡B使用的子着色器
	}
	Fallback "VertexLit"
}
```

# 3.3 Unity Shader 的结构

## 3.3.1 给我们的 Shader 起个名字

每个 Unity Shader 都需要在第一行通过 `Shader "名字/子名字"` 来指定 Unity Shader 的名字

可以通过 "/" 来控制 Shader 在材质面板中出现的位置

## 3.3.2 材质和 Unity Shader 的桥梁：Properties

Properties 语义块中包含了一系列属性(property)，这些属性将会出现在材质面板中。

通常如下：
```
Properties
{
	Name ("display name", PropertyType) = DefaultValue
	Name ("display name", PropertyType) = DefaultValue
	// 更多属性
}
```

开发者这样声明属性，是为了在材质面板中方便的调整各种材质属性：
- Name：在 Shader 中访问这个属性时使用的变量名
- 第一个字符串：显示的名字
- 第二个参数：属性的类型
	- Int
	- Float
	- Range(min, max)
	- Color
	- Vector
	- 2D
	- Cube
	- 3D
- 第三个参数：指定默认值

Unity 允许我们重载默认的材质编辑面板，以提供更多自定义的数据类型

为了在 Shader 中访问到这些属性，我们还需要在 CG 代码片段中定义和这些属性类型相匹配的变量

即使我们不在 Properties 中声明这些属性，我们还是可以在 CG 代码片中定义这些变量，只是需要通过脚本向 Shader 中传递这些属性

因此，Properties 语义块的作用仅仅是为了让这些属性可以出现在材质面板中

## 3.3 重量级成员：SubShader

每个 Unity Shader 文件可以包含多个 SubShader 语义块，但是最少要有一个

当 Unity 需要加载这个 Unity Shader 时，就会扫描所有的 SubShader 语义块，然后选择第一个能够在目标平台上运行的 SubShader。如果都不支持的话，Unity 就会用 Fallback 语义指定的 Unity Shader

Unity 提供这种语义的原因是不同显卡有不同的能力，我们希望在旧的显卡上使用计算复杂度低的着色器，而在高级的显卡上使用计算复杂度高的着色器

SubShader的通常定义：
```
SubShader
{
	// 可选
	[Tags]

	// 可选
	[RenderSetup]

	Pass
	{
	
	}
	// Other Passes
}
```

它定义了可选的标签和状态，还要一系列的 Pass

每个 Pass 都定义了一次完整的渲染流程，但是如果 Pass 过多，往往会造成 Unity 的渲染性能下降，所以我们尽可能使用少的 Pass

标签设置：是一个键值对，键和值都是字符串，用于 SubShader 和渲染引擎之间沟通的桥梁，即用于告诉 Unity 渲染引擎希望怎样及何时渲染这个对象

结构：`Tags {"TagName1" = "Value1" "TagName2" = "Value2"}`
- Queue
- RenderType
- DisableBatching
- ForceNoShadowCasting
- IgnoreProjector
- CanUseSpriteAtlas
- PreviewType

注意：标签不能在 Pass 的内部声明。Pass 内部的标签不同于上面说的标签

状态设置：提供了一系列渲染状态的设置指令，如是否开启混合/深度测试等
- Cull：剔除模式
- ZTest：深度测试
- ZWrite：深度写入
- Blend：混合模式

如果在 Pass 内部进行设置，那么它只会应用到这个 Pass 内部，否则它影响全部的 Pass

Pass 语义块：

结构：
```
Pass
{
	[Name]
	[Tags]
	[RenderSetup]
	// Other code
}
```

- 定义名称：`Name "名字"`
	- 可以使用 `UsePass` 命令来直接使用其他 Unity Shader 中的 Pass
	- Unity 内部会把所有 Pass 的名字转化为大写字母
- 设置标签：也是告诉渲染引擎怎样进行渲染
	- LightMode
	- RequireOptions
- 设置状态：同上，不过只应用于 Pass 内部

特殊的 Pass：
- UsePass：复用其他 Shader 中的 Pass
- GrabPass：抓取屏幕，并将结果存储在一张纹理中，用于后续的 Pass 处理

## 3.3.4 留一条后路：Fallback

结构：
```
Fallback "name"
// 或者
Fallback Off
```

它用于告诉 Unity 引擎，最低级的 Unity Shader 是谁

## 3.3.5 ShaderLab 还有其他语义

可以使用语义进行扩展：
- CustomEditor：扩展编辑界面
- Category：命令分组

# 3.4 Unity Shader 的形式

## 3.4.1 Unity 的宠儿：表面着色器

渲染代价很大，背后仍会被转化为顶点/片元着色器

可以理解为表面着色器是 Unity 为顶点/片元着色器的更高一层的抽象

它存在的价值在于：Unity 为我们处理了很多光照细节

它被定义在 CGPROGRAM 和 ENDCG 之间：因为表面着色器不关注开发者使用多少 Pass，每个 Pass 怎样渲染的问题

这段代码是 CG/HLSL 编写的，语法几乎一模一样

## 3.4.2 最聪明的孩子：顶点/片元着色器

同样使用 CGPROGRAM 和 ENDCG 包围的 CG/HLSL 来编写，但是需要放在 Pass 之内

## 3.4.3 被抛弃的角落：固定函数着色器

老旧的设备可能不支持可编程管线着色器，因此我们旧只能使用固定函数着色器来完成渲染。但是这些着色器只能完成一些非常简单的效果

完全使用 ShaderLab 的语法，而非使用 CG/HLSL

## 3.4.4 选择哪种 Unity Shader 的形式

建议：
- 尽量使用可编程管线着色器
- 如果希望和各种光源打交道：表面着色器。但是要注意性能
- 光照数目非常少：顶点/片元着色器
- 有很多自定义的效果：顶点/片元着色器

如果坚持想使用 GLSL 来编写：嵌套在 GLSLPROGRAM 和 ENDGLSL 之间。但是对于仅支持 DirectX 的平台来说，就只能放弃了。