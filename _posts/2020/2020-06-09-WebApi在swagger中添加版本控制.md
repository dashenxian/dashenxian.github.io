---
title: "Asp.Net WebApi在swagger中添加版本控制"
publishDate: 2020-06-09 19:26:00 +0800
date: 2020-06-09 19:14:08 +0800
categories: Asp.Net  dotnet csharp
position: problem
---

在Asp.Net WebApi中添加版本控制，同时在swagger中按版本显示接口

---

<div id="toc"></div>

## 引用版本控制包

```xml
<package id="Microsoft.AspNet.WebApi.Versioning" version="4.0.0" targetFramework="net46" />
<package id="Microsoft.AspNet.WebApi.Versioning.ApiExplorer" version="4.0.0" targetFramework="net46" />
```

## 添加版本控制代码

按如下修改App_Start中的WebApiConfig文件

```c#
public static void Register(HttpConfiguration config)
{
    config.Filters.Add(new ApiExceptionFilter());
    config.MessageHandlers.Add(new WrapperHandler());
    // Web API 配置和服务

    config.AddApiVersioning(o =>
    {
        o.AssumeDefaultVersionWhenUnspecified = true;//没有标记版本的action默认未1.0版本

        o.ReportApiVersions = true;//返回版本可使用的版本
        o.ApiVersionReader = ApiVersionReader.Combine(new HeaderApiVersionReader("api-version"), new QueryStringApiVersionReader("api-version"));//通过Header或QueryString进行传值来判断api的版本
        o.DefaultApiVersion = new ApiVersion(1, 0);//默认版本号 
    });

    var apiExplorer = config.AddVersionedApiExplorer(
            options =>
            {
                options.GroupNameFormat = "'v'VVV";

                // note: this option is only necessary when versioning by url segment. the SubstitutionFormat
                // can also be used to control the format of the API version in route templates
                options.SubstituteApiVersionInUrl = true;
            });

    // Web API 路由
    config.MapHttpAttributeRoutes();


    config.Routes.MapHttpRoute(
        name: "DefaultApi",
        routeTemplate: "api/{controller}/{action}/{id}",
        defaults: new { id = RouteParameter.Optional }
    );
//注意启用swagger的代码一定要放到路由之后
    SwaggerConfig.Register(config, apiExplorer);

}
```

## 引用swagger包

```xml
<package id="Swashbuckle" version="5.6.0" targetFramework="net46" />
<package id="Swashbuckle.Core" version="5.6.0" targetFramework="net46" />
```

## 修改swagger为多版本api

在引用swagger包后，会自动在App_Start添加一个SwaggerConfig文件，需要修改部分代码，如下：

```c#
//由自动注册改为手动注册swagger，因为版本控制需要Web.Http.Description.VersionedApiExplorer apiExplorer参数
//[assembly: PreApplicationStartMethod(typeof(SwaggerConfig), "Register")]

namespace WebApplication1
{
    public class SwaggerConfig
    {
        public static void Register(HttpConfiguration Configuration, Microsoft.Web.Http.Description.VersionedApiExplorer apiExplorer)
        {
            var thisAssembly = typeof(SwaggerConfig).Assembly;

            Configuration
            .EnableSwagger(c =>
            {
                c.ResolveConflictingActions(apiDescriptions => apiDescriptions.First());

                c.MultipleApiVersions(
                    (apiDescription, version) => apiDescription.GetGroupName() == version,
                    info =>
                    {
                        foreach (var group in apiExplorer.ApiDescriptions)
                        {
                            //如果出现中文乱码问题，可以用vs新建一个SwaggerConfig，把原来SwaggerConfig中的内容拷过去，再删除自动创建的SwaggerConfig文件，
                            var description = "A sample application with Swagger, Swashbuckle, and API versioning.";

                            if (group.IsDeprecated)
                            {
                                description += " This API version has been deprecated.";
                            }

                            info.Version(group.Name, $"Create Wordreprot API {group.ApiVersion}");
                        }
                    });


                //获取目录下的XML文件 显示注释等信息
                var basePath1 = Path.GetDirectoryName(System.AppDomain.CurrentDomain.BaseDirectory);//获取应用程序所在目录（绝对，不受工作目录(平台)影响，建议采用此方法获取路径）
                var xmlComments = Directory.GetFiles(basePath1, "*.xml", SearchOption.AllDirectories).ToList();

                foreach (var xmlComment in xmlComments)
                {
                    c.IncludeXmlComments(xmlComment);
                }
                #region MyRegion

                #endregion
                // 应用Controller的API文档描述信息
                //c.DocumentFilter<SwaggerDocumentFilter>();
            })
             .EnableSwaggerUi(
                swagger =>
                {
                    //显示api版本多个版本选择，选择版本后要切换失去选择焦点然后回车才会触发刷新，不然始终显示默认版本
                    swagger.EnableDiscoveryUrlSelector();
                }
             );
        }
    }
}

```

## 在controller中标记版本

现在可以再controller或者action上添加版本标记来标记版本了，如果没有标记的默认1.0，默认版本设置见代码

```c#
public class Controller1 : ApiController
{
    [ApiVersion("1.0")]
    public async Task<string> Get(){
        returt "1.0"
    }
    public async Task<string> Get2(){
        returt "1.0"
    }
}
public class Controller2 : ApiController
{
    [ApiVersion("2.0")]
    public async Task<string> Get(){
        returt "2.0"
    }
}
```

## 发送请求

在请求中带上版本号标记，如果没有带版本则默认1.0，请求可以通过query参数或者header方式，名称为api-version，这个名称是在前述代码中配置的

## 可能遇到的问题

1. swagger描述中的中文乱码，可以用vs新建一个SwaggerConfig，把原来SwaggerConfig中的内容拷过去，再删除自动创建的SwaggerConfig文件
2. 启动报错"This XML file does not appear to have any style information associated with it. The document tree is shown below."，这是注册swagger的顺序错误，要把SwaggerConfig.Register(config, apiExplorer);放到路由注册之后。
3. 选择api版本后swagger ui页面没有刷新，还是显示上一个版本，选择版本后需要失去焦点再回车，不然会弹出选择框继续选择

---

**参考资料**

- [aspnet-api-versioning-SwaggerWebApiSample](https://github.com/microsoft/aspnet-api-versioning/tree/master/samples/webapi/SwaggerWebApiSample)
- [Swagger UI 中文乱码解决](https://blog.csdn.net/snow_lovelife/article/details/78285070)
