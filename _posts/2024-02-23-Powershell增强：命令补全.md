---
title: "Powershell增强：命令补全"
publishDate: 2024-02-23 00:26:00 +0800
date: 2024-02-23 00:14:08 +0800
categories: windows Powershell
position: problem
---


---

<div id="toc"></div>

## 安装插件

### 安装 PSReadLine

PSReadLine 提供了语法高亮、错误提示、多行编辑、键绑定、历史记录搜索等功能：

```cmd
Install-Module PSReadLine
```

### 安装 posh-git

posh-git 可以在 PowerShell 中显示 Git 状态信息，并提供 Git 命令的自动补全：

```cmd
Install-Module posh-git
```

## 配置插件

自定义配置
执行以下命令，第一次会显示找不到该文件，选择创建新文件：

```cmd
notepad $profile
```

作用是在 PowerShell 启动时运行一些自定义的设置，比如导入模块、设置别名、定义函数等。

粘贴以下配置内容，可以参考注释根据自己需求修改或者删除：

```txt

#------------------------------- Import Modules BEGIN -------------------------------
# 引入 ps-read-line
Import-Module PSReadLine

# 引入 posh-git
Import-Module posh-git

#------------------------------- Import Modules END   -------------------------------

#-------------------------------  Set Hot-keys BEGIN  -------------------------------
# 设置预测文本来源为历史记录
Set-PSReadLineOption -PredictionSource History

# 每次回溯输入历史，光标定位于输入内容末尾
Set-PSReadLineOption -HistorySearchCursorMovesToEnd

# 设置 Tab 为菜单补全和 Intellisense
Set-PSReadLineKeyHandler -Key "Tab" -Function MenuComplete

# 设置 Ctrl+d 为退出 PowerShell
Set-PSReadlineKeyHandler -Key "Ctrl+d" -Function ViExit

# 设置 Ctrl+z 为撤销
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo

# 设置向上键为后向搜索历史记录
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward

# 设置向下键为前向搜索历史纪录
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
#-------------------------------  Set Hot-keys END    -------------------------------
```

## 其他问题

### 解决警告: PowerShell 检测到你可能正在使用屏幕阅读器，并且已出于兼容性目的禁用 PSReadLine。如果要重新启用它，请运行 "Import-Module PSReadLine"

1. 创建一个.ps1文件（名字随便取，比如xiufu_powershell.ps1），粘贴以下代码：

```txt
Add-Type -TypeDefinition '
using System;
using System.ComponentModel;
using System.Runtime.InteropServices;
public static class ScreenReaderFixUtil
{
    public static bool IsScreenReaderActive()
    {
        var ptr = IntPtr.Zero;
        try
        {
            ptr = Marshal.AllocHGlobal(sizeof(int));
            int hr = Interop.SystemParametersInfo(
                Interop.SPI_GETSCREENREADER,
                sizeof(int),
                ptr,
                0);
            if (hr == 0)
            {
                throw new Win32Exception(Marshal.GetLastWin32Error());
            }
            return Marshal.ReadInt32(ptr) != 0;
        }
        finally
        {
            if (ptr != IntPtr.Zero)
            {
                Marshal.FreeHGlobal(ptr);
            }
        }
    }
    public static void SetScreenReaderActiveStatus(bool isActive)
    {
        int hr = Interop.SystemParametersInfo(
            Interop.SPI_SETSCREENREADER,
            isActive ? 1u : 0u,
            IntPtr.Zero,
            Interop.SPIF_SENDCHANGE);
        if (hr == 0)
        {
            throw new Win32Exception(Marshal.GetLastWin32Error());
        }
    }
    private static class Interop
    {
        public const int SPIF_SENDCHANGE = 0x0002;
        public const int SPI_GETSCREENREADER = 0x0046;
        public const int SPI_SETSCREENREADER = 0x0047;
        [DllImport("user32", SetLastError = true, CharSet = CharSet.Unicode)]
        public static extern int SystemParametersInfo(
            uint uiAction,
            uint uiParam,
            IntPtr pvParam,
            uint fWinIni);
    }
}'
if ([ScreenReaderFixUtil]::IsScreenReaderActive()) {
    [ScreenReaderFixUtil]::SetScreenReaderActiveStatus($false)
}
```

2.在PowerShell中运行这个文件
`
.\xiufu_powershell.ps1
`
然后重启PowerShell，就解决了。

### 解决“无法加载文件 ***\WindowsPowerShell\profile.ps1，因为在此系统上禁止运行脚本”

想了解计算机上的现用执行策略，打开 PowerShell 然后输入：

```cmd
>> get-executionpolicy
Restricted
```

更改执行策略，以管理员身份打开 PowerShell 输入：

```cmd
>> set-executionpolicy remotesigned
```

选择“是”，即可。

如果要更改回Windows 客户端计算机的默认执行策略，则设置为restricted：

```cmd
set-executionpolicy restricted
``


---

**参考资料**

- [Powershell增强：命令补全、主题美化及Git扩展保姆级教程](https://cloud.tencent.com/developer/article/2317806)
- [解决“无法加载文件 ***\WindowsPowerShell\profile.ps1，因为在此系统上禁止运行脚本”](https://zhuanlan.zhihu.com/p/452273123)

