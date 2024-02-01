---
title: "net6之后读取response.body方式"
publishDate: 2024-02-01 00:26:00 +0800
date: 2024-02-01 00:14:08 +0800
categories: asp.net core dotnet csharp
position: problem
---

asp.net core 3.1的时候可以直接通过StreamReader读取HttpContext.Response.Body，此方式在更新到.net6后不再可行。

---

<div id="toc"></div>

## 问题

asp.net core 3.1的时候可以直接通过StreamReader读取HttpContext.Response.Body，此方式在更新到.net6后不再可行。调试发现HttpContext.Response.Body的CanRead属性是false，是一个可写不可读的流。

## 解决

.ne6之后要读取body必须用一个内存流替换原来的流，读取完成后再替换回去。以下示例代码是在中间件中写的：

```c#
using (var memoryStream = new MemoryStream())
{
    context.Response.Body = memoryStream;
    await _next.Invoke(context);//_next是下一个中间件
    context.Response.Body = currentBody;
    memoryStream.Seek(0, SeekOrigin.Begin);
    var responseBody = await new StreamReader(memoryStream).ReadToEndAsync();
}
```

## 原因

asp.net core 3.1的时候可以直接通过StreamReader读取HttpContext.Response.Body，此方式在更新到.net6后不再可行。

---

**参考资料**

- [Trouble reading stream from HttpContext.Response.Body in an ActionFilterAttribute #487](https://github.com/dotnet/aspnetcore/issues/487)
