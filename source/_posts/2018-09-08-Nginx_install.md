---
title: Nginx简介及安装
date: 2018/9/8 19:32:00
tag : 
- Synopsis
- install
categories:
- nginx

---

Nginx (engine x) 是一个高性能的HTTP和反向代理服务，也是一个IMAP/POP3/SMTP服务。本文主要简单介绍Nginx及linux（centos6）环境下的安装步骤。

<!-- more -->

# What

C语言开发 轻量级 高性能 http服务器（静态资源）/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。

ps：官方测试nginx能够支撑5万并发连接，且CPU、内存等资源消耗非常低，运行非常稳定。

# When

## 反向代理

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

用途：
可以提供从防火墙外部代理服务器到防火墙内部安全内容服务器的加密连接。
可以允许客户机安全地连接到代理服务器，从而有利于安全地传输信息。

## 负载均衡（软件）

软件负载均衡解决方案是指在一台或多台服务器相应的操作系统上安装一个或多个附加软件来实现负载均衡，它的优点是基于特定环境，配置简单，使用灵活，成本低廉，可以满足一般的负载均衡需求。

当有2台或以上服务器时,根据规则随机的将请求分发到指定的服务器上处理,负载均衡配置一般都需要同时配置反向代理,通过反向代理跳转到负载均衡。 而 Nginx 目前支持自带3种负载均衡策略（RR,默认,按时间；权重；ip_hash）,还有2种常用的第三方策略。

## HTTP服务器

Nginx 本身也是一个静态资源的服务器,当只有静态资源的时候,就可以使用 Nginx 来做服务器,同时现在也很流行动静分离。

## 正向代理

正向代理,意思是一个位于客户端和原始服务器(origin server)之间的服务器,为了从原始服务器取得内容,客户端向代理发送一个请求并指定目标(原始服务器),然后代理向原始服务器转交请求并将获得的内容返回给客户端。

nginx的正向代理，只能代理http、tcp等，不能代理https请求。

# Why

1. 工作在网络的7层结构之上，可以针对http应用设置分流策略，正则规则更加强大灵活
2. 对网络稳定性依赖小，能ping通就能进行负载功能
3. 安装配置简单
4. 高负载能力、高稳定、低资源消耗
5. 可通过端口检测服务器内部故障，根据服务器处理网页返回的状态码、超时等等，并且会把返回错误的请求重新提交到另一个节点
6. 反向加速缓存
7. 可作为中层反向代理使用
8. 作为静态网页和图片服务器性能高
9. 社区活跃，第三方模块多

# Why not

1. 只支持http、https、Email协议，使用范围小
2. 不支持通过url检测后端服务器健康状态，不支持Session的直接保持（但能通过ip_hash来解决）

# How

## 安装

### 环境准备

nginx的编译需要c++，prce（重定向支持）和openssl（https支持），根据要求，本机环境情况，选择安装相应环境。

	# yum install gcc-c++
	# yum -y install pcre*
	# yum -y install openssl*

### 下载并解压

使用wget指令或者直接[官网](http://nginx.org/)下载。

	# cd /usr/local/
	# wget http://nginx.org/download/nginx-1.9.9.tar.gz

解压：
	
	# tar -zxvf nginx-1.9.9.tar.gz

### 编译

1.进入nginx目录

	# cd nginx-1.9.9

2.设置安装目录

	# ./configure --prefix=/usr/local/nginx

3.编译安装

	# make
	# make install


### 启动nginx服务

进入nginx安装目录，运行nginx。
	
	# cd /usr/local/nginx/sbin
	# ./nginx

验证是否有nginx进程：

	# ps -ef|grep nginx

### 设置开启启动

1.在/etc/init.d下创建文件nginx

	# vi /etc/init.d/nginx

根据[官方脚本](http://wiki.nginx.org/RedHatNginxInitScript)修改/etc/init.d/nginx，特别注意修改以下参数：

	nginx="/usr/local/nginx/sbin/nginx" 
	NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf" 

2.设置文件执行权限

	# chmod a+x /etc/init.d/nginx

3.将nginx服务加入chkconfig管理列表

	# chkconfig --add /etc/init.d/nginx

4.开机自启动

	# chkconfig nginx on

### 验证安装

输入IP进行访问，若有nginx的欢迎页面，则安装成功。

## 常用指令

	nginx -c /usr/local/nginx/conf/nginx.conf  启动nginx
	nginx -s quit 		停止ngix
	nginx -s reload 	重新载入nginx(当配置信息发生修改时)
	nginx -s reopen 	打开日志文件
	nginx -v			查看版本
	nginx -t			查看nginx的配置文件的目录
	nginx -h  			查看帮助信息

# 安装过程问题

## 问题一：zlib library：
	./configure: error: the HTTP gzip module requires the zlib library.
	You can either disable the module by using –without-http_gzip_module
	option, or install the zlib library into the system, or build the zlib 
	library
	statically from the source with nginx by using –with-zlib=<path> option.

解决方案：
	yum install -y zlib-devel	

