---
title: "Windows Form定义快捷键（热键）"
publishDate: 2020-07-31 19:26:00 +0800
date: 2020-07-31 19:14:08 +0800
categories: v2rayN
position: problem
---

在windows form编程中定义快捷键（热键），通过重写WndProc消息方法捕获快捷键按下事件

---

<div id="toc"></div>

## 注册快捷键

在窗体加载方法中注册热键

```c#
WinAPIHelper.RegisterHotKey(this.Handle, 100, WinAPIHelper.KeyModifiers.Ctrl , Keys.F);
//WinAPIHelper.RegisterHotKey(this.Handle, 100, WinAPIHelper.KeyModifiers.Ctrl | WinAPIHelper.KeyModifiers.Alt, Keys.None); 只按下Ctrl+Alt触发键盘事件，比如你想实现按下ctrl+alt打开窗口
```

## 快捷键按下时触发事件

重写消息捕获方法拦截快捷键按下事件

```c#
protected override void WndProc(ref Message m)
{
    base.WndProc(ref m);
    Console.WriteLine(m.WParam);
    if (m.Msg == WinAPIHelper.WM_HOTKEY)
    {
        MessageBox.Show(m.WParam.ToString());
        if (m.WParam.ToInt32()==100)//m.WParam是WinAPIHelper.RegisterHotKey第二个参数的值
        {
            Console.WriteLine("你按下了Ctrl+Alt");
        }
    }
}
```

## WindowsAPI辅助类

