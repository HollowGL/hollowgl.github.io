---
title: 博客的迁移
date: 2023-09-12 18:56:37
updated: 2023-09-12 18:56:37
cover:
thumbnail:
categories: 建站历程
tags:
excerpt: 这篇文章简要的记录了迁移博客的方法。
---

在Windows上构建该博客系统后，我将其托管到了GitHub，试图在Mac上克隆博客系统，希望实现在两台电脑上分别写文章，是否可行？先给出结论，由于依赖版本的不同，**跨平台迁移几乎就是重构整个博客项目的过程**。

这意味着，我只能在一台电脑上推送博客，另一台电脑只能用来写md文件。如果两个电脑上能做到下载相同版本的环境话，也许能实现目标，不过可能有点麻烦，具体细节我就没探索了。

所以，这篇文章的意义在于指导如何迁移博客。

## 安装ndoe.js和hexo
使用homebrew安装[node.js](https://nodejs.org/zh-cn)（macOS）
```bash
brew install node
```

查看是否安装成功。
```bash
node -v
npm -v
```
Windows我的node和npm版本分别是v18.17.0和9.8.1，homebrew安装没法指定具体的小版本，导致后面下载到的各种依赖的版本也会有很多不同。

安装hexo
```bash
npm install hexo-cli -g
```

## 安装依赖包和主题
首先clone原仓库，只需要[main分支](https://github.com/HollowGL/hollowgl.github.io)即可，main分支被我用于托管博客系统。
```bash
git clone -b main <repository_url>
```
当然，如果是重建博客的话，只需把原来的目录复制下来即可，其中的node_modules文件夹不必复制。

进入克隆仓库目录，以下命令都在该目录下运行。

接着直接运行以下命令，npm自动会根据package.json和package-lock.json这两个文件的内容，下载并安装所需的包及其依赖项，它们都在新生成的node_modules文件夹里。
```bash
npm install
```

还需安装主题，我使用的主题是[icarus](https://ppoffice.github.io/hexo-theme-icarus/)。按照主题文档说明，需运行以下命令：
```bash
git clone https://github.com/ppoffice/hexo-theme-icarus.git themes/icarus -b <version number> --depth 1
```
icarus也可以通过npm下载。

## 报错 | 冲突产生
使用 `hexo g && hexo s` 来预览页面，但是此时发生了一些错误，提示某些依赖的版本与所需版本不兼容。随即根据报错提示更新了依赖包，最后成功生成博客内容。

但是package.json和package-lock.json都发生了更改，这很可能与下载的node.js和hexo的版本与之前的不一致有关。所以，我没法同时在两个电脑上构建博客系统，保持各个环境和依赖的版本一致恐怕是一件很麻烦的事情。