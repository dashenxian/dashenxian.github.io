---
title: "Visual Studio 调试技巧之即时窗口的妙用(转载)"
publishDate: 2020-12-02 19:26:00 +0800
date: 2020-12-02 19:14:08 +0800
categories: Visual Studio dotnet csharp
position: problem
---

[原文链接](https://www.cnblogs.com/willick/p/14071945.html)
在 Visual Studio 中有一个窗口叫 Immediate 窗口，中文版本应该叫即时窗口。默认会在你启动调试时在 VS 编辑器中弹出来。你也可以通过 Debug | Windows | Immediate 或者使用快捷键 Ctrl+Alt+I 手动把它调出来。

---

<div id="toc"></div>

在 Visual Studio 中有一个窗口叫 Immediate 窗口，中文版本应该叫即时窗口。默认会在你启动调试时在 VS 编辑器中弹出来。你也可以通过 Debug | Windows | Immediate 或者使用快捷键 Ctrl+Alt+I 手动把它调出来。

![](https://w-share.oss-cn-shanghai.aliyuncs.com/20201129223706.png)

这个窗口很实用，尤其是在调试的时候。下面总结几个即时窗口的实用技巧。

1. 临时运行C#代码
有时候你可能只想知道一句C#代码运行的结果，比如你突然想知道一个空数组调用Sum()方法会不会报错，或者想查看一下Math.PI的值。你不用傻傻地把测试代码写在项目里，设个断点，然后把项目跑起来查看。你可以在即时窗口中直接写C#代码，然后按回车即可。比如输入：

Console.WriteLine("Welcome!")
回车运行：



再如，你可以直接输入 Math.PI 等表达式和调用某些方法：



也可以用 VS 的另外一个窗口 View | Other Windows | C# Interactive 来实现个功能。如果只是为了临时运行 C# 代码块，则C# Interactive 会更好用些。两者使用有些区别，C# Interactive 打印内容需要手动调用 Console.Write 等方法：



2. 调试时调用任何方法
假如你正在调试一个方法，你临时测试一下这个方法对于不同的参数的执行过程或运行结果。比如对于这样一段代码：

class Program
{
    static void Main(string[] args)
    {
        var foo = new Foo();
        var result = foo.Add(1, 2, 3);
        Console.WriteLine(result);
    }
}

public class Foo
{
    public int Add(params int[] nums)
    {
        if (nums?.Length < 1)
            return 0;
        var result = 0;
        foreach (var n in nums)
        {
            // ...（其它代码）
            result += n;
        }
        return result;
    }
}
若想用不同的参数来测试foo.Add方法的运行情况，普通的做法是启动多次调试，每次调试都修改一下调用代码 foo.Add 的参数。使用即时窗口，你可以在方法调用处打个断点。然后在即时窗口编写调用代码，它会直接使用当前上下文进行调试。不需要中断 VS 调试再重新启动。



另外，在即时窗口可以调用私有方法，也就是说它不受方法的访问权限限制。



不过，在即时窗口编写调用私有方法的代码时是没有智能提示的。

3. 使方法执行不影响上下文
默认情况下，在即时窗口运行的代码，执行完后会对上下文产生副作用（Side Effect）。比如对于这样一段代码：

class Program
{
    static void Main(string[] args)
    {
        var foo = new Foo();
        Console.WriteLine();
    }
}

public class Foo
{
    public int Num { get; private set; }
    public int Increase()
    {
        return ++Num;
    }
}
在即时窗口中调用foo.Increase后，Num 的变化如下：



但很多时候我们只希望即使窗口只是临时运行一下调试代码，不想让它真修改上下文的状态。我们只需在表达式后面添加 , nse（no side effect 的简写）即可：



加上 nse 后，执行的那句代码相当于在一个沙箱中运行，和上下文互不干扰。

4. 访问特殊变量
Visual Studio 在调试过程中有一些特殊的变量，可以在即时窗口打印它们的值。这些特殊的变量以 $ 作为前缀，通过智能提示可以看到目前有三个这样的特殊变量：

$exception，当前的异常信息。有时候在调试时，你代码的 try/catch 语句没有给 catch 语句使用 Exception 参数，则可以在即使窗口使用该特殊变量打印异常信息。



$returnvalue，当前语句的返回值。有时候你在代码中调用了一个方法，但你并没有用一个变量来存储这个方法的返回值，而你在调试时又想知道它的返回值。此时你可以在方法执行处添加一个断点。当运行到该断点时，按 F10，然后在即时窗口可以通过 $returnvalue 打印该方法的返回值。



$user，可以用来获取当前登录操作系统的用户信息和当前运行的进程和线程信息。这个我也没用过，官方文档介绍也比较简单，也不知道这个特殊变量包含哪些成员。直接打印是这样的：



结束
本文分享的这几个即时窗口的技巧，在调试时很实用，在工作中我经常使用，希望它也可以帮助你提高开发效率。关于调试，VS 还有其它好用的工具或技巧，比如有一个 Watch（监视）窗口，如果调试时要频繁查看一个对象的值，使用监视窗口比即时窗口方便很多。

当然，还是希望大家自己去探索更多的技巧，以做到能更高效灵活地使用 VS 这个强大的编辑器。

作者：精致码农

出处：http://cnblogs.com/willick

联系：liam.wang@live.com

本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。如有问题或建议，请多多赐教，非常感谢。

---