```c#
public struct tagMSG
{
    public int hwnd;
    public uint message;
    public int wParam;
    public long lParam;
    public uint time;
    public POINT pt;
}

public struct POINT
{
    public int x;
    public int y;
}

public class WinAPIHelper
{
    #region 消息定义
    //标准
    public const int WM_CLOSE = 0x10;
    public const int WM_SETICON = 0x80;
    public const int WM_KEYDWON = 0x0100;
    public const int WM_HOTKEY = 0x312;
    public const int IMAGW_ICON = 1;
    public const int LR_LOADFROMFILE = 0x10;

    //自定义
    public const int WM_OPENDT = 0x8888;
    public const int WM_OPENHTWJ = 0x7777;
    public const int WM_OPENPROJ = 0x6666;
    public const int WM_CLOSEPROJ = 0x5555;
    public const int WM_REFRESHPROJ = 0x5511;
    public const int WM_DELENTITY = 0x4444;
    public const int WM_SHOWENTTIY = 0x4443;
    public const int WM_GETCGT = 0x3339;
    public const int WM_CGTEND = 0x3338;
    public const int WM_STRSTR = 0x3333;
    public const int WM_USER = 0x0400;
    public const int WM_UNDO = 0x2229;


    public const uint GW_CHILD = 5;
    public const int GWL_WNDPROC = -4;

    public struct SELFDEFINfO
    {
        public int nInt;
        public string strCon;
        public string objName;
        public object obj;
    }

    #endregion

    #region 消息循环
    [DllImport("user32", EntryPoint = "DispatchMessage")]
    public static extern int DispatchMessage(ref tagMSG lpMsg);



    [DllImport("user32", EntryPoint = "GetMessage")]
    public static extern int GetMessage(
            out tagMSG lpMsg,
            int hwnd,
            int wMsgFilterMin,
            int wMsgFilterMax
    );

    [DllImport("user32", EntryPoint = "TranslateMessage")]
    public static extern int TranslateMessage(ref tagMSG lpMsg);

    #endregion

    #region  WinAPI定义
    [DllImport("user32.dll", EntryPoint = "SendMessageA")]
    public static extern int SendMessage(IntPtr hwnd, int wMsg, int wParam, int lParam);
    [DllImport("User32.dll", EntryPoint = "FindWindow")]
    public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
    [DllImport("User32.dll ")]
    public static extern bool ShowWindow(IntPtr hWnd, int cmdShow);
    [DllImport("User32.dll ", EntryPoint = "SetParent")]
    public static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);


    [DllImport("user32", EntryPoint = "DispatchMessage")]
    public static extern int DispatchMessage(ref Message lpMsg);



    [DllImport("user32", EntryPoint = "GetMessage")]
    public static extern int GetMessage(
            out Message lpMsg,
            int hwnd,
            int wMsgFilterMin,
            int wMsgFilterMax
    );

    [DllImport("user32", EntryPoint = "TranslateMessage")]
    public static extern int TranslateMessage(ref Message lpMsg);

    /// <summary>
    /// 设置窗口显示图标
    /// </summary>
    /// <param name="hInst"></param>
    /// <param name="lpsz"></param>
    /// <param name="un1"></param>
    /// <param name="n1"></param>
    /// <param name="n2"></param>
    /// <param name="un2"></param>
    /// <returns></returns>
    [DllImport("user32", EntryPoint = "LoadImage")]
    public static extern int LoadImageA(int hInst, string lpsz, int un1, int n1, int n2, int un2);
    [DllImport("User32.dll", EntryPoint = "SendMessage")]
    public static extern int SendMessage(
        int hWnd, // handle to destination window 
        int Msg, // message 
        int wParam, // first message parameter 
        int lParam // second message parameter 
    );

    [DllImport("User32.dll", EntryPoint = "PostMessage")]
    public static extern int PostMessage(
        int hWnd, // handle to destination window 
        int Msg, // message 
        int wParam, // first message parameter 
        int lParam // second message parameter 
    );

    [DllImport("user32.dll", CharSet = CharSet.Auto)]
    public static extern int PostMessage(int hwnd, int msg, int wparam, IntPtr lParam);


    [DllImport("user32.dll", SetLastError = true)]
    public static extern bool PostThreadMessage(int threadId, uint msg, IntPtr wParam, IntPtr lParam);
    /// <summary>
    /// 向句柄发送消息（消息内容位对象）
    /// </summary>
    /// <param name="hwnd">句柄</param>
    /// <param name="msg">消息类型</param>
    /// <param name="wparam">参数</param>
    /// <param name="lParam">传递对象</param>
    /// 使用实例：WinAPIHelper.SELFDEFINfO INFO = new WinAPIHelper.SELFDEFINfO();
    ///           INFO.nInt = 5;
    ///           INFO.strCon = "1234";
    ///           INFO.obj = BussParaHelper._Chdata;
    ///           WinAPIHelper.PostMessage(BussParaHelper._ProjDetailView.GetHandle(), WinAPIHelper.WM_USER, 0, INFO);
    /// <returns></returns>
    public static int PostMessage(int hwnd, int msg, int wparam, object lParam)
    {
        System.Runtime.InteropServices.GCHandle h = System.Runtime.InteropServices.GCHandle.Alloc(lParam, System.Runtime.InteropServices.GCHandleType.WeakTrackResurrection);
        System.IntPtr addr = System.Runtime.InteropServices.GCHandle.ToIntPtr(h);

        return PostMessage(hwnd, msg, wparam, addr);
    }

    /// <summary>
    /// 根据消息获取实体
    /// </summary>
    /// <param name="msg">收到的消息</param>
    /// 使用实例： var obj23 = (WinAPIHelper.SELFDEFINfO)WinAPIHelper.GetObjectByMsg(m);
    ///            CHData jjj1 = obj23.obj as CHData; 
    /// <returns></returns>
    public static object GetObjectByMsg(Message msg)
    {
        System.Runtime.InteropServices.GCHandle handle2 = System.Runtime.InteropServices.GCHandle.FromIntPtr(msg.LParam);
        if (handle2 == null)
            return null;
        return handle2.Target;
    }

    [DllImport("user32.dll", EntryPoint = "SendMessageA")]
    public static extern int SendMessageA(IntPtr hwnd, int wMsg, IntPtr wParam, IntPtr lParam);

    /// <summary>
    /// 设置窗口显示文本
    /// </summary>
    /// <param name="hwnd"></param>
    /// <param name="lpString"></param>
    /// <returns></returns>
    [DllImport("user32", EntryPoint = "SetWindowText")]
    public static extern int SetWindowTextA(int hwnd, string lpString);
    #endregion

    /// <summary>
    /// 如果函数执行成功，返回值不为0。
    /// 如果函数执行失败，返回值为0。要得到扩展错误信息，调用GetLastError。
    /// </summary>
    /// <param name="hWnd">要定义热键的窗口的句柄</param>
    /// <param name="id">定义热键ID（不能与其它ID重复）</param>
    /// <param name="fsModifiers">标识热键是否在按Alt、Ctrl、Shift、Windows等键时才会生效</param>
    /// <param name="vk">定义热键的内容</param>
    /// <returns></returns>
    [DllImport("user32.dll", SetLastError = true)]
    public static extern bool RegisterHotKey(
        IntPtr hWnd,                //要定义热键的窗口的句柄
        int id,                     //定义热键ID（不能与其它ID重复）           
        KeyModifiers fsModifiers,   //标识热键是否在按Alt、Ctrl、Shift、Windows等键时才会生效
        Keys vk                     //定义热键的内容
        );
    /// <summary>
    /// 取消注册
    /// </summary>
    /// <param name="hWnd">要取消热键的窗口的句柄</param>
    /// <param name="id">要取消热键的ID</param>
    /// <returns></returns>
    [DllImport("user32.dll", SetLastError = true)]
    public static extern bool UnregisterHotKey(
        IntPtr hWnd,                //要取消热键的窗口的句柄
        int id                      //要取消热键的ID
    );

    //强制释放进程
    [DllImport("kernel32.dll", EntryPoint = "SetProcessWorkingSetSize")]
    public static extern int SetProcessWorkingSetSize(IntPtr process, int minSize, int maxSize);

    [DllImport("user32", EntryPoint = "HideCaret")]
    public static extern bool HideCaret(IntPtr hWnd);

    [System.Runtime.InteropServices.DllImport("user32.dll")]
    public static extern IntPtr GetWindow(IntPtr hWnd, uint uCmd);
    [System.Runtime.InteropServices.DllImport("user32.dll")]
    public static extern int SetWindowLong(IntPtr hWnd, int nIndex, int dwNewLong);
    [System.Runtime.InteropServices.DllImport("user32.dll")]
    public static extern int GetWindowLong(IntPtr hWnd, int nIndex);
    [System.Runtime.InteropServices.DllImport("User32.dll")]
    public static extern IntPtr CallWindowProc(IntPtr lpPrevWndFunc,
    IntPtr hWnd, int Msg, IntPtr wParam, IntPtr lParam);

    [DllImport("psapi.dll")]
    public static extern int EmptyWorkingSet(IntPtr hwProc);
    /// <summary>
    /// 定义了辅助键的名称（将数字转变为字符以便于记忆，也可去除此枚举而直接使用数值）
    /// </summary>
    [Flags]
    public enum KeyModifiers
    {
        None = 0,
        Alt = 1,
        Ctrl = 2,
        Shift = 4,
        WindowsKey = 8
    }

}
```


---
