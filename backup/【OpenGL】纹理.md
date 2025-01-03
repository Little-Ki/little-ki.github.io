> 纹理可以是1D / 2D / 3D，用来添加物体的细节。


# API
```c
// 生成纹理 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glGenTextures.xhtml)
void glGenTextures(GLsizei n, GLuint * textures);

// 绑定纹理 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glBindTexture.xhtml)
void glGenTextures(GLenum target, GLuint textures);

// 上传纹理数据
// [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexImage1D.xhtml)
void glTexImage1D(
	GLenum target, GLint level, GLint internalformat, GLsizei width,
	GLint border, GLenum format, GLenum type, const void * data);
// [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexImage2D.xhtml)
void glTexImage2D(
	GLenum target, GLint level, GLint internalformat, GLsizei width, GLsizei height, 
	GLint border, GLenum format, GLenum type, const void * data);
// [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexImage3D.xhtml)
void glTexImage3D(
	GLenum target, GLint level, GLint internalformat, GLsizei width, GLsizei height,
	GLsizei depth, GLint border, GLenum format, GLenum type, const void * data);
// 生成mipmap [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glGenerateMipmap.xhtml)
void glGenerateMipmap(GLenum target);
void glGenerateTextureMipmap(GLuint texture);

// 设置纹理参数 [参考](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexParameter.xhtml)
void glTexParameterf(GLenum target, GLenum pname, GLfloat param);
void glTexParameteri(GLenum target, GLenum pname, GLint param);
void glTextureParameterf(GLuint texture, GLenum pname, GLfloat param);
void glTextureParameteri(GLuint texture, GLenum pname, GLint param);
void glTexParameterfv(GLenum target, GLenum pname, const GLfloat * params);
void glTexParameteriv(GLenum target, GLenum pname, const GLint * params);
void glTexParameterIiv(GLenum target, GLenum pname, const GLint * params);
void glTexParameterIuiv(GLenum target, GLenum pname, const GLuint * params);
void glTextureParameterfv(GLuint texture, GLenum pname, const GLfloat *params);
void glTextureParameteriv(GLuint texture, GLenum pname, const GLint *params);
void glTextureParameterIiv(GLuint texture, GLenum pname, const GLint *params);
void glTextureParameterIuiv(GLuint texture, GLenum pname, const GLuint *params);
```

> [参考文档](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexParameter.xhtml)

# 纹理环绕方式
> 纹理坐标 u v 分量映射到 $`[0, 1]`$，用于采样图片上的点。环绕方式决定了当超出该范围时的采样方式，默认为重复（循环）采样

| 环绕方式 | 行为 |
| :-: | :-: |
| GL_REPEAT | 对纹理的默认行为。重复纹理图像。 |
| GL_MIRRORED_REPEAT | 和GL_REPEAT一样，但每次重复图片是镜像放置的。|
| GL_CLAMP_TO_EDGE | 纹理坐标被限制在[0, 1]，超出时采样边缘 |
| GL_CLAMP_TO_BORDER | 超出时使用指定颜色填充 |

```c
// 设置各个方向的纹理环绕方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_R, GL_MIRRORED_REPEAT);
// 若采样方式为 GL_CLAMP_TO_BORDER，可指定颜色值
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f };
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```

# 纹理采样方式
> 采样方式决定了如何采样颜色，OpenGL 有多种采样方式，其中最重要的有 GL_NEAREST，GL_LINEAR
> GL_NEAREST：邻近采样，使用离采样点最近的像素点作为颜色值
> GL_LINEAR：线性插值，在采样点及其附近像素点进行插值计算最终颜色

```c
// 设置纹理的采样方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

# 多级渐远纹理（MipMap）
> 此方式为单个纹理生成不同大小的纹理图像，并根据顶点距离来决定采样哪一个纹理，用于优化采样速度
> OpenGL 具有函数 glGenerateMipmap 用于自动生成纹理，使用以下参数指定生成纹理时的采样方式
| 采样方式 | 行为 |
| :-: | :-: |
| GL_NEAREST_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样 |
| GL_LINEAR_MIPMAP_NEAREST | 使用最邻近的多级渐远纹理级别，并使用线性插值进行采样 |
| GL_NEAREST_MIPMAP_LINEAR | 在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样 |
| GL_LINEAR_MIPMAP_LINEAR | 在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样 |

```c
// 设置MipMap的采样方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

# 生成纹理
```c
unsigned int texture; // 储存纹理 ID
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture); // 绑定 2D 纹理到纹理 ID
// 上传纹理数据 参数为 纹理模板 mipmap级别 颜色格式 图片宽度 图片高度 保留 颜色分量数据格式 指向图片数据的指针
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D);
```
