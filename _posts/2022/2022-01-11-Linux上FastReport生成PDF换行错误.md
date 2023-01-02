---
title: "Linux上FastReport生成PDF换行错误"
publishDate: 2022-01-11 19:26:00 +0800
date: 2022-01-11 19:14:08 +0800
categories: linux .net core Graphics DrawString
position: problem
---

在Linux上使用FastReport生成PDF出现文本内容换行无效，这是由于fastReport调用Graphics.DrawString绘制文字，而在Linux系统中Graphics.DrawString则调用mono实现的libgdiplus库，最终问题就是libgdiplus库实现有问题，这样不光FastReport有问题，凡是调用Graphics.DrawString绘制文字都会有问题。

---

<div id="toc"></div>

## 解决方法

需要自己使用Pango重新编译libgdiplus库

1. 安装依赖
`sudo apt-get install libgif-dev autoconf libtool automake build-essential gettext libglib2.0-dev libcairo2-dev libtiff-dev libexif-dev libpango1.0-dev`

2. 获取libgdiplus源码
`git clone https://github.com/mono/libgdiplus.git`

3. 编译

``` cmd
 ./autogen.sh --with-pango --prefix=/usr
 make
```

4. 安装或替换

`sudo make install`
或者把编译的`libgdiplus.so.0.0.0`文件复制到`/usr/lib`目录中，注意手动复制的可能需要修改文件权限`chmod 644 /usr/lib/libgdiplus.so.0.0.0`

5. 如果需要拷贝到其他系统使用，libgdiplus还有依赖其他库，可以通过`ldd usr/lib/libgdiplus.so.0.0.0`查看对应的依赖并安装，你也可以直接安装以下两个库就可以了

`apt-get install -y libcairo2 libpango1.0`

---

## 测试代码

```c#
using System.Drawing;
internal class Program
{

    static void Main(string[] args)
    {
        var width = 50;
        var height = 400;
        var pageBitmap = new Bitmap(width, height);
        var pageGraphics = Graphics.FromImage(pageBitmap);

        // Create font and brush.
        var drawFont = new Font("宋体", 11);
        var drawBrush = new SolidBrush(Color.Black);
        // Set format of string.
        var drawString = "这是一段测试文本";
        var drawFormat = new StringFormat();
        drawFormat.FormatFlags = 0;//StringFormatFlags.NoWrap;
        drawFormat.Trimming = StringTrimming.Word;
        // Draw string to screen.
        pageGraphics.DrawString(drawString, drawFont, drawBrush, new Rectangle(0, 0, 50, 400), drawFormat);
        pageBitmap.Save("a.jpg");
    }
}
```

### 在项目文件中添加System.Drawing.Common引用

```xml
  <ItemGroup>
    <PackageReference Include="System.Drawing.Common" Version="5.0.3" />
  </ItemGroup>
```

**参考资料**

- [Building libgdiplus library from source](https://www.fast-report.com/en/blog/show/Building-libgdiplus-library-from-source/)
