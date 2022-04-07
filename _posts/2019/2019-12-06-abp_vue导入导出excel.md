---
title: "abp_vue导入导出excel"
publishDate: 2019-12-06 19:33:39 +0800
date: 2019-12-06 19:33:39 +0800
categories: abp_vue导入导出excel
position: problem
---

后端abp，前端vue导入excel，开始准备用直接用npoi，觉得要写太多的代码，就算从以前的复制粘贴也麻烦，所以偷懒直接用别人的轮子[Magicodes.IE](https://github.com/xin-lai/Magicodes.IE)。这样可以节省很多工作，根据实体生成excel模板、支持枚举、导入时自动验证数据是否合法（必填、类型等）

---

<div id="toc"></div>

## Excel模板

要导入首先要有录入数据的excel模板，以前都是把模板做好，放到服务器上，给一个下载链接给用户下载，这里可以直接用对象动态生成模板。

```c#
    //ExcelAppService.cs
    /// <summary>
    /// 生成excel模板
    /// </summary>
    /// <typeparam name="T">模板内容实体</typeparam>
    /// <param name="fileName">下载文件名称</param>
    /// <returns>输出文件流</returns>
    internal async Task<FileContentResult> GetTemplate<T>(string fileName = "模板") where T : class, new()
    {
        byte[] fileBytes = await importer.GenerateTemplateBytes<T>();

        return new FileContentResult(fileBytes, System.Net.Mime.MediaTypeNames.Application.Octet)
        {
            FileDownloadName = $"{fileName}.xlsx"
        };
    }
```

importer是在构造函数中注入的IImporter类型,如果你使用注入需要先在module的Initialize()方法中注册。

```c#
//module.Initialize()方法
IocManager.Register<Magicodes.ExporterAndImporter.Core.IImporter, Magicodes.ExporterAndImporter.Excel.ExcelImporter>(DependencyLifeStyle.Transient);
```

你也可以直接使用

```c#
IImporter importer=new ExcelImporter()
```

生成模板就做完了，剩下的就是在需要下载的地方调用此方法，公开一个api接口就可以了

```c#
/// <summary>
/// 下载导入模板
/// </summary>
/// <returns></returns>
public async Task<ActionResult> GetTemplate()
{
    return await excelAppService.GetTemplate<XXXXImportExcelDto>();
}
```

XXXXImportExcelDto是导入的实体类型，具体定义方式可以见[https://github.com/xin-lai/Magicodes.IE](https://github.com/xin-lai/Magicodes.IE)

如果你用的abp官方提供的vue项目，使用的axios请求后端，也就是ajax请求，这个文件流是不会弹出保存文件框的，需要在axios请求后拦截文件流弹出下载框。找到src\lib\ajax.ts文件,修改ajax.interceptors.response方法，并添加一个downloadUrl方法,如果后端验证了权限（登录），那么这里下载请求会出错"Refused to display in a frame because it set 'X-Frame-Options' to 'sameorigin'"，要么取消后端权限（登录）验证，要么在url中把jwtToken带上，我选择带上token。

```diff
++  import AppConsts from './appconst'
++  import Util from './util'

ajax.interceptors.response.use((respon)=>{    
++    //拦截文件下载请求
++    if (respon.headers && (respon.headers['content-type'] === 'application/octet-stream')) {
++        downloadUrl(respon.request.responseURL)
++        respon.data='';
++        respon.headers['content-type'] = 'text/json'
++        return respon;
++   }
    return respon
},(error)=>{
    if(!!error.response&&!!error.response.data.error&&!!error.response.data.error.message&&error.response.data.error.details){
        vm.$Modal.error({title:error.response.data.error.message,content:error.response.data.error.details})
    }else if(!!error.response&&!!error.response.data.error&&!!error.response.data.error.message){
        vm.$Modal.error({title:window.abp.localization.localize("LoginFailed"),content:error.response.data.error.message})
    }else if(!error.response){
        vm.$Modal.error(window.abp.localization.localize('UnknownError'));
    }
    setTimeout(()=>{
       vm.$Message.destroy();
    },1000);
    return Promise.reject(error);
})
++const downloadUrl = url => {
++  var encryptedAuthToken = Util.abp.utils.getCookieValue(AppConsts.authorization.encrptedAuthTokenName);
++  let iframe = document.createElement('iframe')
++  iframe.style.display = 'none'
++  iframe.src = url+"?"+AppConsts.authorization.encrptedAuthTokenName + "=" + encodeURIComponent(encryptedAuthToken)
++  iframe.onload = function () {
++    document.body.removeChild(iframe)
++  }
++  document.body.appendChild(iframe)
++}
```

后端 Web.Host.Startup.AuthConfigurer类中的QueryStringTokenResolver
```diff
    private static Task QueryStringTokenResolver(MessageReceivedContext context)
    {
--        if (!context.HttpContext.Request.Path.HasValue ||
--            !context.HttpContext.Request.Path.Value.StartsWith("/signalr"))
++        if (!context.HttpContext.Request.Path.HasValue
++            || !(context.HttpContext.Request.Path.Value.StartsWith("/signalr")
++                || context.HttpContext.Request.Path.Value.Contains("/GetTemplate"))
++                )
        {
            // We are just looking for signalr clients
            return Task.CompletedTask;
        }

        var qsAuthToken = context.HttpContext.Request.Query["enc_auth_token"].FirstOrDefault();
        if (qsAuthToken == null)
        {
            // Cookie value does not matches to querystring value
            return Task.CompletedTask;
        }

        // Set auth token from cookie
        context.Token = SimpleStringCipher.Instance.Decrypt(qsAuthToken, AppConsts.DefaultPassPhrase);
        return Task.CompletedTask;
    }
```



## 导入excel

导入分为两步：上传excel文件和解析数据。由于没有找到一个一次能处理这两步的方法（因为需要指定解析后的类型，这是一个强类型参数），我采用的方式是：

1. 加一个自定义组件，主要用于上传，提供一个上传完成事件,在上传完成后触发事件并传入后台excel文件的名称，
2. 使用的地方绑定事件并把带着文件名请求后台，
3. 后台再调用通用方法的解析数据

### 定义组件

```html
<template>
  <div>
    <Upload
      :action="uploadURL"
      :on-success="onSuccess"
      accept=".xls, .xlsx"
      :show-upload-list="false"
    >
      <Button icon="android-add" type="primary">{{btnTitle}}</Button>
    </Upload>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Inject, Prop, Watch } from "vue-property-decorator";
import Util from "../../lib/util";
import AbpBase from "../../lib/abpbase";
import appconst from "../../lib/appconst";

@Component
export default class ImportExcel extends AbpBase {
  uploadURL =
    appconst.remoteServiceBaseUrl + "/api/services/app/Excel/UploadExcelFile";
  async onSuccess(response, file, fileList) {
      //上传完成触发事件uploadCompleted
    this.$emit("uploadCompleted", response.result);
  }
  /**按钮显示内容 */
  @Prop({ type: String, default: "" }) btnTitle: String;
}
</script>
<style lang="less" scoped>
</style>
```

### 后端接收文件方法

```c#
    //ExcelAppService.cs
    /// <summary>
    /// 接收上传文件方法
    /// </summary>
    /// <param name="file">文件内容</param>
    /// <returns>文件名称</returns>
    public async Task<string> UploadExcelFile(IFormFile file)
    {
        //FileDir是存储临时文件的目录，相对路径
        //private const string FileDir = "/File/ExcelTemp";
        string url = await WriteFile(file, FileDir);

        string fullpath = Path.GetFullPath($"{Environment.CurrentDirectory}" + url);

        return Path.GetFileName(url);
    }
    /// <summary>
    /// 写入文件
    /// </summary>
    /// <param name="avatar"></param>
    /// <param name="reDir"></param>
    /// <returns></returns>
    public async Task<string> WriteFile(IFormFile avatar, string reDir)
    {
        string reName = Guid.NewGuid() + Path.GetExtension(avatar.FileName);
        string dir = GetDirPath(reDir);
        string path = $"{dir}\\{reName}";
        Stream stream = avatar.OpenReadStream();
        using (FileStream fileStream = new FileStream(path, FileMode.Create))
        {
            await avatar.CopyToAsync(fileStream);
        }
        return $"{reDir}/{reName}";
    }
    public string GetDirPath(string reDir)
    {
        string dir = $"{Environment.CurrentDirectory}/{reDir}";
        if (!Directory.Exists(dir))
        {
            Directory.CreateDirectory(dir);
        }
        return Path.GetFullPath(dir);
    }
```

### 使用组件

```html
<template>
    <ImprotExcel @uploadCompleted="importExcel" :btnTitle="'导入excel'" ></ImprotExcel>
</template>
<script>
 async importExcel(fileName: string) {
     //请求后端api
    await this.$store.dispatch({
      type: "xxx/importExcel",
      data: { fileName: fileName, labId: this.labId }
    });
    (<any>this.$Message).success({ background: true, content: "导入成功" });
  }
</script>
```

### 后端解析文件方法

```c#
    /// <summary>
    /// 导入
    /// </summary>
    /// <param name="input">导入excel参数</param>
    /// <returns></returns>
    [HttpPost]
    public async Task ImportExcel(XXXImprotExcelInput input)
    {
        var data = await excelAppService.GetData<XXXImportExcelDto>(input.FileName);
        if (!data.Any())
        {
            return;
        }
        //你的逻辑
    }

    //XXXImprotExcelInput.cs
    /// <summary>
    /// 导入excel
    /// </summary>
    public class XXXImprotExcelInput
    {
        /// <summary>
        /// 上传的excel文件名称
        /// </summary>
        public string FileName { get; set; }
        //你的其他参数
    }
    //ExcelAppService.cs
    /// <summary>
    /// 解析excel数据
    /// </summary>
    /// <typeparam name="T">要解析的数据类型</typeparam>
    /// <param name="fileName">excel文件名称，不含路径</param>
    /// <returns></returns>
    internal async Task<IEnumerable<T>> GetData<T>(string fileName) where T : class, new()
    {
        var fullpath = GetFullPath(fileName);
        var result = await importer.Import<T>(fullpath);
        if (result.HasError)
        {
            var errFile = Path.GetFileNameWithoutExtension(fileName) + "_" + Path.GetExtension(fileName);
            //如果excel文件内容不符合要求（格式错误、必填数据未填、数据类型错误），则弹出错误提示并给出下载链接
            throw new UserFriendlyException("导入错误", GetErrorExcelDownLoadUrl(errFile));
        }
        return result.Data;
    }
    /// <summary>
    /// 下载excel文件
    /// </summary>
    /// <param name="fileName"></param>
    /// <returns></returns>
    [HttpGet]
    public async Task<FileContentResult> DownLoadFile(string fileName)
    {
        var fullPath = GetFullPath(fileName);
        byte[] fileBytes = await File.ReadAllBytesAsync(fullPath);
        return new FileContentResult(fileBytes, System.Net.Mime.MediaTypeNames.Application.Octet)
        {
            FileDownloadName = fileName
        };
    }
    /// <summary>
    /// 获取文件全路径
    /// </summary>
    /// <param name="fileName"></param>
    /// <returns></returns>
    private string GetFullPath(string fileName)
    {
        fileName = Path.GetFileName(fileName);
        var fullpath = Path.GetFullPath(Environment.CurrentDirectory.EnsureEndsWith('/') + FileDir.EnsureEndsWith('/') + fileName);
        return fullpath;
    }
    /// <summary>
    /// 获取excel下载链接
    /// </summary>
    /// <param name="fileName"></param>
    /// <returns></returns>
    private string GetErrorExcelDownLoadUrl(string fileName)
    {
        return $"请按照excel文件内的错误提示修改后再次导入，<a href='{GetHost()}/api/services/app/Excel/DownLoadFile?fileName={fileName}' target='_blank'>点击下载excel</a>"
            ;
    }
    /// <summary>
    /// 获取当前域名地址
    /// </summary>
    /// <returns></returns>
    private string GetHost()
    {
        var req = httpContextAccessor.HttpContext.Request;
        return $"{req.Scheme}://{req.Host}";
    }
```

---

**参考资料**

- [Magicodes.IE](https://github.com/xin-lai/Magicodes.IE)
