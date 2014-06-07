---
layout: post
title: "s3cmd 配置和使用"
date: 2014-06-05 18:52:03 +0800
comments: true
categories: AWS
---
s3cmd是AWS S3的命令行工具，可以用来下载、上传、同步文件，还可以配置权限。
<!--more-->
### 一、配置 ###
#### 1.安装python-pip ####
    yum install python-pip
如果提示No package python-pip available，先安装[epel-release-6-8.noarch.rpm](http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm)

    rpm -ivh epel-release-6-8.noarch.rpm

#### 2.安装s3cmd ####
    pip install s3cmd

#### 3.配置s3cmd ####
    s3cmd --configure

分别输入Access Key和Secret Key
    Access Key:
    Secret Key:

回车，如果提示：
    Encryption password is used to protect your files from reading
    by unauthorized persons while in transfer to S3
    Encryption password:

输入当前登录的操作系统用户

#### 4.HTTPS协议选择no ####
    Use HTTPS protocol [No]: no

#### 5.代理不填直接回车 ####
    HTTP Proxy server name: 

#### 6.最后就是测试保存 ####
    Test access with supplied credentials? [Y/n] y
    Save settings? [y/N] y

#### 7.测试 ####
    s3cmd ls

#### 8.下载整个目录 ####
    s3cmd get --recursive dir_name

### 二、使用 ###
#### 1.查看所有的Buckets ####
    s3cmd ls

#### 2.创建Buckets ####
    s3cmd mb s3://my-bucket-name

#### 3.删除空Buckets ####
    s3cmd rb s3://my-bucket-name

#### 4.上传 ####
    上传单个文件：s3cmd put file.txt s3://my-bucket-name/file.txt
    批量上传：s3cmd put ./* s3://my-bucket-name/
    上传整个目录：s3cmd put -r dir1 s3://my-bucket-name/
    上传目录下所有文件：s3cmd put -r dir1/ s3://my-bucket-name/

#### 5.下载 ####
    下载单个文件：s3cmd get s3://my-bucket-name/file.txt file.txt
    下载整个目录下的文件：s3cmd get -r  s3://my-bucket-name/bucket_dir ./

#### 6.删除文件 ####
    s3cmd del s3://my-bucket-name/file.txt

#### 7.查看空间大小 ####
    s3cmd du -H s3://my-bucket-name

#### 8.同步 ####
    同步当前目录下所有文件：s3cmd sync  ./  s3://my-bucket-name/
    只列出不同步：s3cmd sync  --dry-run ./  s3://my-bucket-name/
    同步且删除本地不存在的文件：s3cmd sync  --delete-removed ./  s3://my-bucket-name/
    不检验跳过本地已经存在的文件：s3cmd sync  --skip-existing ./  s3://my-bucket-name/
    排除包含文件规则：s3cmd sync --dry-run --exclude '*.txt' --include 'dir2/*' ./ 
    更多参考这里：http://s3tools.org/s3cmd-sync



