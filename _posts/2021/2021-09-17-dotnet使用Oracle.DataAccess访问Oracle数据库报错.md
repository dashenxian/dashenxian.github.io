---
title: "dotnet使用Oracle.DataAccess访问Oracle数据库报错"
publishDate: 2021-09-19 19:26:00 +0800
date: 2021-09-19 19:14:08 +0800
categories: Oracle windows dotnet csharp DataAccess
position: problem
---

在dotnet中使用oracle客户端访问oracle数据库提示Oracle.DataAccess版本不匹配。

---

<div id="toc"></div>

## 问题

在dotnet中使用oracle客户端访问oracle数据库提示Oracle.DataAccess版本不匹配。

环境说明：

1. 接手的电脑已经装过oracle客户端（Oracle.DataAccess版本是2.112.3.0）
2. 新接手的项目使用的是Oracle.DataAccess版本是2.112.1.0

然后按照网上搜的各种卸载安装，还是报错。


## 解决

1. 拉取项目代码，运行报错，提示Oracle.DataAccess版本不匹配，这时并不知道已经电脑已经安装Oracle.DataAccess-2.112.3.0。
2. 直接安装2.112.1.0版本Oracle客户端（先装32位，再装64位），仍然报错
3. 查看C:\Windows\assembly发现有两个版本的Oracle.DataAccess（2.112.3.0和2.112.1.0），右键卸载2.112.3.0，提示无法卸载
4. 网络搜索卸载oracle，这其中反复卸载安装oracle客户端N次。问题依旧，卸载不掉2.112.3.0。
5. 最终还是找到了C:\Windows\assembly\Oracle.DataAccess的卸载方法,以管理员身份打开Visual Studio的开发人员命令提示符，输入
`gacutil /u "Oracle.DataAccess,Version=2.112.3.0, Culture=neutral,PublicKeyToken=b03f5f7f11d50a3a`
6. 卸载之后，再运行项目，提示信息不再是版本冲突，而是找不到程序集了，说明引用的仍然是2.112.3.0
7. 在vs异常管理界面，开启全部异常，这次终于获得了一条非常重要的信息。在`C:\Windows\assembly\GAC_64\Policy.2.112.Oracle.DataAccess\2.112.3.0__89b483f429c47342\Policy.2.112.Oracle.DataAccess.config`中重定向的2.112.3.0版本Oracle.DataAccess不存在。
8. 打开`C:\Windows\assembly\GAC_64\Policy.2.112.Oracle.DataAccess\2.112.3.0__89b483f429c47342\Policy.2.112.Oracle.DataAccess.config`，看到配置中的newVersion="2.112.3.0"，所以不管你怎么卸载安装，最终都会指向2.112.3.0，就算2.112.3.0不存在了也只会报错。
9. 复制Policy.2.112.Oracle.DataAccess.config，修改newVersion="2.112.1.0"，再覆盖回去，因为这个文件无法直接编辑保存，只能先复制出来再覆盖回去。
10. 运行项目，成功！

## 原因

`C:\Windows\assembly\GAC_64\Policy.2.112.Oracle.DataAccess\2.112.3.0__89b483f429c47342\Policy.2.112.Oracle.DataAccess.config`中配置了指向的dll版本，所以无论怎么卸载、安装都没用。

---

## oracle卸载删除注册表脚本

