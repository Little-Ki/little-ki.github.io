[原文](http://learnopengl.com/#!Lighting/Materials)

# 材质
> 材质（Material）表示了光线是如何与物体进行交互的，当描述一个表面时，我们可以分别为三个光照分量定义一个材质颜色(Material Color)：环境光照(Ambient Lighting)、漫反射光照(Diffuse Lighting)和镜面光照(Specular Lighting)：
```glsl
#version 330 core
struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
}; 
  
uniform Material material;
```
&emsp;&emsp; ambient材质向量定义了在环境光照下这个表面反射的是什么颜色，通常与表面的颜色相同。diffuse材质向量定义了在漫反射光照下表面的颜色。漫反射颜色（和环境光照一样）也被设置为我们期望的物体颜色。specular材质向量设置的是表面上镜面高光的颜色（或者甚至可能反映一个特定表面的颜色）。最后，shininess影响镜面高光的散射/半径。

![materials_real_world](https://github.com/user-attachments/assets/2968052d-c784-43dd-aee8-3b050b8a8b68)
> **图 1**：不同材质属性对光照结果的影响

# 应用材质
```glsl
...
void main()
{    
    // 环境光
    vec3 ambient = lightColor * material.ambient;

    // 漫反射 
    vec3 norm = normalize(Normal);
    vec3 lightDir = normalize(lightPos - FragPos);
    float diff = max(dot(norm, lightDir), 0.0);
    vec3 diffuse = lightColor * (diff * material.diffuse);

    // 镜面光
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir, norm);  
    float spec = pow(max(dot(viewDir, reflectDir), 0.0), material.shininess);
    vec3 specular = lightColor * (spec * material.specular);  

    vec3 result = ambient + diffuse + specular;
    FragColor = vec4(result, 1.0);
}
```

![materials_with_material](https://github.com/user-attachments/assets/52f05969-3645-42de-b3de-c257a11fb775)
> **图 2**：类似过曝的效果

&emsp;&emsp;由于结果由三种类型光线相加，可能使颜色分量大于1.0产生类似过曝的效果，解决方法之一是调整lightColor的值，如 vec3(0.1)来降低亮度

# 光的属性
&emsp;&emsp;同样地还可以给光线分配类似的属性，使之在不同类型的光线模型上产生不同的结果，对光线进行更细致的调整：
```glsl
struct Light {
    vec3 position;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

uniform Light light;
```
&emsp;&emsp;前一个代码段中的lightColor替换为对应的属性：
```glsl
vec3 ambient  = light.ambient * material.ambient;
vec3 diffuse  = light.diffuse * (diff * material.diffuse);
vec3 specular = light.specular * (spec * material.specular);
```
