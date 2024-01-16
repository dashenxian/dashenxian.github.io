---
title: "2024-01-16-CityEngine导出glb、gltf设置透明模式"
publishDate: 2024-01-16 00:26:00 +0800
date: 2024-01-16 00:14:08 +0800
categories: cityengine windows dotnet csharp
position: problem
---

CityEngine默认导出glb、gltf时是透明的，但是obj不是透明的，在blender中查看会发现glb、gltf模型能透视，如果模型是房子这是不合理的

---

<div id="toc"></div>

## 问题

CityEngine默认导出glb、gltf时是透明的，但是obj不是透明的，在blender中查看会发现glb、gltf模型能透视，如果模型是房子这是不合理的

## 解决

在cga中有material.opacitymap.mode属性，这个属性对应gltf模型alphaMode属性。material.opacitymap.mode有三个可选值"blend"、"mask" 、 "opaque"，注意`必须是小写`。在任意cga方法中设置`set(material.opacitymap.mode,"opaque")`就可以把材质设置为不透明，设置对当前方法作用域有效，即从当前方法开始内部执行的逻辑都会有效，含子方法，所以如果要设置全局的，可以把这句写在根方法中。

## 原因

查阅gltf规范指定，gltf模型资源是否透明受materials节点下alphaMode属性控制，有三个可选模式,`注意alphaMode的值必须是大写才有效`

```json
{
    "materials": [
        {
            "name": "Material0",
            "pbrMetallicRoughness": {
                "baseColorFactor": [ 0.5, 0.5, 0.5, 1.0 ],
                "metallicFactor": 1,
                "roughnessFactor": 1,
            },
            "doubleSided": true,
            "alphaMode": "OPAQUE"
            "emissiveFactor": [ 0.2, 0.1, 0.0 ]
        }
    ]
}
```

* “OPAQUE”:默认模式，完全不透明，忽略任何的alpha值
* “MASK”:与另一个属性alphaCutoff，如果小于alphaCutoff的值则为完全透明，否则为完全不透明。alphaCutoff值只有在"MASK"模式中生效，其他模式忽略该值。
* “BLEND”:混合模式，该模式的显示效果取决于各个引擎对该属性的支持。

---

**参考资料**

- [glTF全解析——materials](https://blog.csdn.net/qq_34205932/article/details/97151022)
- [导出 glTF/glb (Khronos Group)](https://doc.arcgis.com/zh-cn/cityengine/latest/help/help-export-gltf.htm)
- [材料形状属性](https://doc.arcgis.com/zh-cn/cityengine/latest/cga/cga-material-attribute.htm)
