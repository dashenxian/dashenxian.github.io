---
title: "abp替换IRepository默认注入类型"
publishDate: 2019-12-9 19:01:35 +0800
date: 2019-12-9 19:01:35 +0800
categories: abp替换IRepository默认注入类型
position: problem
---

如果要想重写部分方法，但是又没有实现自定义仓储，就必须从项目的抽象仓储入手，就是继承自EfCoreRepositoryBase的XXXRepositoryBase<TEntity, TPrimaryKey>和XXXRepositoryBase<TEntity>。

---

<div id="toc"></div>

## 实现方法

1. 去掉abstract关键字，把XXXRepositoryBase<TEntity, TPrimaryKey>和XXXRepositoryBase<TEntity>改为非抽象类
2. 修改构造函数的访问级别为pulic，否则依赖注入无法实例化对象
3. 在ef上下文类XXXDbContext增加特性

```c#  
   [AutoRepositoryTypes(
   typeof(IRepository<>),
   typeof(IRepository<,>),
   typeof(HItekLabRepositoryBase<>),
   typeof(HItekLabRepositoryBase<,>)
)]
 public class XXXDbContext : AbpZeroDbContext<Tenant, Role, User, XXXDbContext>{
     //...
 }
```

皮卡丘，就决定用你了。

---

**参考资料**

- [ABP官方文档翻译 9.2 Entity Framework Core](https://www.cnblogs.com/xajh/p/7153017.html#DefaultRepositories)
- [AspnetBoilerplate modular: The entity type XXX is not part of the model for the current context](https://stackoverflow.com/questions/46298898/aspnetboilerplate-modular-the-entity-type-xxx-is-not-part-of-the-model-for-the)
- [Custom repository exception](https://support.aspnetzero.com/QA/Questions/4185)
