---
title: "net6之后不能通过替换HttpContext.Request.Path实现转发到其他路由"
publishDate: 2024-02-01 00:26:00 +0800
date: 2024-02-01 00:14:08 +0800
categories: asp.net core dotnet csharp
position: problem
---

asp.net core 3.1的时候可以直接通过替换HttpContext.Request.Path实现转发到其他路由，此方式在更新到.net6后默认不再可行。

---

<div id="toc"></div>

## 问题

asp.net core 3.1的时候可以直接通过替换HttpContext.Request.Path实现转发到其他路由，此方式在更新到.net6后默认不再可行。

## 解决

.ne6之后需要修改路由不能使用app.MapControllers()而应该使用app.UseRouting()和app.UseEndpoints(endpoints => endpoints.MapControllers())。

```diff
app.UseMiddleware<RequestPathCheckMiddleware>();
--app.MapControllers();
++app.UseRouting();
++app.UseEndpoints(endpoints => endpoints.MapControllers());
```

## 原因

调试发现不管怎么设置HttpContext.Endpoint都已经计算完成了，且是原来的路由，所以修改无效。而使用后面的写法则可以再次计算Endpoint。

---

**参考资料**

- [Updating HttpContext.Request.Path inside middleware has no effect in selecting the endpoint for the request #49454](https://github.com/dotnet/aspnetcore/issues/49454)
