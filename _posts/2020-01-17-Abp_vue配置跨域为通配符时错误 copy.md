---
title: "Abp_配置跨域为*通配符时报跨域请求错误"
publishDate: 2020-01-17 19:26:00 +0800
date: 2020-01-17 19:14:08 +0800
categories: Abp_配置跨域为*通配符时报跨域请求错误
position: problem
---

当配置跨域域名为*时，signalr链接报错

---

<div id="toc"></div>

## 问题

像这样配置：
![CorsOrigins](/static/posts/2020-01-17-Abp_配置跨域为通配符时错误-01.png)

然后在浏览器调试窗口中就会看到如下错误：
![错误](/static/posts/2020-01-17-Abp_配置跨域为通配符时错误-02.png)

## 解决

解决方式应该有两种：第一种是修改服务端跨域配置为自己判断；第二种是修改前端不使用cookie，但是这样会有新的问题就是无法支持负载均衡。我采用第一种，因为第二种要修改abp源码并自己发布js包。

1. 使用SetIsOriginAllowed代替WithOrigins中间件配置跨域
![SetIsOriginAllowed](/static/posts/2020-01-17-Abp_配置跨域为通配符时错误-03.png)
![SetIsOriginAllowed](/static/posts/2020-01-17-Abp_配置跨域为通配符时错误-04.png)

2. 修改SignalR连接设置，不使用cookie，这种方式我没有试，不知道是否可行

```js
var hubConnectionBuilder = new HubConnectionBuilder();
var hubConnection = hubConnectionBuilder.WithUrl("https://localhost:21021/Hub",options => {
    options.UseDefaultCredentials = false;
}).Build();
await hubConnection.StartAsync();
```


## 原因

大概是因为在同时使用*配置跨域和AllowCredentials时服务器会返回一个[警告信息，这个修改是从2.2版本开始的](https://trailheadtechnology.com/breaking-change-in-aspnetcore-2-2-for-signalr-and-cors/)，操作signalR识别不了，然后就不进行链接请求了。

---

**参考资料**

- [Breaking change in AspNetCore 2.2 for SignalR and CORS](https://trailheadtechnology.com/breaking-change-in-aspnetcore-2-2-for-signalr-and-cors/)
- [Setting XMLHttpRequest "withCredentials" attribute #2110](https://github.com/aspnet/SignalR/issues/2110)
- [SignalR: Configuring CORS #6078](https://github.com/aspnet/AspNetCore.Docs/issues/6078)
- [Setting XHR "withCredentials" attribute in SignalR #14570](https://github.com/dotnet/aspnetcore/issues/14570)