```reg
Windows Registry Editor Version 5.00
 
[-HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE]
 
[-HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\ODP.NET]
 
[-HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\MenuOrder\StartMenu\Programs\Oracle - OraClient11g_home1]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\eventlog\Application\Oracle.VSSWriter.CD]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\OracleServices for MTS]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\Oracle.portal]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\OracleDBConsoleportal]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\services\OracleDBConsoleorcl]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\services\eventlog\Application\Oracle.VSSWriter.CD]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\OracleServices for MTS]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\Oracle.portal]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\OracleDBConsoleportal]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\services\eventlog\Application\Oracle.cd]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\services\OracleDBConsoleorcl]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Oracle11]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Oracle11\Performance]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Oracle11\Performance\KEY_OraDb11g_home1]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleDBConsolemain]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleDBConsolemain\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleDBConsolemain\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleDBConsoleoracl]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleDBConsoleoracl\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleDBConsoleoracl\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleJobSchedulerMAIN]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleJobSchedulerMAIN\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleJobSchedulerORACL]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleJobSchedulerORACL\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleMTSRecoveryService]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleMTSRecoveryService\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleMTSRecoveryService\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleOraDb11g_home1ClrAgent]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleOraDb11g_home1ClrAgent\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleOraDb11g_home1TNSListener]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleOraDb11g_home1TNSListener\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleOraDb11g_home1TNSListener\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleRemExecService]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleRemExecService\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleRemExecService\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleServiceMAIN]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleServiceMAIN\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleServiceMAIN\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleServiceORACL]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleServiceORACL\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleServiceORACL\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleVssWriterMAIN]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleVssWriterMAIN\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleVssWriterMAIN\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleVssWriterORACL]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleVssWriterORACL\Security]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\OracleVssWriterORACL\Enum]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\OracleServices for MTS]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\Oracle.main]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\Oracle.oracl]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\Oracle.VSSWriter.MAIN]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\Oracle.VSSWriter.ORACL]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\OracleDBConsolemain]
 
[-HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\OracleDBConsoleoracl]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData\CurVer]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData.1]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData\CurVer]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData.1]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About\CLSID]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About.1]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData\CurVer]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData.1]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleConfig.OracleConfig]
 
[-HKEY_CLASSES_ROOT\OracleConfig.OracleConfig\CurVer]
 
[-HKEY_CLASSES_ROOT\OracleConfig.OracleConfig.1]
 
[-HKEY_CLASSES_ROOT\OracleConfig.OracleConfig.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleDatabase.OracleDatabase]
 
[-HKEY_CLASSES_ROOT\OracleDatabase.OracleDatabase\CurVer]
 
[-HKEY_CLASSES_ROOT\OracleDatabase.OracleDatabase.1]
 
[-HKEY_CLASSES_ROOT\OracleDatabase.OracleDatabase.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleHome.OracleHome]
 
[-HKEY_CLASSES_ROOT\OracleHome.OracleHome\CurVer]
 
[-HKEY_CLASSES_ROOT\OracleHome.OracleHome.1]
 
[-HKEY_CLASSES_ROOT\OracleHome.OracleHome.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraServer]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraServer\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraServer\CurVer]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraServer.5]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraServer.5\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraSession]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraSession\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraSession\CurVer]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraSession.5]
 
[-HKEY_CLASSES_ROOT\OracleInProcServer.XOraSession.5\CLSID]
 
[-HKEY_CLASSES_ROOT\OracleProcess.OracleProcess]
 
[-HKEY_CLASSES_ROOT\OracleProcess.OracleProcess\CurVer]
 
[-HKEY_CLASSES_ROOT\OracleProcess.OracleProcess.1]
 
[-HKEY_CLASSES_ROOT\OracleProcess.OracleProcess.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORADC.ORADCCtrl.1]
 
[-HKEY_CLASSES_ROOT\ORADC.ORADCCtrl.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData\CurVer]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData.1]
 
[-HKEY_CLASSES_ROOT\ORAMMCCFG11.ComponentData.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData\CurVer]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData.1]
 
[-HKEY_CLASSES_ROOT\ORAMMCPMON11.ComponentData.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.ErrorLookup]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.ErrorLookup\CLSID]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.ErrorLookup\CurVer]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.ErrorLookup.1]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.ErrorLookup.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.Oracle]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.Oracle\CLSID]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.Oracle\CurVer]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.Oracle.1]
 
[-HKEY_CLASSES_ROOT\OraOLEDB.Oracle.1\CLSID]
 
[-HKEY_CLASSES_ROOT\OraPerfMon.OraPerfMon]
 
[-HKEY_CLASSES_ROOT\OraPerfMon.OraPerfMon\CurVer]
 
[-HKEY_CLASSES_ROOT\OraPerfMon.OraPerfMon.1]
 
[-HKEY_CLASSES_ROOT\OraPerfMon.OraPerfMon.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About\CLSID]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About.1]
 
[-HKEY_CLASSES_ROOT\ORCLMMC.About.1\CLSID]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData\CurVer]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData.1]
 
[-HKEY_CLASSES_ROOT\ORCLSSO.ComponentData.1\CLSID]
 
[-HKEY_CLASSES_ROOT\EnumOraHomes.EnumOraHomes]
 
[-HKEY_CLASSES_ROOT\EnumOraHomes.EnumOraHomes\CurVer]
 
[-HKEY_CLASSES_ROOT\EnumOraHomes.EnumOraHomes.1]
 
[-HKEY_CLASSES_ROOT\EnumOraHomes.EnumOraHomes.1\CLSID]
```

**参考资料**

- [项目篇--win10卸载C:\Windows\assembly下的程序集](https://blog.csdn.net/weixin_34218579/article/details/93674485)
- [完全卸载Oracle方法(超详细)](https://blog.csdn.net/Ninewind/article/details/89520400)
- [oracle卸载删除注册表脚本](https://cloud.tencent.com/developer/article/1563146)
