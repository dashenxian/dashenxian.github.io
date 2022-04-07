---
title: "SilverLight类似WinForm弹窗等待结果再继续执行"
publishDate: 2020-06-29 19:26:00 +0800
date: 2020-06-29 19:14:08 +0800
categories: CSharp Http Web SilverLight
position: problem
---

在开发SilverLight时，弹窗一直都是用的回调方式，比如需要用户确认才能继续操作的，如果有好几个确认步骤，这时候回调函数就比较深了，代码基本看不懂，可以使用TaskCompletionSource把事件改为异步等待方法，全部改成同步的写法，爽的飞起。

---

<div id="toc"></div>

## 关键代码

```c#
[Flags]
public enum MsgBoxButton
{
    Ok = 1,
    YesNo = 2,
    OkCancel = 4,
    YesNoCancel = 8,

    //图标
    IconInfo = 16,
    IconWarn = 32,
    IconQuestion = 64,
    IconError = 128,
}

public static Task<System.Windows.MessageBoxResult> ShowAsync(string message, string title, MsgBoxButton buttons)
{
    var taskResult = new TaskCompletionSource<System.Windows.MessageBoxResult>();
    MsgBoxWindow messageBox = new MsgBoxWindow();//这是一个ChildWindow，只是自定义了一些样式和加了一些按钮：Yes、no、OK等，仿照winform
    messageBox.generateButtons(buttons);
    messageBox.Title = string.IsNullOrEmpty(title) ? "系统提示" : title;
    messageBox.Message = message;
    messageBox.MessageTextBlock.Width = twidth;

    messageBox.Closed += (ss, ee) =>
    {
        //异步等待关键代码，只有SetResult后，await才会继续执行
        taskResult.SetResult(messageBox._msgBoxResult);//根据点击按钮转换成了System.Windows.MessageBoxResult枚举结果
    };
    messageBox.Show();
    return taskResult.Task;
}
//创建按钮时在点击按钮事件中设置对应的结果
private void createOkButton()
{
    if (_okButton != null) return;

    _okButton = new Button
    {
        Content = "确定",
        Width = 75,
        Margin = new Thickness(2)
    };
    _okButton.Click += (sender, args) => { this._msgBoxResult = MessageBoxResult.OK; DialogResult = true; };
}
```

## 这样使用

```c#
var result = await MsgBoxWindow.ShowAsync("点吧", "店不大", MsgBoxButton.YesNo);
MessageBox.Show(result.ToString());
var result2 = await MsgBoxWindow.ShowAsync("点吧2", "店不大2", MsgBoxButton.YesNo);
MessageBox.Show(result2.ToString());
```

## 再也不需要这样了

```c#
MsgBoxWindow.Show("点吧", "店不大",  MsgBoxButton.YesNo, rs => {
    MessageBox.Show(rs.ToString());
    MsgBoxWindow.Show("点吧2", "店不大2", MsgBoxButton.YesNo, rs2 =>
    {
        MessageBox.Show(rs2.ToString());
    });
});
```

---
**参考资料**

- [C# dotnet 使用 TaskCompletionSource 让事件转异步方法](https://blog.lindexi.com/post/C-dotnet-%E4%BD%BF%E7%94%A8-TaskCompletionSource-%E8%AE%A9%E4%BA%8B%E4%BB%B6%E8%BD%AC%E5%BC%82%E6%AD%A5%E6%96%B9%E6%B3%95.html)

