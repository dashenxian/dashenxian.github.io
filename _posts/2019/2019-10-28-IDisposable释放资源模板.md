---
title: "IDisposable释放资源模板"
publishDate: 2019-10-28 19:46:53 +0800
date: 2019-10-28 19:46:53 +0800
categories: IDisposable释放资源模板
position: template
---

---

<div id="toc"></div>

```c#
public class FatherClass : IDisposable
{
    private bool isDisposed = false;
    public void Dispose()
    {
        Dispose(true); // 通知 GC，这个对象已经完全被清理。 
        GC.SuppressFinalize(this);
    }
    ~FatherClass()
    {
        Dispose(false);
    }
    protected virtual Dispose(bool isDisposing)
    {
        if (isDisposed) return;
        if (isDisposing)
        {
            // 释放托管资源。
        }
        // 释放非托管资源。 
        isDisposed = true;
    }
    public void TestMethod()
    {
        if (isDisposed)
        {
            throw new ObjectDisposedException("对象已经被释放。");
        }
    }
}
public class ChildClass : FatherClass
{
    private bool isDisposed = false;
    protected override void Dispose(bool isDisposing)
    {
        if (isDisposed) return; if (isDisposing)
        {
            // 释放托管资源。 
        }
        base.Dispose(isDisposing);
        isDisposed = true;
    }
}

```

**参考资料**