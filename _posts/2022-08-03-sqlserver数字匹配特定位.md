---
title: "sqlserver数字匹配特定位"
publishDate: 2022-08-03 19:26:00 +0800
date: 2022-08-03 19:14:08 +0800
categories: sqlserver windows dotnet csharp
position: problem
---

由于某种原因需要在sqlserver数据库中查找千分位上为5的所有数据，如：xxx.xx5xx

---

<div id="toc"></div>

## 问题

由于某种原因需要在sqlserver数据库中查找千分位上为5的所有数据，如：xxx.xx5xx

## 解决

数据查找可以通配符+占位符实现，如：

```sql
SELECT * FROM [dbo].[tbl_test] WHERE [mj] LIKE '%.__5%'
```

但是我们的mj字段的类型是real，直接匹配不了必须是字符串才能匹配，需要数据转换。直接是用converter函数转换发现精度会丢失，本来是4位小数转换后全部成了3位。
经过研究（百度）要先转换成money，再转换成varchar，再匹配。

最终sql如下：

```sql
SELECT * FROM [dbo].[tbl_test] WHERE Convert(varchar(30), CAST(mj AS money ),2) LIKE '%.__5%'
```

![一天工资到手，关机下班](/static/common/offduty.gif)

---

**参考资料**

- [CAST 和 CONVERT (Transact-SQL)](https://docs.microsoft.com/zh-cn/sql/t-sql/functions/cast-and-convert-transact-sql?view=sql-server-ver16)
