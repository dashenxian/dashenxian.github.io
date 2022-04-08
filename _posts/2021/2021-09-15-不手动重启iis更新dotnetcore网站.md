---
title: "不手动重启iis更新dotnetcore网站"
publishDate: 2021-09-15 19:26:00 +0800
date: 2021-09-15 19:14:08 +0800
categories: dotnet core windows csharp ftp iis
position: problem
---

由于没有服务器权限，只有一个ftp目录权限，ftp上传网站文件后（虽然提示上传成功，但是找管理员手动重启iis后程序还是没更新，估计文件被占用应该是没上传成功的），网站程序还是以前的。

---

<div id="toc"></div>

## 步骤

1. 先上传一个名字叫app_offline.htm的文件
2. 再上传正式的网站程序文件
3. 删除app_offline.htm

总结：更新前在根目录下放个名字叫app_offline.htm的文件，更新完删掉。

---

**参考资料**

- [解决ASP.NET Core部署到IIS，更新项目"另一个程序正在使用此文件，进程无法访问"](https://www.cnblogs.com/zhangchunyu/p/11066078.html)
- [App Offline file (app_offline.htm)](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/app-offline?view=aspnetcore-5.0)
