# 术语：
> 顶点数组对象：Vertex Array Object，VAO，保存了多个顶点数组对象，用于在多个顶点数组间切换.
> 顶点缓冲对象：Vertex Buffer Object，VBO，储存了单个顶点数组。
> 元素缓冲对象：Element Buffer Object，EBO 或 索引缓冲对象 Index Buffer Object，IBO，储存了绘制顶点的索引。

# 顶点数组对象
```c
void glGenVertexArrays(GLsizei n, GLuint *arrays);
void glBindVertexArray(GLuint array);
```
&emsp;&emsp;glGenVertexArrays申请一个或多个顶点数组对象，在执行操作之前需要使用glBindVertexArray绑定至某个对象。

# 顶点属性
```c
// 用于声明属性的属性
void glVertexAttribPointer(
	GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei stride, const void * pointer);
void glVertexAttribIPointer(GLuint index, GLint size, GLenum type, GLsizei stride, const void * pointer);
void glVertexAttribLPointer(GLuint index, GLint size, GLenum type, GLsizei stride, const void * pointer);

// 用于启用或停用指定索引的顶点属性
void glEnableVertexAttribArray(GLuint index); 
void glDisableVertexAttribArray(GLuint index);
void glEnableVertexArrayAttrib(GLuint vaobj, GLuint index); 
void glDisableVertexArrayAttrib(GLuint vaobj, GLuint index);
```

&emsp;&emsp;对于含有多个属性的顶点，使用此方式声明每个属性的属性。
&emsp;&emsp;其中index为顶点属性的索引，size指定该属性的成员数（1，2，3或4），type指定该属性的的成员的类型。
&emsp;&emsp;对于type属性，GL_BYTE、 GL_UNSIGNED_BYTE、 GL_SHORT、 GL_UNSIGNED_SHORT、 GL_INT、GL_UNSIGNED_INT可用于函数glVertexAttribPointer和glVertexAttribIPointer，GL_HALF_FLOAT、GL_FLOAT、GL_DOUBLE、GL_FIXED、GL_INT_2_10_10_10_REV、GL_UNSIGNED_INT_2_10_10_10_REV、GL_UNSIGNED_INT_10F_11F_11F_REV可用于glVertexAttribPointer，GL_DOUBLE也可用于该函数且仅可用于该函数。
&emsp;&emsp;normalized指定是否对属性进行归一化，GL_TRUE或GL_FALSE。
&emsp;&emsp;stride指定连续通用顶点属性之间的字节偏移量，如果为 0，则通用顶点属性被理解为紧密包装在数组中。初始值为0。
&emsp;&emsp;pointer指定当前绑定到 GL_ARRAY_BUFFER 目标的缓冲区的数据存储中数组中第一个通用顶点属性的第一个组件的偏移量。

# 数据缓冲
## 申请与绑定
```c
void glGenBuffers(GLsizei n, GLuint * buffers);
void glBindBuffer(GLenum target, GLuint array);
```
&emsp;&emsp;glGenBuffers在显存中申请缓冲区，返回缓冲区id。使用glBindBuffer可为缓冲区指定目标，取决于参数target。
&emsp;&emsp;可用于VBO与EBO

## 上传数据
```c
void glBufferData(GLenum target, GLsizeiptr size, const void * data, GLenum usage);
void glNamedBufferData(GLuint buffer, GLsizeiptr size, const void *data, GLenum usage);
void glBufferSubData(GLenum target, GLintptr offset, GLsizeiptr size, const void * data);
void glNamedBufferSubData(GLuint buffer, GLintptr offset, GLsizeiptr size, const void *data);
```
&emsp;&emsp;在使用glBindBuffer将缓冲区绑定至目标后，可以使用此api上传数据，参数target指定api绑定缓冲区对象的目标。在此之前使用glBindBuffer先将缓冲区绑定至目标。

## target

