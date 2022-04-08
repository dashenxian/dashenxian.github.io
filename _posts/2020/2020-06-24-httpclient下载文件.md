---
title: "httpclient下载文件"
publishDate: 2020-06-24 19:26:00 +0800
date: 2020-06-24 19:14:08 +0800
categories: CSharp Http Web HttpClient
position: problem
---

httpclient下载文件

---

<div id="toc"></div>

## 上代码

```c#
private async Task<string> DownLoadFileAsync(string url)
{
    var handler = new HttpClientHandler() { UseCookies = true };
    var client = HttpClientFactory.Create(handler);
    client.DefaultRequestHeaders.Add("User-Agent", @"Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:57.0) Gecko/20100101 Firefox/57.0");//添加自主驱动 很重要，没细研究，确实添加上就能下载文件
    client.DefaultRequestHeaders.Add("Accept", @"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8");//添加数据格式
    byte[] bytes = null;
    bytes = await client.GetByteArrayAsync(url);

    var tplfile = $"c:/{Guid.NewGuid()}{Path.GetExtension(url)}";

    using (var fs = new System.IO.FileStream(tplfile, FileMode.CreateNew))
    {
        fs.Write(bytes, 0, bytes.Length);
        fs.Close();
    }
    return tplfile;
}
```

---
