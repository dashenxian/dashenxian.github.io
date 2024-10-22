---
title: "snap 安装的docker，如何添加加速镜像和重启服务"
publishDate: 2024-10-22 00:26:00 +0800
date: 2024-10-22 00:14:08 +0800
categories: docker ubantu snap
position: problem
---

在我们这里无法拉取docker镜像，一般可以通过设置国内镜像源/加速列表来拉取镜像，但是ubantu22开始建议用snap来安装docker，如果用snap安装镜像后，你会发现搜到的各种设置后还是拉取不了，因为snap安装的docker根本不读取那个配置了

---

<div id="toc"></div>

<b>通过 snap 安装的 Docker 需要特别的步骤来配置镜像地址。以下是具体的步骤：</b>

### 创建 Docker 配置文件目录

Snap 安装的 Docker 可能没有默认的配置文件目录，需要手动创建。
`sudo mkdir -p /var/snap/docker/current/config`

### 创建并编辑配置文件

在 `/var/snap/docker/current/config` 目录下创建 daemon.json 文件，并添加你的镜像地址。

`sudo vim /var/snap/docker/current/config/daemon.json`
将以下内容粘贴到文件中：

```json
{
  "registry-mirrors": [
    "https://xdark.top",
    "https://dockerproxy.cn",
    "https://docker.rainbond.cc",
    "https://docker.udayun.com",
    "https://docker.211678.top",
  ]
}
```

最新镜像地址：[https://xuanyuan.me/blog/archives/1154?from=tencent#_registry_mirror](https://xuanyuan.me/blog/archives/1154?from=tencent#_registry_mirror)

保存并退出编辑器（在 vim 中，按 ESC 然后 输入`:wq`保存）。

### 重启 Docker 服务

由于是通过 snap 安装的 Docker，需要使用 snap 命令重启服务。
`sudo snap restart docker`

### 验证配置

使用以下命令验证 Docker 是否正确应用了配置。
`docker info`

在输出中查找 Registry Mirrors 部分，确认包含你的镜像地址。

### 检查 Snap 的 Docker 日志

如果有问题，可以查看 Snap 的 Docker 日志以了解更多信息。
`sudo snap logs docker`

通过这些步骤，你应该能够配置 Snap 安装的 Docker 使用加速镜像地址。

---

**参考资料**

- [snap 安装的docker，如何添加加速镜像和重启服务](https://blog.enianteam.com/u/sun/content/265)
- [Docker/DockerHub 国内镜像源/加速列表（10月21日更新-长期维护）](https://xuanyuan.me/blog/archives/1154?from=tencent#_registry_mirror)
