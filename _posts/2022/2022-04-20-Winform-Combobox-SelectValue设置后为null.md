---
title: "Winform-Combobox设置SelectValue无效"
publishDate: 2022-04-20 19:26:00 +0800
date: 2022-04-20 19:14:08 +0800
categories: winform windows dotnet csharp
position: problem
---

Winform控件Combobox设置SelectedValue、SelectedItem、SelectedIndex后，值仍然为null的问题。

---

<div id="toc"></div>

## 问题

Winform控件Combobox设置SelectValue后，值仍然为null的问题；

代码片段如下：

```c#
cmb.DisplayMember = "NAME";//cmb是一个combobox组件，是在TabPage上的一个控件，TabPage没有加到TabControl上
cmb.ValueMember = "CODE";
cmb.DataSource = dics;//dics是一个包含NAME和CODE属性的列表
if (dics.Any(d => d.CODE == "9527"))
{
    cmb.SelectedValue = "9527";//执行这句后发现SelectedValue仍然为null
}
```

## 解决

在设置SelectedValue前必须把combobox加到页面中去，否则设置选中项是无效的，甚至直接设置SelectedIndex还会报错。

```c#
tab1.Controls.Add(tabPage2);
```

如果你发现设置combobox的选中项无效时可以查看是否combobox没有在窗体的控件树中。

## 原因

正在探索……

---

**参考资料**

- [Winform动态增加ComboBox后SelectedValue无效的问题](https://blog.csdn.net/winnyrain/article/details/73331140)
