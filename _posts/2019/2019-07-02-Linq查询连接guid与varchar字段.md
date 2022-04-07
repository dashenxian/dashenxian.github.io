---
title: "Linq查询连接guid与varchar字段"
publishDate: 2019-07-02 14:26:00 +0800
date: 2019-07-02 14:14:08 +0800
categories: Linq查询连接guid与varchar字段
position: problem
---

---

<div id="toc"></div>

## 使用场景

在数据库设计中进场会出现一些通用表，如通用附件表，一般都是通过ForeignTable（关联的表名）和ForeignKey（关联表的主键）与其他表关联。这样的表在数据库中没有外键关系，而且一般ForeignKey的类型是varchar，为了兼容其他表的主键可能不一样。这样在Linq查询的时候就不能直接关联了，如下代码会编译不通过：

```C#
from a in db.WorkflowInstance
join b in d.xxx//xxx.ID为guid类型
on new { a.ForeignTable, a.ForeignKey } equals new { ForeignTable = nameof(xxx), ForeignKey = b.ID }
select a;
```

因为xxx.id是Guid(uniqueidentifier)类型和WorkflowInstance.ForeignKey是string(varchar)类型。就算是强行把xxx.id转成string类型，编译通过了运行也会报错，如下：

```C#
from a in db.WorkflowInstance
join b in d.xxx//xxx.ID为guid类型
on new { a.ForeignTable, a.ForeignKey } equals new { ForeignTable = nameof(xxx), ForeignKey = b.ID+"" }
select a;
```

以为这段代码最终都会转成sql语句，而Guid是不能直接转换成varchar的。

## 解决方案

如果xxx.id是数字类型(int,float,double,decimal)是可以使用SqlFunctions.StringConvert(xxx.id)转换成string类型，这样就可以了,SqlFunctions.StringConvert支持double和decimal，基本上数字都可以转换成这两种类型，但是注意下转换时小数点后0的个数，因为string比较时少一个0是不一样的。
但是Guid不行，因为没有对应的函数。通过面向百度编程，微软爸爸给我们提供了一个解决方案：自定义函数。相当于我们自己实现一个SqlFunctions.StringConvert()。

### 首先在数据库定义一个转换函数

```sql
if EXISTS(select * from dbo.sysobjects where id = object_id(N'[dbo].[ConvertGuidToChar]') and xtype in (N'FN', N'IF', N'TF'))
drop function [dbo].ConvertGuidToChar
GO
CREATE FUNCTION ConvertGuidToChar
(
	@id UNIQUEIDENTIFIER
)
RETURNS VARCHAR(50)
AS
BEGIN
    RETURN CONVERT(VARCHAR(50),@id)
END
```

### 把函数添加到db模型

可以直接编辑edmx模型文件添加如下代码：

```xml
<Function Name="ConvertGuidToChar" ReturnType="varchar" Schema="dbo" >
  <Parameter Name="id" Mode="In" Type="uniqueidentifier" />
</Function>
```

也可通过从数据库更新模型添加
![](https://img2018.cnblogs.com/blog/208398/201907/208398-20190702193416259-1788815274.png)

### 添加自定义函数对应的方法

```c#
/// <summary>
/// sql函数Guid转varchar
/// </summary>
/// <param name="id"></param>
/// <returns></returns>
[EdmFunction("iLISModel.Store", "ConvertGuidToChar")]
public static string ConvertGuidToChar(Guid id)
{
    throw new NotSupportedException("Direct calls are not supported.");
}
```

### Linq中使用自定义函数转换类型

```c#
from a in d.WorkflowInstance
join b in d.xxx//xxx.ID为guid类型
on new { a.ForeignTable, a.ForeignKey } equals new { ForeignTable = nameof(xxx), ForeignKey =  SqlFunctionsExtension.ConvertGuidToChar(b.ID) }
select a;
```

这样就能正常查询数据了。
注：codefirst是没有edmx模型的，但是应该可以通过[其他方式](https://github.com/moozzyk/CodeFirstFunctions)添加，我没试，我随便说的，你别信啊。

**参考资料**
[如何：调用自定义数据库函数](https://docs.microsoft.com/zh-cn/dotnet/framework/data/adonet/ef/language-reference/how-to-call-custom-database-functions)
