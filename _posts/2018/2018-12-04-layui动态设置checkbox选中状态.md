---
title: "layui动态设置checkbox选中状态"
publishDate: 2018-12-04 14:26:00 +0800
date: 2018-12-04  14:14:08 +0800
categories: layui动态设置checkbox选中状态
position: problem
---

---

<div id="toc"></div>

今天在使用jquery动态设置layui的checkbox元素的选中状态时始终只能取消选中，却不能重新勾选，点击勾选则没有问题，代码如下

```js
if (value == "true") {
    $("#select1").attr("checked", "checked");
} else {
    $("#select1").removeAttr("checked");
}
```

百度很久终于找到一个可用的解决方案,使用prop代替attr，如下：

```js
if (value == "true") {
    //$id.attr("checked", "checked");
    $id.prop("checked", true);
} else {
    $id.prop("checked", false);
    //$id.removeAttr("checked");
}
```
