> 术语：
> 顶点数组对象：Vertex Array Object，VAO，保存了多个顶点数组对象，用于在多个顶点数组间切换
> 顶点缓冲对象：Vertex Buffer Object，VBO，储存了单个顶点数组
> 元素缓冲对象：Element Buffer Object，EBO 或 索引缓冲对象 Index Buffer Object，IBO，保存了要绘制的顶点在数组中的索引
# API
```c
// 生成顶点数组对象 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glGenVertexArrays.xhtml)
void glGenVertexArrays(GLsizei n, GLuint *arrays);
// 绑定顶点数组对象 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBindVertexArray.xhtml)
void glBindVertexArray(GLuint array);

// 申请缓冲区 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glGenBuffers.xhtml)
void glGenBuffers(GLsizei n, GLuint * buffers);
// 将缓冲区绑定至目标 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBindBuffer.xhtml)
void glBindBuffer(GLenum target, GLuint buffer);

// 上传数据至缓冲区[参考] (https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBufferData.xhtml)
void glBufferData(GLenum target, GLsizeiptr size, const void * data, GLenum usage);
void glNamedBufferData(GLuint buffer, GLsizeiptr size, const void *data, GLenum usage);

// 上传数据至缓冲区（部分）
// [参考] (https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBufferSubData.xhtml)
void glBufferSubData(GLenum target, GLintptr offset, GLsizeiptr size, const void * data);
void glNamedBufferSubData(GLuint buffer, GLintptr offset, GLsizeiptr size, const void *data);

// 创建着色器程序 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glCreateProgram.xhtml)
GLuint glCreateProgram(void);
// 将着色器附加到程序 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glAttachShader.xhtml)
void glAttachShader(GLuint program, GLuint shader);
// 链接着色器程序 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glLinkProgram.xhtml)
void glLinkProgram(GLuint program);
// 获取着色器程序参数 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glGetProgram.xhtml)
void glGetProgramiv(GLuint program, GLenum pname, GLint *params);
// 删除着色器程序
void glDeleteProgram(GLuint program);

// 创建着色器 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glCreateShader.xhtml)
GLuint glCreateShader(GLenum shaderType);
// 设置着色器代码 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glShaderSource.xhtml)
void glShaderSource(GLuint shader, GLsizei count, const GLchar **string, const GLint *length);
// 编译着色器 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glCompileShader.xhtml)
void glCompileShader(GLuint shader);
// 获取着色器参数 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glGetProgram.xhtml)
void glGetShaderiv(GLuint shader, GLenum pname, GLint *params);
// 删除着色器
void glDeleteShader(GLuint shader);

// 定义通用顶点属性数据的数组
// [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glVertexAttribPointer.xhtml)
void glVertexAttribPointer(
	GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const void * pointer);
void glVertexAttribIPointer(GLuint index, GLint size, GLenum type, GLsizei stride, const void * pointer);
void glVertexAttribLPointer(GLuint index, GLint size, GLenum type, GLsizei stride, const void * pointer);

// 使用索引绘制顶点 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glDrawElements.xhtml)
void glDrawElements(GLenum mode, GLsizei count, GLenum type, const void * indices);
```
# 顶点数组对象（VAO）
```c
unsigned int VAO;
// 申请一个顶点数组对象
// 一个数组对象关联了一个VBO / EBO
// 当在多个不同的VBO / EBO之间切换时，使用此方法
glGenVertexArrays(1, &VAO);
// 绑定VAO
glBindVertexArray(VAO);
// 之后的操作将在其上进行
```

# 顶点缓冲对象（VBO）
```c
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f 
};
unsigned int VBO;
// 申请了一个顶点缓冲区
glGenBuffers(1, &VBO);
// 绑定至 GL_ARRAY_BUFFER
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
// 上传数据 其中第四个参数指定了如何管理数据
// GL_STATIC_DRAW ：数据不会或几乎不会改变。
// GL_DYNAMIC_DRAW：数据会被改变很多。
// GL_STREAM_DRAW ：数据每次绘制时都会改变。
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

# 索引缓冲对象（EBO）
```c
// 当顶点数较多，并且有大量三角形共用顶点，需要用此方式来减少数据量
unsigned int EBO;
// 申请了一个索引缓冲区
glGenBuffers(1, &EBO);
// 绑定至 GL_ELEMENT_ARRAY_BUFFER
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
// 上传索引数据
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```


# 编译着色器
```c
const char *shaderSource= "......";
unsigned int shaderId;
// 参数可以为 GL_COMPUTE_SHADER，GL_VERTEX_SHADER
// GL_TESS_CONTROL_SHADER，GL_TESS_EVALUATION_SHADER
// GL_GEOMETRY_SHADER，GL_FRAGMENT_SHADER
shaderId= glCreateShader(...);
glShaderSource(shaderId, 1, &shaderSource, NULL);
glCompileShader(shaderId);
...
// 检查是否编译成功
glGetShaderiv(shaderId, GL_COMPILE_STATUS, &success);
if(!success)
{
    glGetShaderInfoLog(shaderId, 512, NULL, infoLog);
    ...
}

