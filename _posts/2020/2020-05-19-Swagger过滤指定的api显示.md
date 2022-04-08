---
title: "Swagger过滤指定的api显示"
publishDate: 2020-05-19 19:26:00 +0800
date: 2020-05-19 19:14:08 +0800
categories: Swagger过滤指定的api显示
position: problem
---

在swagger显示的api文档中，隐藏指定的api

---

<div id="toc"></div>

## 实现过滤类

### Asp.Net版本

```c#
public class SwaggerDocumentFilter : Swashbuckle.Swagger.IDocumentFilter
{
    public void Apply(SwaggerDocument swaggerDoc, SchemaRegistry schemaRegistry, IApiExplorer apiExplorer)
    {
        // 骚操作之隐藏abp动态生成的api
        var apis = apiExplorer.ApiDescriptions.Where(x => x.RelativePath=="xxx");
        if (apis.Any())
        {
            foreach (var item in apis)
            {
                swaggerDoc.paths.Remove("/" + item.RelativePath);
            }
        }
    }
}
```

### Asp.Net Core版本

```c#
    public class SwaggerDocumentFilter : IDocumentFilter
    {
        public void Apply(OpenApiDocument swaggerDoc, DocumentFilterContext context)
        {
            var apis = context.ApiDescriptions.Where(x => x.RelativePath=="xxx");
            if (apis.Any())
            {
                foreach (var item in apis)
                {
                    swaggerDoc.Paths.Remove("/" + item.RelativePath);
                }
            }
        }
    }

```

## 引用过滤器

### Asp.Net版本

```c#
GlobalConfiguration.Configuration
.EnableSwagger(c =>
{
    c.DocumentFilter<SwaggerDocumentFilter>();
})
.EnableSwaggerUi(c =>
{
});
```

### Asp.Net Core版本

```c#
services.AddSwaggerGen(options =>
{
    // 应用Controller的API文档描述信息
    options.DocumentFilter<SwaggerDocumentFilter>();
});

```

---
