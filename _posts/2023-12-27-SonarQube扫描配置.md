---
title: "SonarQube扫描配置"
publishDate: 2023-12-27 00:26:00 +0800
date: 2023-12-27 00:14:08 +0800
categories: SonarQube windows dotnet csharp
position: problem
---

SonarQube扫描配置，需要先安装SonarQube服务

---

<div id="toc"></div>

## 配置

在项目目录中增加一个脚本文件scan.ps1,我用的是powershell格式，也可以用bat格式

```ps1
#安装扫描工具，只有第一次才需要这句
dotnet tool install --global dotnet-sonarscanner

dotnet sonarscanner begin /k:"GarendService" /d:sonar.host.url="http://localhost:9000" /d:sonar.login="squ_68c5d637b47414c157289d5555f6a32c06d6fd15" /d:sonar.scm.provider="svn"
dotnet build GarendService.sln
dotnet sonarscanner end /d:sonar.login="squ_68c5d637b47414c157289d5555f6a32c06d6fd15"
```

---

**参考资料**

- [SonarScanner for .NET](https://docs.sonarsource.com/sonarqube/latest/analyzing-source-code/scanners/sonarscanner-for-dotnet/)
