---
title: Hello world!
cover: /img/头像B站.jpg
thumbnail: /img/头像B站.jpg
updated: 2023-07-23 14:30:00
categories: 碎碎念
---

花了数个小时，终于建立起了**专属于我的个人网站**。


先是下载了一个名为 *Node.js* 的东西，接着用其包管理器 *npm* 下载了 *hexo*，然后又是一系列包的安装……第二天才开始着手主题的配置，一个名为 *icarus* 的主题最终被我选中，又是一系列的配置……


*hexo* 的部署过程令我迷惑，它替我完成了 *git* 的推送，但是我的分支管理好像出了亿点问题。到现在本地还有4000+个挂起的更改，然而`hexo d`命令依然能推送更改，并且 *github pages* 的显示效果与本地预览一模一样。以往习惯的add、commit和push工作流现在却换为了这样一套模式：`hexo g`，`hexo s`，`hexo clean`和`hexo d`，好在很快便适应了。

第四天又花了点时间治理一下混乱的版本控制，未果。我已经受够繁文缛节了，直接`git push origin --force --all`强制推送。现在本地必要文件全放在远端main分支，`hexo d`生成文件则推送到gh-pages分支。推送后发现hexo还会自动创建很多远程分支来提示我更新package.json和package-lock.json中的依赖项版本，甚至已经pull requests，在我手动合并后还能自动删除分支，这些都是一个名为Dependabot的自动化管理服务做的。合并，合并，通通合并~

**不过，以后应该写点啥呢？**