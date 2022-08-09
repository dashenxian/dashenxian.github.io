---
title: "AutoCAD-RibbonButton图标显示不全"
publishDate: 2022-08-09 19:26:00 +0800
date: 2022-08-09 19:14:08 +0800
categories: AutoCAD windows dotnet csharp
position: problem
---

AutoCAD添加RibbonButton时，图标显示不全。

---

<div id="toc"></div>

## 问题

AutoCAD添加RibbonButton时，图标显示不全，但是部分图标又能显示完整，像被裁剪了一部分。

## 解决

我的这个图标大小在4.3kb左右，在squoosh网站把图标压缩一下大小到1.7kb就能正常显示了。


## 原因

可能是AutoCAD对菜单图标大小有限制，超过3kb（没有尝试具体大小，有些图标2kb+也能正常显示）。

---

**参考资料**

