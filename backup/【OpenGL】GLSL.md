# 简介
&emsp;&emsp;GLSL（OpenGL Shading Language） 全称 OpenGL 着色语言，是用来在 OpenGL 中着色编程的语言，也即开发人员写的短小的自定义程序，他们是在图形卡的 GPU上执行的，代替了固定的渲染管线的一部分，使渲染管线中不同层次具有可编程性。 GLSL 其使用 C 语言作为基础高阶着色语言，避免了使用汇编语言或硬件规格语言的复杂性，本篇**参考版本 >= 3.0**。

# 基本语法
## 着色器基本结构
一个典型的着色器有下面的结构：

```glsl
#version version_number
in type in_variable_name;
in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

void main()
{
  // 处理输入并进行一些图形操作
  ...
  // 输出处理过的结果到输出变量
  out_variable_name = weird_stuff_we_processed;
}
```

<details><summary>着色器示例</summary>


```glsl
#version 330 core
// 顶点着色器

layout (location = 0) in vec3 aPos; // 输入坐标

out vec4 color; // 为片段着色器指定一个颜色输出

void main()
{
    gl_Position = vec4(aPos, 1.0); // 输出顶点
    color = vec4(0.5, 0.0, 0.0, 1.0); // 输出颜色
}
```

```glsl
#version 330 core
// 片段着色器

out vec4 FragColor;

in vec4 color; // 从顶点着色器传来的输入变量（名称相同、类型相同）

void main()
{
    FragColor = color;
}
```

</details>

# 数据类型

## 基础数据类型
GLSL和其他编程语言一样，GLSL有数据类型可以来指定变量的种类。GLSL中包含C等其它语言大部分的默认基础数据类型：`int`、`float`、`double`、`uint`和`bool`。GLSL也有两种容器类型，分别是向量(Vector)和矩阵(Matrix)。

## 向量
GLSL中的向量是一个可以包含有2、3或者4个分量的容器，分量的类型可以是前面默认基础类型的任意一个。它们可以是下面的形式（N代表分量的数量）：
| 类型 | 描述 |
| :-:  | :-: |
| vecN |  包含N个float分量的默认向量 |
| ivecN | 包含N个int分量的向量 | 
| uvecN | 包含N个uint分量的向量 |
| bvecN | 包含N个bool分量的向量 |
| dvecN | 包含N个double分量的向量 |

## 矩阵

GLSL中的矩阵是一个可以包含有M * N个分量的容器，其中 N 和 M 可以是2、3、4，分量的类型可以是前面默认基础类型的任意一个。它们可以是下面的形式：
| 类型 | 描述 |
| :-: | :-: |
| matN | NxN 大小的浮点型矩阵 |
| matMxN | M行N列的浮点型矩阵（OpenGl的矩阵是列主顺序的） |

## 纹理采样类型
| 类型 | 描述 |
| :-:  | :-: |
| sampler1D |  用于内建的纹理函数中引用指定的 1D 纹理的句柄。只可以作为一致变量或者函数参数使用 |
| sampler2D |  二维纹理句柄 | 
| sampler3D |  三维纹理句柄   |
| samplerCube |  CubeMap 纹理句柄  |
| sampler1DShadow | 一维深度纹理句柄 |
| sampler2DShadow | 二维深度纹理句柄 |

## 结构体
```glsl
// 结构体可以组合基本类型和数组来形成用户自定义的类型。在定义一个结构体的同时，你可以定义一个结构体实例。或者后面再定义。

struct someType {
 vec4 color;
 float start;
 float end;
 vec3 points[3];
} someVal;
```

## 数组
```glsl
// glsl 中只可以使用一维的数组。数组的类型可以是一切基本类型或者结构体。下面的几种数组声明是合法的
float a[4];
vec4 b[2];
float c[4] = float[](1.0,2.0,3.0,4.0);
vec2 d[2] = vec2[2](vec2(1.0, 2.0),vec2(3.0, 4.0));

// 数组类型内建了一个length()函数，可以返回数组的长度。
lightPositions.length()
```

