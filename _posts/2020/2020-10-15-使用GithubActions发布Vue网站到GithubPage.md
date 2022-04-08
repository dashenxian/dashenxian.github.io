---
title: "使用GithubActions发布Vue网站到GithubPage"
publishDate: 2020-10-15 19:26:00 +0800
date: 2020-10-15 19:14:08 +0800
categories: Vue GithubActions GithubPage
position: problem
---

偶然看到一个介绍使用GitHubAction做的百度贴吧[自动签到](https://github.com/srcrs/TiebaSignIn)，然后一路顺藤摸瓜看到了阮一峰大神写的[GitHub Actions 入门教程中介绍](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)，示例中介绍了把React项目发布到GitHub Pages，我就想能不能把vue的项目发布到GitHub Pages呢(主要是不会React)。
说干就干。

---

<div id="toc"></div>

## 创建vue项目

这个不讲了用vue-cli一个命令就可以了。现在我已经创建了一个ts模板的项目（js项目是一样的），项目名称叫vue-github-actions-demo，结构如下。你们初始化的项目可能有些文件没有，如果是我后面添加发布流程文件我会讲到，如果是vue原生文件的差异不影响发布流程。
![ci.yml](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-3.jpg)

## 创建配置文件

从cli3开始取消了自动创建配置文件，这里需要手动创建配置文件，创建配置文件主要是因为我的githubPage主页，已经有一个网站了(https://dashenxian.github.io)，所以我只能用项目地址访问https://dashenxian.github.io/vue-github-actions-demo/，如果这里我不加这个二级目录，vue对js文件的文件引用就有问题。如果你想用你的githubPage主页访问，这一步骤可以跳过，但是需要把项目名称改成你的github用户名称。
在根目录下创建vue.config.js文件，添加如下代码：

```js
//vue.config.js
module.exports = {
    // 选项...
    publicPath: process.env.NODE_ENV === 'production'
    ? '/vue-github-actions-demo/'
    : '/'
  }
```

![vue.config.js](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-1.jpg)

## 创建workflow（工作流程）

在根目录逐级创建.github\workflows目录，在workflows下创建yml工作流程文件，文件名称可以随意，githubActions会执行全部的yml流程。
输入以下代码：

```yml
name: GitHub Actions Build and Deploy Demo
on:
  push:
    branches:
      - master
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2.3.1 # If you're using actions/checkout@v2 you must set persist-credentials to false in most cases for the deployment to work correctly.
        with:
          persist-credentials: false

      - name: Install and Build 🔧 # This example project is built using npm and outputs the result to the 'build' folder. Replace with the commands required to build your project, or remove this step entirely if your site is pre-built.
        run: | #注意我这里是使用的yarn管理包，如果你使用的npm，请换成npm的命令：npm install和npm run build
          yarn
          yarn build
      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@3.6.2
        with:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }} #secrets.ACCESS_TOKEN是项目配置的密钥
          BRANCH: gh-pages # The branch the action should deploy to.
          FOLDER: dist # The folder the action should deploy.
          CLEAN: true # Automatically remove deleted files from the deploy branch
```

![ci.yml](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-2.jpg)

## 推送项目到GitHub

把项目代码推送到github，这里可以用vs打开项目文件夹，vs的团队管理中可以直接推送到github，当然你也可以选择其他方式，比如vscode或者命令。

## 配置项目密钥

现在你的项目已经推送到github，在GitHub中定位到项目

1. 找到Settings配置，按下图步骤添加密钥，如果你还没有密钥，按第二步生成密钥后再添加。
   - ![Settings](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-4.jpg)
   - ![密钥](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-5.png)
2. 没有密钥按下图步骤生成密钥
   - ![密钥](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-6.jpg)
   - ![密钥](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-7.jpg)
   - ![密钥](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-8.jpg)

## 配置GitHubPage

仍然在项目的Settings配置页面，找到GitHub Pages选项（在页面靠近最底部），按下图配置。

![密钥](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-9.jpg)

## 完成

现在你修改代码并推送到项目仓库，点击项目的Actions应该就能看到自动生成正在运行了。
等到运行完成通过，在GitHubPage就能看到熟悉的vue启动页面。

![密钥](/static/posts/2020/使用GithubActions发布Vue网站到GithubPage-10.jpg)

---

**参考资料**

- [GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)
- [配置参考](https://cli.vuejs.org/zh/config/#devserver)
- [Deploy to GitHub Pages](https://github.com/marketplace/actions/deploy-to-github-pages)