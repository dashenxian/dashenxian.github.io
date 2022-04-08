---
title: "EF Core在日志中输出sql语句"
publishDate: 2020-08-28 19:26:00 +0800
date: 2020-08-28 19:14:08 +0800
categories: efcore asp.net dotnet csharp
position: problem
---

在我们使用ef core的时候，有时候我们需要查看ef core生成的sql语句以便我们可以做进一步的分析处理。

---

<div id="toc"></div>

## 修改日志级别输出日志

在配置文件appsettings.json中添加如下语句

```diff
 "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
++    "Microsoft.EntityFrameworkCore.Database": "Information"
    }
  }
```

这里相当于修改了ef core的默认输出日志级别，这样就会把sql输出，上面的配置对serilog依然有效

## 显示sql中的参数值

如果你的sql包含了参数，但是这样是不会输出参数值，参数值都是"?"，这是因为ef默认会对数据进行保护，如果需要显示参数值需要在DbContext中添加如下配置。

```diff
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
++  optionsBuilder.EnableSensitiveDataLogging();
    base.OnConfiguring(optionsBuilder);
}
```

---

**参考资料**

- [.NET Core实用技巧（一）如何将EF Core生成的SQL语句显示在控制台中](https://www.cnblogs.com/lwqlun/p/13551149.html)
