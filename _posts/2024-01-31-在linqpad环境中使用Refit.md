---
title: "在linqpad环境中使用Refit"
publishDate: 2024-01-31 00:26:00 +0800
date: 2024-01-31 00:14:08 +0800
categories: refit linqpad dotnet csharp
position: problem
---

在linqpad中使用Refit报错.

---

<div id="toc"></div>

## 问题

在linqpad中使用Refit报错:InvalidOperationException: xxx doesn't look like a Refit interface. Make sure it has at least one method with a Refit HTTP method attribute and Refit is installed in the project.

## 解决

1. nuget引用Castle.Core包
2. 添加一个代理生成类

```c#
public class ProxyRestService
{
	static readonly ProxyGenerator Generator = new ProxyGenerator();

	public static T For<T>(HttpClient client)
		where T : class
	{
		if (!typeof(T).IsInterface)
		{
			throw new InvalidOperationException("T must be an interface.");
		}

		var interceptor = new RestMethodInterceptor<T>(client);
		return Generator.CreateInterfaceProxyWithoutTarget<T>(interceptor);
	}

	public static T For<T>(string hostUrl)
		where T : class
	{
		var client = new HttpClient() { BaseAddress = new Uri(hostUrl) };
		return For<T>(client);
	}

	class RestMethodInterceptor<T> : IInterceptor
	{
		static readonly Dictionary<string, Func<HttpClient, object[], object>> methodImpls
			= typeof(T).GetMethods().Select(m => m.Name) //Refit 版本4+
			//RequestBuilder.ForType<T>().InterfaceHttpMethods //Refit 版本4-
				.ToDictionary(k => k, v => RequestBuilder.ForType<T>().BuildRestResultFuncForMethod(v));

		readonly HttpClient client;

		public RestMethodInterceptor(HttpClient client)
		{
			this.client = client;
		}

		public void Intercept(IInvocation invocation)
		{
			if (!methodImpls.ContainsKey(invocation.Method.Name))
			{
				throw new NotImplementedException();
			}
			invocation.ReturnValue = methodImpls[invocation.Method.Name](client, invocation.Arguments);
		}
	}
}
```

3. 调用的地方改为使用代理类

```diff
--    var gitApi = RestService.For<IGitHubApi>(url);
++    var gitApi = ProxyRestService.For<IGitHubApi>(url);
```

## 原因

在linqpad中由于不支持source generator，而Refit需要通过source generator来生成代理接口的实现，造成不能使用

---

**参考资料**

- [A polyfill for Refit 2.0+ in scripting environments (e.g. LINQPad). Warning: This won't work on all platforms Refit 2.0+ supports (e.g. iOS, maybe WinRT?).](https://gist.github.com/bennor/c73870e810f8245b2b1d)
- [Source Generator Support?](https://forum.linqpad.net/discussion/2396/source-generator-support)