// 不再使用时
glDeleteShader(shadeId);
```

# 着色器程序
```c
unsigned int programId;
programId= glCreateProgram();
// ... 编译顶点着色器，将其附加到程序
glAttachShader(programId, vertexShader);
// ... 编译片段着色器，将其附加到程序
glAttachShader(programId, fragmentShader);
// 链接着色器程序
glLinkProgram(programId);
...
// 检查是否链接成功
glGetProgramiv(programId, GL_LINK_STATUS, &success);
if(!success) {
    glGetProgramInfoLog(programId, 512, NULL, infoLog);
    ...
}

// 不再使用时
glDeleteProgram(programId);
```

# 链接顶点属性
> 着色器需要知道顶点所包含的数据，一个顶点可以包含如坐标、颜色、纹理坐标等多个参数
> 需要由用户声明每个参数的属性，告诉OpenGL该如何解析顶点数据

```c
// 声明了一个顶点参数 位置为 0（对应着色器内的 location），包含 3 个分量，分量属性为float，不启用归一化
// 大小为 3 * sizeof(float)，相对偏移（在结构体内的偏移）为 0
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
// 启用位置 0 的顶点属性
glEnableVertexAttribArray(0);
```

# 代码示例
```c
struct Color {
	float r, g, b, a;
};
struct Pos {
	float x, y, z;
};
struct TexCoord {
	float u, v;
};
struct Vert {
	Pos pos;
	Color col;
	TexCoord uv;
};

GLuint vao, vbo, ebo, vs, fs, program;
Vert[] vertices = { ... };
unsigned int[] indices = { ... };
// ... 编译顶点着色器与片段着色器
// ... 创建着色器程序，附加着色器并链接
// ... 创建VAO，VBO，EBO
// 绑定VAO
glBindVertexArray(VAO);
// 绑定VBO
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 绑定EBO
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
// 启用第 0 个属性
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vert), (void*)0);
glEnableVertexAttribArray(0);
// 启用第 1 个属性
glVertexAttribPointer(1, 4, GL_FLOAT, GL_FALSE, sizeof(Vert), (void*)sizeof(Pos));
glEnableVertexAttribArray(1);
// 启用第 2 个属性
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, sizeof(Vert), (void*)(sizeof(Pos) + sizeof(Color)));
glEnableVertexAttribArray(2);
...
// 启用着色器程序
glUseProgram(programId);
// 使用该VAO关联的顶点
glBindVertexArray(vao);
// 使用顶点及其索引绘制三角洲
// 绘制方式为 GL_TRIANGLES，可以是以下之一 
// GL_POINTS，GL_LINE_STRIP，GL_LINE_LOOP，GL_LINES，GL_LINE_STRIP_ADJACENCY，
// GL_LINES_ADJACENCY，GL_TRIANGLE_STRIP，GL_TRIANGLE_FAN，GL_TRIANGLES，
// GL_TRIANGLE_STRIP_ADJACENCY，GL_TRIANGLES_ADJACENCY、GL_PATCHES
// 索引数量为, sizeof(indices)
// 起始偏移为 0
// 索引数据类型为 unsigned int
glDrawElements(GL_TRIANGLES, sizeof(indices), GL_UNSIGNED_INT, 0);
glBindVertexArray(0);
```


<details><summary>着色器参考</summary>

```glsl
#version 330 core
// 顶点着色器
layout (location=0) in vec3 aPos;
layout (location=1) in vec4 aColor;
layout (location=2) in vec2 aUV;

out vec2 iUV;
out vec4 iColor;

void main()
{
    iUV = aUV;
    iColor = aColor;
    gl_Position=vec4(aPos,1.0)
}
```

```glsl
#version 330 core
// 片段着色器
out vec4 fragColor;

in vec2 iUV;
in vec4 iColor;

void main()
{
    FragColor = iColor;
}
```

</details>
