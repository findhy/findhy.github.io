---
layout: post
title: "s3cmd 多账户配置"
date: 2014-07-24 15:16:17 +0800
comments: true
categories: AWS
keywords: s3cmd,多账户配置
description: s3cmd 多账户配置
---
关于s3cmd的安装配置参考这篇文章：[s3cmd Configure](http://findhy.com/blog/2014/06/05/s3cmd-config/)，本文介绍s3cmd多账户配置。
<!--more-->
#### 复制配置文件 ####
s3cmd安装完成之后，在home/user/目录下会生成一个 .s3cfg 配置文件，在复制一个出来

    cp .s3cfg .s3ad

#### 修改配置文件 ####

    vi .s3ad
    将access_key和secret_key修改成对应账户的值

#### 测试 ####

使用 -c 指令指定配置文件

    s3cmd -c ~/.s3ad ls s3://log-ad/adserver/ 

#### 改进 ####
会不会觉得每次用 -c 指定配置文件太麻烦了，用 Bash 别名，我们用一个别名来替代 *s3cmd -c ~/.s3ad*

    vi ~/.bashrc  
    添加
    alias s3ad='s3cmd -c ~/.s3ad'  

	执行下面命令，让配置文件生效
    source .bashrc  

    测试
    s3ad ls s3://log-ad/adserver/
    
    OK!这样就不用每次指定配置文件了

#### 参考 ####
https://blog.techopsguru.com/2011/12/s3-bucket-copying-with-multiple-accounts.html  
http://mikesisk.tumblr.com/post/8703449578/s3cmd-and-multiple-accounts  
http://mikesisk.com/2011/08/09/s3cmd-with-multiple-aws-accounts/  







