---
title: Linux下SSH免密登陆
date: 2018/9/14 21:19:12
updated: 
tag: 
- ssh
- operation
categories:
- linux
---
与密码验证登陆方式相比，采用证书认证的方式，比使用密码登陆更安全（防止暴力破解）也更便捷本文主要记录如何免密码。本文默认已安装SSH。

<!-- more -->

# 生成密钥（客户端）

已有密钥可跳过。

1、输入以下命令：

	ssh-keygen -t rsa

2、后续的直接敲3个回车，其中：
* 第一次回车是采用默认的密钥名称：rsa_id
ps：**建议采用默认名称**。否则，在登陆时可能出现找不到密钥的情况，而导致无法登陆（未找到解决方案）;
* 第二、三次回车是采用空的密钥密码。

3、等待密钥生成（生成位置默认为当前用户文件夹下的.ssh文件夹内：~/.ssh/）,其中rsa_id为私钥，rsa_id.pub为公钥。

# 配置公钥（服务端）

1、复制公钥到服务端（使用scp、sftp客户端等方式）

2、确认需免密登陆的用户主目录下（普通用户:/home/userName;Root:/root）是否有.ssh目录,并且.ssh目录下有authorized_keys文件，没有则进行创建；

	# cd ~
	# mkdir .ssh
	# vi ./.ssh/authorized_keys

3、将生成的公钥复制到.ssh目录下，并追加到authorized_keys文件中。
	# cd ~/.ssh
	# cat ./rsa_id.pub >> ./authorized_keys

4、保证.ssh目录的权限是700，且.ssh/authorized_keys的权限是600，不是则进行权限变更。

	# cd ~/.ssh
	# chmod 700 .ssh
	# chmod 600 .ssh/authorized_keys

# 配置SSH（服务端）

1、修改/etc/ssh/sshd_config的配置，确定以下配置正确。

	RSAAuthentication yes  //允许用RSA密钥进行身份验证
	PubkeyAuthentication yes  //允许用公钥进行身份验证
	AuthorizedKeysFile .ssh/authorized_keys  //本机保存的公钥的文件

2、重启服务

	service sshd restart

# 验证

在客户端输入以下指令进行验证：

	ssh userName@hostIp -p port

首次登陆时，会弹出主机名的确认，输入yes并回车即可；若未配置整数密码，但登陆要求输入密码，可再重新试一次，若还需输入密码，则证明配置有误。

# 注意事项

免密登陆失败，主要有以下的原因：

服务端方面：
1. 文件及权限问题，请在次确认存在.ssh目录及.ssh/authorized_keys，且文件的权限为700和600；
2. sshd配置问题，请保证开启了rsa验证；
3. authorized_keys内容问题，请保证authorized_keys中追加的公钥的格式正确（被这个关了很久，cat进去的格式有问题）

客户端方面：
1. rsa_id文件的路径。确认rsa_id的名称及路径是否正确。

若有问题，可用 

	ssh userName@hostIp -p port -vvv 

进行调试。
