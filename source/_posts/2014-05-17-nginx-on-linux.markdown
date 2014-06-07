---
layout: post
title: "Nginx在Linux上的安装和配置"
date: 2014-05-17 07:12:34 +0800
comments: true
categories: Nginx
---
Nginx是一个高性能的HTTP和反向代理服务器，官网在[这里](http://nginx.org/)。

### 1.配置nginx的yum源 ###
    sudo vi /etc/yum.repos.d/nginx.repo
添加下面的内容：
    [nginx]
    name=nginx repo 
    baseurl=http://nginx.org/packages/centos/$releasever/$basearch/ 
    gpgcheck=0 
    enabled=1
保存退出
### 2.安装 ###
    sudo yum install nginx
 
### 3.启动Nginx ###
    sudo /etc/init.d/nginx start
访问：http://localhost
 
### 4.其它命令 ###
    停止nginx服务：# /etc/init.d/nginx stop 
    启动nginx服务：# /etc/init.d/nginx start 
    编辑nginx配置文件：# vi /etc/nginx/nginx.conf
### 5.反向代理配置 ###
    sudo vi /etc/nginx/nginx.conf
配置：  
    upstream rexster {
    	server 127.0.0.1:8182 weight=1 max_fails=1 fail_timeout=10s;
    }
    
    upstream yarn {
    	server 127.0.0.1:8089 weight=1 max_fails=1 fail_timeout=10s;
    }
    
    upstream storm {
    	server 127.0.0.1:7070 weight=1 max_fails=1 fail_timeout=10s;
    }
    
    server {
    	listen 80;
    	server_name yarn.xxx.com;
    	location / {
    		proxy_pass http://yarn;
    		proxy_set_header Host $host;
    		proxy_set_header X-Real-IP $remote_addr;
    		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    		}
    	# redirect server error pages to the static page /50x.html
    }
    server {
    	listen 80;
    	server_name storm.xxx.com;
    	location / {
    		proxy_pass http://storm;
    		proxy_set_header Host $host;
    		proxy_set_header X-Real-IP $remote_addr;
    		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    		}
    	# redirect server error pages to the static page /50x.html
    }
输入域名测试，跳转成功

