---
title: "从dotnet的resources中获取dll"
publishDate: 2024-01-17 00:26:00 +0800
date: 22024-01-17 00:14:08 +0800
categories: wpf windows dotnet csharp
position: problem
---

从dotnet的resources中获取dll

---

<div id="toc"></div>

从dotnet的resources中获取dll，支持将Fody/Costura嵌入式的资源文件还原

```c#
void Main()
{
	var path = @"";
	var assembly = Assembly.LoadFrom(path);
	var resources = assembly.GetManifestResourceNames();
	foreach (var element in resources)
	{
		using FileStream output = new FileStream($@"d:\{element.Replace(".compressed", "")}", FileMode.Create);
		using var sm = assembly.GetManifestResourceStream(element);
		//将Fody/Costura嵌入式的资源文件还原
		if (element.Contains("compressed"))
		{
			using DeflateStream source = new DeflateStream(sm, CompressionMode.Decompress);
			source.CopyTo(output);
		}
		else
		{
			sm.CopyTo(output);
		}
	}
}
```


---

**参考资料**

- [Fody-Costura解压缩,.dll.compresse还原为.dll](https://www.cnblogs.com/icejd/p/14738325.html)
