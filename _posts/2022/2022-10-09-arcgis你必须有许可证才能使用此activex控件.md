---
title: "arcgis你必须有许可证才能使用此activex控件"
publishDate: 2022-10-09 00:26:00 +0800
date: 2022-10-09 00:14:08 +0800
categories: winform windows dotnet csharp arcgis
position: problem
---

在开发arcgis时把版本从10.6将为10.1后报错

---

<div id="toc"></div>

## 问题

卸载10.6版本的arcgis套件，重新安装10.1后，同样的代码报错授权不行，但是arcgis map能正常使用，说明授权是成功了的。

## 解决

在main函数种去掉授权的代码

```c#
ESRI.ArcGIS.RuntimeManager.Bind(ESRI.ArcGIS.ProductCode.EngineOrDesktop);
//这是10.6授权需要添加的，10.1要去掉
//IAoInitialize aoInit = new AoInitializeClass();
//aoInit.Initialize(esriLicenseProductCode.esriLicenseProductCodeArcServer);
```

## 原因

10.6和10.1授权代码有一点区别

---

**参考资料**
