---
title: 碎碎念(纸条)模块实现
toc: false
date: 2023-10-07 22:01:11
updated: 2023-10-07 22:01:11
cover:
thumbnail:
categories: 建站历程
tags:
excerpt: 记录了自己模块化实现碎碎念功能的一些坑。
---

## 背景

看到网上有大佬在自己的博客里开发了一个[碎碎念模块](https://removeif.github.io/self-talking/)，便想着也~~实现~~COPY一个。虽然该博主慷慨地开源了其[博客源码](https://removeif.github.io/theme/%E5%8D%9A%E5%AE%A2%E6%BA%90%E7%A0%81%E5%88%86%E4%BA%AB.html)，但我在实现碎碎念模块时仍遇到了不少问题。


原博主后来还直接自制了一个主题，集成了各种有用的功能。但对我来说这些功能可能不太实用，我不太想为了适配新主题再去修改各种配置。于是，我便开始探索碎碎念这一功能的具体实现。
> 后面遇到麻烦时，我不得不去将新主题下载下来，以弄清楚碎碎念的实现细节。不过，新主题的配置与目前使用的icarus主题的配置有很大不同，且由于缺少细致的文档，最终也没配置好新主题。


## 准备工作

首先需要新开一个页面，名字随意，其实和写新文章差不多：
```bash
hexo new page self_talking
```

此时source文件下新增一名为self_talking的目录，其下新增名为index.md的文件。随后还需在_config.icarus.yml文件中新增碎碎念词条：
```yml
navbar:
    # Navigation menu items
    menu:
        时间轴: /archives
        分类: /categories
        标签: /tags
        关于: /about
        碎碎念: /self_talking
```
此时使用 `hexo g && hexo s` 预览就能看到导航栏上的“碎碎念”了，不过点击后页面还没有任何内容。


## Gitalk

碎碎念实际上是基于评论功能实现的，而后者又依托[Gitalk插件](https://github.com/gitalk/gitalk#install)实现。Gitalk的使用比较简单，它的文档也描述得很清晰。简单概括，就是在文章结尾加上以下代码：
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>
<script>
var gitalk = new Gitalk({
    clientID: 'GitHub Application Client ID',
    clientSecret: 'GitHub Application Client Secret',
    repo: 'GitHub repo',
    owner: 'GitHub repo owner',
    admin: ['GitHub repo owner and collaborators, only these guys can initialize github issues'],
    id: location.pathname,      // Ensure uniqueness and length less than 50
    distractionFreeMode: false  // Facebook-like distraction free mode
})
gitalk.render('gitalk-container')
</script>
```
Gitalk所作的事其实就是把一个公共仓库的issues里的评论拿来做博客的评论。在repo这一栏，可以直接填写远端托管博客的仓库名，也可以新建一个专门用于存储评论的空仓库并填写该仓库名字。

`id` 一栏可暂时随便填写。html标签中的两个链接值得注意，它们是实现碎碎念的关键，之后会对其进行修改。

此时使用 `hexo d` 将博客推送到远端，待更新后进入碎碎念页面时，Gitalk会自动帮你初始化一个issues以构建评论功能。

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
也可以选择手动参加，需进行配置，参加Gitalk文档
{% raw %}</div></article>{% endraw %}

至此，你和其它访问该页面的人，都能添加评论。


## 碎碎念

现在，只需在Gitalk基础上再完善两个功能：
- 取消其他人的评论，且限制其添加issues
- 取消评论显示中的头像，仅留下文本

### js

在第一个步骤上，我们可以直接抄作业。可以直接翻看大佬的碎碎念页面的[源代码]:
```html 点击展开 >folded
<div class = "text-center"><h1>碎碎念</h1></div><div class = "text-tips">

tips：github登录后按时间正序查看、可点赞加❤️、本插件[地址](https://github.com/removeif/gitalk)..<span id="busuanzi_container_page_pv">「<span id="busuanzi_value_page_pv">+99</span>次查看」</span></div>
<div id="comment-container1"><div class="text-tips">碎碎念加载中，请稍等...</div></div>
<link rel="stylesheet" href="https://cdnjs.loli.net/ajax/libs/gitalk/1.6.0/gitalk.css"/>
<script>
    $.getScript("/js/gitalk_self.min.js", function () {
        var gitalk = new Gitalk({
            clientID: '46a9f3481b46ea0129d8',
            clientSecret: '79c7c9cb847e141757d7864453bcbf89f0655b24',
            id: '666666',
            repo: 'issue_database',
            owner: 'removeif',
            admin: "removeif",
            createIssueManually: true,
            distractionFreeMode: false
        });
        gitalk.render('comment-container1');
    });
</script>
```
它调用JavaScript函数的方式与Gitalk原版略有差异，我照搬了这一代码，在我这儿它并没有奏效。所以最后我还是按照Gitalk的方式调用函数，其中的[js文件](https://github.com/removeif/hexo-theme-icarus-removeif/blob/master/themes/icarus/source/js/gitalk_self.min.js)和[css文件](https://cdnjs.loli.net/ajax/libs/gitalk/1.6.0/gitalk.css)可以下载到本地：

```html js文件也可以引用 >folded
<script src="/js/gitalk_self.min.js"></script>
<link rel="stylesheet" href="/css/self_talking.css">
<div id="comment-container1"><div class="text-tips">碎碎念加载中，请稍等...</div></div>
<script>
    var gitalk = new Gitalk({
        clientID: 'dae8c0049c7911dd0ba5',
        clientSecret: '74886ab9d902dd7e8ffd560862db00676a34d2c4',
        id: 'test',
        repo: 'hollowgl.github.io',
        owner: 'hollowgl',
        admin: "hollowgl",
        createIssueManually: true,
        distractionFreeMode: false
    })
    gitalk.render('comment-container1')
</script>
```

此时的碎碎念前端已经有了变化，访客不再拥有评论功能。但是，访客可以通过直接在issues里评论，这些评论也会被Gitalk加载到碎碎念中。这个问题很好解决，只需在issues页面右下角选择`lock conversation`
{% raw %}<div class="message is-info">{% endraw %}
大佬修改的js代码还实现了查看次数统计和碎碎念统计。虽然博主已经魔改Gitalk为其添加上了[新的功能](https://github.com/removeif/gitalk/tree/self-talking1)，但是却没写详尽的说明文档。
{% raw %}</div>{% endraw %}

### css

另一个问题是取消评论里头像的显示，这个问题也很好解决，只需在本地下载的css文件作以下修改：
```css
.gt-container .gt-avatar {
  /* display: inline-block; */
  display: none;
  width: 3.125em;
  height: 3.125em;
}
```


但是让我很困惑：原博主使用的CSS文件经我翻阅好像和原来的Gitalk无异，然而最终显示效果却有不小的差异，且JavaScript文件似乎也没有起到更改样式的作用。我向ta发了邮件请教，目前没得到答复。


## 参考链接

- [Gitalk中文说明](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)
- [@辣椒の酱 源码分享](https://removeif.github.io/theme/%E5%8D%9A%E5%AE%A2%E6%BA%90%E7%A0%81%E5%88%86%E4%BA%AB.html)




