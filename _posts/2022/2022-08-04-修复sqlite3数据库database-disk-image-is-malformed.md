---
title: "修复sqlite3数据库database-disk-image-is-malformed错误"
publishDate: 2022-08-04 19:26:00 +0800
date: 2022-08-04 19:14:08 +0800
categories: wpf windows dotnet csharp
position: problem
---

不知什么原因，发现系统突然不能使用，使用工具连接数据库文件时提示“database-disk-image-is-malformed”错误。

---

<div id="toc"></div>

## 问题

不知什么原因，发现系统突然不能使用，使用工具连接数据库文件时提示“database-disk-image-is-malformed”错误。
能怎么办呢，直接开始解决（百度）。

## 解决

到C:\Program Files (x86)\Android\android-sdk\platform-tools目录下找到sqlite3.exe文件工具开始一顿如下操作。

1. 命令行打开sqlite3.exe并读取损坏的文件

```cmd
sqlite3.exe oldDB
```

此时进入了sqlite命令行环境
2. 导出sql语句到临时文件

```cmd
sqlite>.output tmp.sql
sqlite>.dump
sqlite>.quit
```

3. 修改tmp.sql文件
由于数据库文件损坏，所以sqlite自动将tmp.sql最后一行加上了一句Rollback，因此我们需要手动修改tmp.sql文件，将最后一行的Rollback改为Commit;。

4. 读取tmp.sql并写入到新库中

```cmd
>sqlite3.exe NewDB
sqlite>.read tmp.sql
sqlite>.quit
```

![一天工资到手，关机下班](/static/common/offduty.gif)

## 原因

正在探索……

---

**参考资料**

- [修复sqlite3数据库,database disk image is malformed](https://www.jianshu.com/p/d2c53d654e4a)