|值|说明|
|:-:|:-:|
|GL_ARRAY_BUFFER|顶点属性|
|GL_ATOMIC_COUNTER_BUFFER|原子计数器存储|
|GL_COPY_READ_BUFFER|缓冲区复制源|
|GL_COPY_WRITE_BUFFER|缓冲区复制目标|
|GL_DISPATCH_INDIRECT_BUFFER|间接计算调度命令|
|GL_DRAW_INDIRECT_BUFFER|间接命令参数|
|GL_ELEMENT_ARRAY_BUFFER|顶点索引数组|
|GL_PIXEL_PACK_BUFFER|像素读取目标|
|GL_PIXEL_UNPACK_BUFFER|纹理数据源|
|GL_QUERY_BUFFER|查询结果缓冲区|
|GL_SHADER_STORAGE_BUFFER|着色器的读写存储|
|GL_TEXTURE_BUFFER|纹理数据缓冲区|
|GL_TRANSFORM_FEEDBACK_BUFFER|变换反馈缓冲区|
|GL_UNIFORM_BUFFER|Uniform 块存储|


# 着色器与程序

## 程序
```c
GLuint glCreateProgram(); // 创建，返回程序id
void glAttachShader(GLuint program, GLuint shader); // 将着色器附加到程序
void glLinkProgram(GLuint program); // 链接程序
void glGetProgramiv(GLuint program, GLenum pname, GLint *params); // 查询程序参数
void glDeleteProgram(GLuint program); // 删除重新
void glGetProgramInfoLog(GLuint program, GLsizei maxLength, GLsizei *length, GLchar *infoLog); // 获取日志
```
&emsp;&emsp;glGetProgramiv创建用于查询程序的状态，其中pname指定了属性名，可选项：GL_DELETE_STATUS、GL_LINK_STATUS、GL_VALIDATE_STATUS、GL_INFO_LOG_LENGTH、GL_ATTACHED_SHADERS、GL_ACTIVE_ATOMIC_COUNTER_BUFFERS、GL_ACTIVE_ATTRIBUTES、GL_ACTIVE_ATTRIBUTE_MAX_LENGTH、GL_ACTIVE_UNIFORMS、GL_ACTIVE_UNIFORM_BLOCKS、GL_ACTIVE_UNIFORM_BLOCK_MAX_NAME_LENGTH、GL_ACTIVE_UNIFORM_MAX_LENGTH、GL_COMPUTE_WORK_GROUP_SIZE、GL_PROGRAM_BINARY_LENGTH、GL_TRANSFORM_FEEDBACK_BUFFER_MODE、GL_TRANSFORM_FEEDBACK_VARYINGS、GL_TRANSFORM_FEEDBACK_VARYING_MAX_LENGTH、GL_GEOMETRY_VERTICES_OUT、GL_GEOMETRY_INPUT_TYPE、GL_GEOMETRY_OUTPUT_TYPE。

## 着色器
```c
GLuint glCreateShader(GLenum shaderType); // 创建着色器
void glShaderSource(GLuint shader, GLsizei count, const GLchar **string, const GLint *length); // 上传着色器代码
void glCompileShader(GLuint shader); // 编译着色器
void glGetShaderiv(GLuint shader, GLenum pname, GLint *params); // 查询着色器参数
void glDeleteShader(GLuint shader); // 删除着色器
void glGetShaderInfoLog(GLuint shader, GLsizei maxLength, GLsizei *length, GLchar *infoLog); // 获取日志
``` 
&emsp;&emsp;glCreateShader创建指定类型的着色器，参数可为：GL_COMPUTE_SHADER、GL_VERTEX_SHADER、GL_TESS_CONTROL_SHADER、GL_TESS_EVALUATION_SHADER、GL_GEOMETRY_SHADER、GL_FRAGMENT_SHADER。
&emsp;&emsp;glGetShaderiv创建用于查询着色器的状态，其中pname指定了属性名，可选项：GL_SHADER_TYPE、GL_DELETE_STATUS、GL_COMPILE_STATUS、GL_INFO_LOG_LENGTH、GL_SHADER_SOURCE_LENGTH。
