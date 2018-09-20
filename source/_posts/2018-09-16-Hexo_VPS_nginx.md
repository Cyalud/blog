---
title: 基于Hexo的个人博客搭建
date: 2018/m/d h:mm:ss
updated: 
tag: 
- Vps
- blog
categories:
- hexo
---

上周四，心血来潮想整个博客，然后，说整就整，然后就被各种有的没的问题关了两个礼拜，截至目前，除了域名解析问题外，其他的也都算解决了吧。特此，记录下过程及坑。

本次主要基于HEXO进行搭建（就是现在的这个网站了），环境是VPS+nginx（也可以部署在github上），部署主要运用了hexo的hexo-deployer-rsync插件进行自动（另有ftpsync及githook的方式），为防止意外把网站的主题和md文件放到了github上进行备份。

Let's begin.

<!-- more -->

# 本地环境准备（win10下）

## nodeJs

下载及运行HEXO的前置条件，不多说，直接上[官网](https://nodejs.org/zh-cn/)下载安装。

对于强迫症可能会用到：

	npm config set prefix “#Your_DIR” 
	npm config set cache “#Your_DIR”

以上为更改npm的全局模块及缓存的目录，#Your_DIR换成你想存放的路径，另，为了便于使用，需修改系统的环境变量。

## hexo

用于blog的框架、静态代码的生成、部署。同样，直接上[官网教程](https://hexo.io/zh-cn/docs/)。

怎么安装、怎么初始化、怎么设置一站搞定。

因为用到hexo-deployer-rsync进行部署，所以，需要下载。

## SSH

因为用到hexo-deployer-rsync进行部署，所以，SSH也是必要的（为了与VPS服务器连接及数据同步）。win10好像是自带ssh（好像），为了管理VPS也下载了SSH Client，没有的话需要自己装。

# 服务器环境准备（centOS6下）

## rsync

rsync用于本地与服务器的同步，安装步骤见本站博文[Rsync简介、安装及使用](../../08/2018-09-08-rsync_intro_and_install/#%E5%AE%89%E8%A3%85)，也可自行谷歌或度娘;

PS：rsync的相关配置中，用户建议使用root。原因如下：windows与linux的权限机制不一样，同步后，文件只保留最低权限，因此，用权限低的用户会出现permission denied的问题，引起额外的麻烦；应该有好的解决方案，但是没有找到，等找到后再进行更新补充。

## nginx 

nginx作为静态资源服务器使用。安装步骤见本站博文[Nginx简介及安装](../../08/2018-09-08-Nginx_install/#%E5%AE%89%E8%A3%85)，也可自行谷歌或度娘;

PS：nginx相关配置中，用户也建议使用root，因为nginx会用到rsync同步的文件，而上述rsync就是使用root更新的，连锁反应。

# 本地与服务器的同步部署

用于一条指令同步部署，因为采用hexo-deployer-rsync，要求提供一个可以使用SSH登陆远程主机的Linux用户，为避免每次更新输密码，所以配置免密登陆。

搭建过程大部分时间浪费在这个上面，具体操作见博文[Linux下SSH免密登陆](../../14/2018-09-15-Linux_ssh_login/)

# nginx及hexo配置

## 配置nginx

修改nginx的配置/usr/local/nginx/nginxconf（根据自己的安装目录）

	listen       80;
	root   /home/blog;
	server_name  localhost;

	location / {
  		index  index.html index.htm;
	}
	#其余配置掠过不讲，在nginx相关知识点中再总结


## 配置hexo

配置本地博客项目下hexo的配置文件_config，设置以下内容：

	deploy:
		type: rsync
		host: hostIp     #主机名称
		user: root		 #用户名称（rsync使用的用户），建议为root
		root: /home/blog #目录，即上面nginx对应的目录，及linux上rsync同步的目录
		prot: port       #服务器上ssh的监听端口
	#其余配置略过不讲，请参考官方文档。

# 总结

花了比较长的时间再搭建博客上面，也算多了解了一点点吧。
在想说，博客建好了，形式也就有了，至于内容如何，能不能坚持都是后话，只是，也是该努力一下了。
