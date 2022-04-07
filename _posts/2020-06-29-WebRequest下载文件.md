---
title: "WebRequest下载文件"
publishDate: 2020-06-29 19:26:00 +0800
date: 2020-06-29 19:14:08 +0800
categories: CSharp Http Web WebRequest
position: problem
---

WebRequest下载文件

---

<div id="toc"></div>

## 上代码

```c#
private void DownloadFile(string fileUrl, string filename)
{
    try
    {
        var request = WebRequest.Create(fileUrl);
        var response = request.GetResponse();

        using (var responseStream = response.GetResponseStream())
        {
            var readedSize = 0L;

            using (var fs = new FileStream(filename, FileMode.Create))
            {
                var buffer = new byte[1024 * 500];
                var readSize = responseStream.Read(buffer, 0, buffer.Length);

                while (readSize > 0)
                {
                    fs.Write(buffer, 0, readSize);
                    readSize = responseStream.Read(buffer, 0, buffer.Length);

                    readedSize += readSize;
                }
            }
        }

        response.Close();
    }
    catch (WebException ex)
    {
        var rsp = ex.Response as HttpWebResponse;

        if (rsp != null && rsp.StatusCode == HttpStatusCode.NotFound)
        {
            throw new FileNotFoundException(fileUrl);
        }
        else
        {
            throw ex;
        }
    }
}
```

---