# 内置变量和函数
## 内置变量
> OpenGL 着色语言为各个着色器阶段定义了许多特殊变量。这些内置变量（或内置变量）具有特殊的属性。它们通常用于与某些固定功能进行通信。按照惯例，所有预定义变量都以“gl_”开头；用户定义的变量不能以此开头。
### 顶点着色器输入输出
```glsl
// 顶点着色器具有以下内置输入变量
in int gl_VertexID; // 当前正在处理的顶点的索引
in int gl_InstanceID; // 进行某种形式的实例渲染时当前实例的索引
in int gl_DrawID; // 需要 GLSL 4.60 或ARB_shader_draw_parameters
in int gl_BaseVertex; // 需要 GLSL 4.60 或ARB_shader_draw_parameters
in int gl_BaseInstance; // 需要 GLSL 4.60 或ARB_shader_draw_parameters

// 顶点着色器具有以下预定义输出
out gl_PerVertex
{
  vec4 gl_Position; // 输出属性-变换后的顶点的位置
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

### 片段着色器输入
```glsl
// 片段着色器具有以下内置输入变量
in vec4 gl_FragCoord; // 只读输入，窗口的 x, y, z 和1/w
in bool gl_FrontFacing; // 只读输入，如果是窗口正面图元的一部分，则这个值为true
in vec2 gl_PointCoord; // 点精灵的二维空间坐标范围在(0.0, 0.0)到(1.0, 1.0)之间，仅用于点图元和点精灵开启的情况下。
```

## 内置函数
> glsl 提供了非常丰富的函数库,供我们使用,这些功能都是非常有用且会经常用到的. 这些函数按功能区分大改可以分成7类:
> 下文中的 类型 T 可以是 float, vec2, vec3, vec4,且可以逐分量操作

### 通用函数:
```glsl
T abs(T x) // 返回 x 的绝对值
T sign(T x) // 比较 x 与 0 的值，大于，等于，小于分别返回 1.0，0.0，-1.0
T floor(T x) // 向下取整
T ceil(T x) // 向上取整
T fract(T x) // 取 x 的小数部分
// x 对 y 取余
T mod(T x, T y) 
T mod(T x, float y)
// 取最小值
T min(T x, T y)
T min(T x, float y)
// 取最大值
T max(T x, T y)
T max(T x, float y)
// 将 x 限定为 minVal <= x <= maxVal
T clamp(T x, T minVal, T maxVal) 
T clamp(T x, float minVal, float maxVal)
// 取 x，y 的线性混合，x(1-a)+ya
T mix(T x, T y, T a)
T mix(T x, T y, float a)
// x < edge ? 0.0 : 1.0
T step(T edge, T x)
T step(float edge, T x)
// 如果 x < edge0 返回 0.0，如果 x > edge1返回 1.0，否则返回Hermite插值
T smoothstep(T edge0, T edge1, T x)
T smoothstep(float edge0,float edge1, T x)
```

### 角度&三角函数
```glsl
T radians(T degrees) // 角度转弧度
T degrees(T radians) // 弧度转角度
T sin(T angle) // 正弦函数，参数是弧度
T cos(T angle) // 余弦函数，参数是弧度
T tan(T angle) // 正切函数，参数是弧度
T asin(T x)  // 反正弦函数，返回值是弧度
T acos(T x) // 反余弦函数，返回值是弧度
 // 反正切函数，返回值是弧度
T atan(T x)
T atan(T y_over_x)
```

### 指数函数
```glsl
T pow(T x, T y) // 返回 x 的 y 次幂
T exp(T x) // 返回 x 的自然指数幂 ex
T log(T x) // 返回 x 的自然对数 ln
T exp2(T x) // 返回 2 的 x 次幂
T log2(T x) // 返回以 2 为底的 x 的对数
T sqrt(T x) // 开方 x
T inversesqrt(T x) // 求平方根倒数
```

### 几何函数
```glsl
float length(T x) // 返回向量长度
float distance(T p0, T p1) // 返回两点距离
float dot(T x, T y) // 返回向量点积
vec3 cross(vec3 x, vec3 y) // 返回向量叉积
T normalize(T x) // 返回单位向量
T faceforward(T N, T I, T Nref) // 根据 矢量 N 与 Nref 调整法向量
T reflect(T I, T N) // 返回 I - 2 * dot(N, I) * N（结果是入射矢量 I 关于法向量N的镜面反射向量）
T refract(T I, T N, float eta) // 返回入射矢量 I 关于法向量 N 的折射向量,折射率为eta
```

### 矩阵函数
```glsl
mat matrixCompMult(mat x, mat y) // 将矩阵 x 和 y的元素逐分量相乘
```

### 向量函数
> bvec指的是由bool类型组成的一个向量
> 
```glsl
vec3 v3= vec3(0.,0.,0.);
vec3 v3_1= vec3(1.,1.,1.);
bvec3 aa= lessThan(v3,v3_1); //bvec3(true,true,true)
```

```glsl
bvec lessThan(T x, T y) // 逐分量比较x < y,将结果写入bvec对应位置
bvec lessThanEqual(T x, T y) // 逐分量比较 x <= y,将结果写入bvec对应位置
bvec greaterThan(T x, T y) // 逐分量比较 x > y,将结果写入bvec对应位置
bvec greaterThanEqual(T x, T y) // 逐分量比较 x >= y,将结果写入bvec对应位置
// 逐分量比较 x == y,将结果写入bvec对应位置
bvec equal(T x, T y) 
bvec equal(bvec x, bvec y)
// 逐分量比较 x!= y,将结果写入bvec对应位置
bvec notEqual(T x, T y)
bvec notEqual(bvec x, bvec y)
bool any(bvec x) // 如果x的任意一个分量是true,则结果为true
bool all(bvec x) // 如果x的所有分量是true,则结果为true
bvec not(bvec x) // bool矢量的逐分量取反
```

### 纹理查询函数

> 图像纹理有两种 一种是平面2d纹理,另一种是盒纹理,针对不同的纹理类型有不同访问方法.
> 纹理查询的最终目的是从sampler中提取指定坐标的颜色信息. 函数中带有Cube字样的是指 需要传入盒状纹理. 带有Proj字样的是指带投影的版本.
```glsl
vec4 texture(sampler2D sampler, vec2 coord);
vec4 texture(samplerCube sampler, vec3 coord);
vec4 textureProj(sampler2D sampler, vec3 coord);
vec4 textureProj(sampler2D sampler, vec4 coord);
```