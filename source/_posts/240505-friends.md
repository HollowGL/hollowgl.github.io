---
title: 友链页面的实现
toc: true
date: 2024-05-05 20:45:27
updated: 2024-05-05 20:45:27
cover:
thumbnail:
categories: 建站历程
tags:
---

一个好友也建了[博客](https://maksymilan.github.io/)，特此创建友链以示支持
<!-- more -->

## 前言
有了实现“纸条”模块的经验，这次创建“友链”页面就很轻车熟路了，依然是大量参考~~搬运~~[这位大佬](https://removeif.github.io/theme/%E5%8D%9A%E5%AE%A2%E6%BA%90%E7%A0%81%E5%88%86%E4%BA%AB.html)开源的代码。

发明**全局搜索**的人真是个天才，在实现过程中帮了我不少忙。仅需要一丁点儿前端三件套的知识，然后**抄就完事**。

最终实现效果见[友链](https://hollowgl.github.io/firends/)。

## 页面主体

新建一个页面：
```bash
hexo new page friends
```

添加配置：
```diff _config.icarus.yml
navbar:
    # Navigation menu items
    menu:
        时间轴: /archives
        分类: /categories
        标签: /tags
+       友链: /friends
        纸条: /self_talking
        关于: /about
```

新页面中添加内容（摘自[此处](https://github.com/removeif/hexo-theme-icarus-removeif/blob/master/source/friend/index.md)）：
```html
<script type="text/javascript" defer src="/js/friends.js"></script>
<div class="links-content">加载中，稍等几秒...</div>
```

## js
在`source/js`目录下新建`friends.js`文件，内容如下（较[原代码]((https://github.com/removeif/hexo-theme-icarus-removeif/blob/master/themes/icarus/source/js/friend.js))有所简化）：
```javascript 点击展开 >folded
// author by removef
// https://removeif.github.io/

$(function () { //获取处理友链数据
    $.getJSON("../json_data/friends.json", function (data) {

        $('.links-content').html("");

        // 随机排序过滤失效的
        let notValid = data.filter((item, a, b) => item.valid == 0);
        data = data.filter((item, a, b) => item.valid != 0).sort(function (a, b) {
            return Math.random() > .5 ? -1 : 1;
        });
        $('.links-content').append("<div class='friend-title-item'><br>朋友们<br><br><hr></div>");
        $.each(data, function (i, e) {
            var html = "<div class=\"friend-card-item\">";
            if (e.src == undefined) {
                html += "    <img class=\"ava\" src=\"/img/nopic.jpg\" title=\"图片链接不可用，使用的默认图片\">";
            } else {
                html += "    <img class=\"ava\" src=\"" + e.src + "\">";
            }
            html +=
                "<div class='text-desc' title=\""+e.desc+"\">    网址：<a href=\"" + e.url + "\" target=\"_blank\">" + e.name + "</a>" +
                "    <br>时间：" + e.date +
                "<br>简介：" + e.desc + "</div>" +
                "    </div>";

            $('.links-content').append(html);
        });
    })
});
```

不得不说，这段js代码的可读性比纸条模块的[js代码](https://github.com/removeif/hexo-theme-icarus-removeif/blob/master/themes/icarus/source/js/gitalk_self.min.js)好太多


## json数据源
从js代码中不难看出，友链的具体信息是从`json_data/friends.json`文件中读取的，所以我们需要新建这个文件，里面填写需要展示的友链，比如：
```json
[
    {
        "date": "2024.05.04",
        "src": "https://images.pexels.com/photos/1292241/pexels-photo-1292241.jpeg",
        "name": "Maksymilan",
        "desc": "Hi!",
        "url": "https://maksymilan.github.io/"
    }
]
```

js代码对没有头像的博客进行了处理，使用了默认头像。如果json数据中没有`src`字段，就会使用`/img/nopic.jpg`作头像。


## CSS
js代码中出现名为`friend-title-item`和`friend-card-item`的类，都不是原主题中的类。通过搜索，我发现需要在`themes\icarus\source\css\default.styl`中实现，需添加（摘自[此处](https://github.com/removeif/hexo-theme-icarus-removeif/blob/97d700c97fb3226c1a55d6f38785dc6e83322a13/themes/icarus/source/css/base.styl#L887)）：

```css >folded
.friend-title-item {
    font-weight: bold;
    text-align: center;
}

.friend-card-item {
    width: 50%;
    border-radius: 2px;
    color: #4a4a4a;
    padding: 0.5rem;
    display: inline-block;
    margin: 5px;
    margin-top: 15px;
    .text-desc {
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    .ava {
        width: 5rem !important;
        height: 5rem !important;
        margin: 0 !important;
        margin-right: 0.2em !important;
        border-radius: 40px;
    }
    img {
        float: left;
        transition: all 0.5s ease-in;
        -webkit-transition: all 0.5s ease-in;
        -moz-transition: all 0.5s ease-in;
        -o-transition: all 0.5s ease-in;
    }
    img:hover {
        transform: rotate(360deg);
        -webkit-transform: rotate(360deg);
        -moz-transform: rotate(360deg);
        -o-transform: rotate(360deg);
        -ms-transform: rotate(360deg);
    }
    &:hover {
        transform: scale(0.995);
        box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.12), 0 0 6px 0 rgba(0, 0, 0, 0.04);
    }
}
```

至此，大功告成！