[原文](http://learnopengl.com/#!Lighting/Lighting-maps)

# 漫反射贴图与镜面光贴图
&emsp;&emsp;两者都是纹理图像，在前文中材质的漫反射以及镜面光颜色在片段的任何位置都是一致的，为了能通过某种方式对物体的每个片段单独设置漫反射以及镜面光颜色。可用为其分配贴图并通过采样获取，同时由于环境光颜色在几乎所有情况下都等于漫反射颜色，所以可以将其省略：

```glsl
struct Material {
    sampler2D diffuse;
    sampler2D specular;
    float     shininess;
}; 
...
in vec2 TexCoords;
```

![container2](https://github.com/user-attachments/assets/3572e4c6-6a06-4b40-8958-b4afaf45f1bc)
> **图 1**：漫反射贴图

![container2_specular](https://github.com/user-attachments/assets/1a0fae33-4340-479e-bb1c-5a70f27d1632)
> **图 2**：镜面光贴图

然后有：
```glsl
// 环境光颜色与漫反射颜色一致
vec3 ambient = light.ambient * vec3(texture(material.diffuse, TexCoords));
// 漫反射
vec3 diffuse = light.diffuse * diff * vec3(texture(material.diffuse, TexCoords));
// 镜面光
vec3 specular = light.specular * spec * vec3(texture(material.specular, TexCoords));
FragColor = vec4(ambient + diffuse + specular, 1.0);
```

![materials_specular_map](https://github.com/user-attachments/assets/040d7e6d-4834-4349-aa63-8f5bce81a5a2)
> **图 3**：应用光照贴图的结果