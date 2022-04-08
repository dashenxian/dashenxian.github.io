---
title: "HttpClient上传文件"
publishDate: 2020-11-21 19:26:00 +0800
date: 2020-11-21 19:14:08 +0800
categories: httpclient dotnet csharp
position: problem
---

有时候需要使用hpptclient来上传文件，以前不会，这里记录以下

---

<div id="toc"></div>

## 代码

```c#
var client = _httpClientFactory.CreateClient();
var filePath="c:\\1.txt";
var fileName = Path.GetFileName(filePath);
var content = new MultipartFormDataContent();
content.Add(new ByteArrayContent(System.IO.File.ReadAllBytes(filePath)), "file", fileName);
var resultContent = await (await client.PostAsync("http://www.123.com/upfile", content)).Content.ReadAsStringAsync();
```

_httpClientFactory是依赖注入的，需要添加依赖注册

```c#
services.AddHttpClient();
```

---

**参考资料**

- [C#使用HttpClient上传文件并附带其他参数](https://www.cnblogs.com/cplemom/p/11264040.html)
