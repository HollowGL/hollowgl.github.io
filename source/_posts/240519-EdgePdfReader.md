---
title: Edge禁用Adobe Acrobat作为PDF阅读器
toc: true
date: 2024-05-19 22:34:42
updated: 2024-05-19 22:34:42
cover:
thumbnail:
categories: 代码以外的Bug
tags:
---

参考链接：[[Tip] How to Restore Classic Native PDF Viewer in Microsoft Edge](https://www.askvg.com/enable-or-disable-adobe-acrobat-pdf-reader-in-microsoft-edge/)

1. Edge地址栏输入`edge://flags/`
2. 搜索 `new PDF`，找到`New PDF Viewer`一项，将选项置为**Disabled**
3. 重启Edge即可生效

<!--more-->

## 问题

Edge目前是我的打开PDF的默认程序，但是今天在打开一个PDF时，发现其中的**公式被渲染成了乱码**。尝试使用其它PDF阅读器打开，比如Chrome浏览器和福昕阅读器，发现都能正常显示。之前冲浪时看到有人抱怨Edge的新PDF阅读器，所以猜测是Edge内置的PDF阅读器渲染有问题。
<div align="center">
<a href="https://imgse.com/i/pkKd5WR"><img src="https://s21.ax1x.com/2024/05/20/pkKd5WR.md.png" alt="pkKd5WR.png" border="1"/></a>
</div>
早期Edge使用自己的PDF阅读器，后来在23年2月的一次更新中改用Adobe Acrobat。更新后使用Edge打开PDF，右下角会显示Acrobat的LOGO。

<div align="center">
<a href="https://imgse.com/i/pkKdLwD"><img src="https://s21.ax1x.com/2024/05/20/pkKdLwD.png" alt="pkKdLwD.png" border="0"/></a>
</div>

## 解决
google了一下“Edge关闭Adobe Acrobat”，搜寻结果都是关于早期关于Acrobat扩展的内容，那时Acrobat应该是作为Edge的插件存在，还未嵌入浏览器中。随后使用英文搜索“edge disable adobe acrobat”，找到了参考链接中的文章。

文章中提供了两种解决方法，一是修改Edge的flags，禁用新的PDF阅读器；二是修改注册表，禁用Acrobat插件。我试了第一种方案，成功解决。
<div align="center">
<a href="https://imgse.com/i/pkKdjFH"><img src="https://s21.ax1x.com/2024/05/20/pkKdjFH.md.png" alt="pkKdjFH.png" border="1" /></a>
</div>

再次贴出[**原文**](https://www.askvg.com/enable-or-disable-adobe-acrobat-pdf-reader-in-microsoft-edge/) 

## flags
Edge的flags是一个高级设置页面，可以修改一些实验性功能，很久之前还改动过里面的内容，但是现在已经忘了。挖个坑在此，后续再遇到flags再看看。
