---
title: "c++连接IIS部署的SignalR失败"
publishDate: 2022-03-27 19:26:00 +0800
date: 2022-03-27 19:14:08 +0800
categories: SignalR windows dotnet csharp cpp
position: problem
---

本地vs调试没有问题包括iis express，部署到服务器IIS上C++客户端就连接不上，js客户端连接没问题

---

<div id="toc"></div>

## 问题

本地vs调试没有问题包括iis express，部署到服务器IIS上就连接不上，js客户端连接没问题，只有C++客户端连不上。
vs版本：vs2019
IIS系统：win10

## 解决

在控制面板-程序-启用或关闭Windows功能中安装WebSocket功能
![安装WebSocket功能](/static/posts/2022/2022-04-27-c++连接IIS部署的SignalR失败-01.jpg)

## 原因

因为c++客户端采用WebSockets连接，iis未启用WebSockets功能。js客户端能连接是因为js客户端采用了三种连接方式WebSockets、Server-Sent Events、长轮询，如果WebSocket不可用时可以自动降级，而c++只支持WebSocket，失败就连不上了。

---

**参考资料**

- [开源SignalR-Client-CPP使用总结](https://blog.csdn.net/qq_22642239/article/details/114382710)
