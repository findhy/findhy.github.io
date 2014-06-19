---
layout: post
title: "怎么在Octopress blog中嵌入slideshare"
date: 2014-06-19 10:56:50 +0800
comments: true
categories: Others
keywords: Octopress,slideshare
description: 怎么在Octopress blog中嵌入slideshare
---
怎么在Octopress blog中嵌入slideshar，使用该插件[Octopress-Slideshare-Plugin](https://github.com/petehamilton/Octopress-Slideshare-Plugin)，下面是操作过程。
<!--more-->
### 1.下载插件 ###

    git clone https://github.com/petehamilton/Octopress-Slideshare-Plugin.git

### 2.安装插件 ###

进入上面下载的插件目录，拷贝Octopress-Slideshare-Plugin/slideshare.rb到octopress/plugins目录下面

### 3.生成对应的Slideshare Embed ID ###

到[slideshare](http://www.slideshare.net/)网站找到你需要嵌入的ppt。

{% img /images/slideshare-1.png %}

点击Embed，生成类似下面的链接，最后的291600就是我们要的Slideshare Embed ID   
    http://www.slideshare.net/slideshow/embed_code/291600

### 4.嵌入 ###

在你的bolg MD文件中添加下面的代码：

{% img /images/slideshare-2.png %}

执行

    rake generate
    rake preview

看到  

{% slideshare 291600 %}

