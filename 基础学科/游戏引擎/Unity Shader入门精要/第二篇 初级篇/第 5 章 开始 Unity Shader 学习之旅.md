# 5.1 本书使用的软件和环境

# 5.2 一个最简单的顶点/片元着色器

## 5.2.1 顶点/片元着色器的基本结构

示例程序：
```
// 定义名字
Shader "Unity Shaders Book/Chapter 5/Simple Shader"
{
	// 声明 SubShader
    SubShader
    {
		// 声明 Pass
        pass
        {
            CGPROGRAM

			// 语句格式：#pragma vertex 函数名
			// 它告诉编译器
			// 哪个函数包含了顶点着色器的代码
			// 哪个函数包含了片元着色器的代码
            #pragma vertex vert
            #pragma fragment frag

			// 输入 v 包含了顶点的位置。通过 POSITION 语义指定的
			// 返回的是一个 float4 类型的变量，是该顶点在裁剪空间中的位置
		    // SV_POSITION 表示顶点着色器的输出是裁剪空间中的顶点坐标
		    // 如果没有语义来限制的话，渲染器完全不知道用户的输入和输出是什么
		    // UNITY_MATRIX_MVP 矩阵是内置的模型，观察，投影矩阵
            float4 vert(float4 v:POSITION) : SV_POSITION
            {
                return mul (UNITY_MATRIX_MVP, v);
                // 会被自动更改为：return UnityObjectToClipPos (v);
            }

			// SV_Target：告诉渲染器将用户输出的颜色存储到渲染目标里
            fixed4 frag() : SV_Target
            {
                return fixed4(0.0, 1.0, 1.0, 1.0);
            }

            ENDCG
        }
    }
}
```

## 5.2.2 模型数据从哪里来

在上述例子，我们只能通过 POSITION 语义来得到模型的顶点位置

所以我们需要一个新的输入参数

```
	struct a2v
    {
		float4 vertex : POSITION;
        float3 normal:NORMAL;
        float4 texcoord:TEXCOORD0;
    }

    float4 vert(a2v v) : SV_POSITION
    {
        return UnityObjectToClipPos (v.vertex);
    }
```

Unity 会根据语义来填充这个结构体

Unity 支持的语义：
- POSITION
- TANGENT
- NORMAL
- TEXCOORD0
- TEXCOORD1
- TEXCOORD2
- TEXCOORD3
- COLOR

创建一个结构体的格式：
```
struct 结构体名字
{
	Type Name : Semantic;
}
```
- 语义不可以被忽略

语义中的数据是通过 Mesh Render 组件提供的，每帧调用 Draw Call 的时候，Mesh Render 组件就会把它负责渲染的模型数据发给 Unity Shader

## 5.2.3 顶点着色器和片元着色器之间如何通信

我们往往会希望从顶点着色器输出一些数据，例如法线，纹理坐标等传递给片元着色器

这就涉及到了顶点着色器和片元着色器之间的通信：我们可以通过顶点着色器的输出值，传递给片元着色器来完成

## 5.2.4 如何使用属性

我们可以定义属性，以便外部通过 Inspector 窗口便捷的修改它。

当我们使用这个属性时，需要定义一个和它类型对应，名称相同的变量，才能够使用它

# 5.3 强大的援手：Unity 提供的内置文件和变量