---
title: "实现通用IEqualityComparer比较器"
publishDate: 2019-11-14 15:12:29 +0800
date: 2019-11-14 15:12:29 +0800
categories: 实现通用IEqualityComparer比较器
position: problem
---

在使用Linq做两个列表的集合运算时(交集、并集、差集)，如果不是使用对象比较而是要用部分属性比较，需要用到比较器(如果没有比较器则会直接比较对象引用)，但是(垃圾)微软没有提供默认的比较器。

---

<div id="toc"></div>

## 问题
每次比较一个类型都需要对类型实现比较器，有没有可以像写lambda表达式一样直接比较呢

## 解决

```c#
namespace System.Collections.Generic
{
    public class GenericCompare<T> : IEqualityComparer<T> where T : class
    {
        private Func<T, T, bool> Expr { get; set; }
        public GenericCompare(Func<T, T, bool> expr)
        {
            this.Expr = expr;
        }
        public bool Equals(T x, T y)
        {
            if (Expr(x, y))
                return true;
            else
                return false;
        }
        public int GetHashCode(T obj)
        {
            return 0;
        }
    }
}
```

## 使用方法

```c#
public class Test
{
    public void T1()
    {
        var ls1 = new List<Student> { new Student { Name="小明"},new Student { Name="小红"} };
        var ls2 = new List<Student> { new Student { Name = "小明" }, new Student { Name = "小钢" } };
        ls1.Except(ls2, new GenericCompare<Student>((x, y) => x.Name == y.Name));
    }
}
public class Student
{
    public string Name { get; set; }
}
```

---

