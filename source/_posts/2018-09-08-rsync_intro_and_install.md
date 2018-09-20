---
title: Rsync简介、安装及使用
date: 2018/9/8 19:32:00
tag : 
- rsync
- Synopsis
- install
- operation
categories:
- rsync

---

rsync（remote sync）是类unix系统下的数据镜像备份工具，被用于在linux/unix系统中执行备份操作。

<!-- more -->

# 官网

<https://www.samba.org/ftp/rsync/rsync.html>

# What

rsync是以实现两端主机的文件同步为主要目的而设计的，可以远程同步，支持本地复制，或者与其他SSH、rsync主机同步。虽然，rsync可以实现linux中scp、cp、rm等功能效果，但其是以自身算法实现。

# Why

与传统的cp、tar备份方式相比，rsync具有安全性高、备份迅速、支持增量备份、本地复制，远程同步等优点。其特性主要如下：

1. 能更新整个目录和树和文件系统
2. 有选择性的保持原来文件的符号链接、硬链接、文件属性、权限、设备以及时间等
3. 对于安装来说，无任何特殊权限要求
4. 优化的流程，文件传输效率高
5. 能用rsh、ssh或直接端口作为传输入口端口
6. 支持匿名rsync同步文件，是理想的镜像工具

# How

## 安装

	# yum install rsync

## 服务端配置

1.配置/etc/xinetd.d/rsync


	# vi /etc/xinetd.d/rsync

修改以下内容：


	disable = yes   =>  disable = no

2.配置/etc/rsyncd.conf（配置文件）


	# vi /etc/rsyncd.conf

配置以下内容：

	log file = /var/rsyncd.log
	pid file = /var/rsyncd.pid
	lock file = /var/rsyncd.lock
	secrets file = /etc/rsyncd.pas
	motd file = /etc/rsyncd.motd
	read only = no
	hosts allow = 192.168.0.0/16,192.168.10.0/24
	list = yes
	uid = root
	gid = root
	use chroot = no
	max connections = 30
	[bak]
	path = /home/bak/testSync
	comment = www bak
	auth users = root

ps：具体配置说明见[官网]()或附录A。

3.配置/etc/rsyncd.pas（密码文件），并修改文件权限


	# vi /etc/rsyncd.pas

配置以下内容：


	username:passwd

修改文件权限：


	# chmod 600 /etc/rsyncd.pas

4 配置/etc/rsyncd.motd（欢迎信息，内容可为空）


	# vi /etc/rsyncd.motd

5.启动


 	# /usr/bin/rsync --daemon

6.验证启动


	# ps -ef | grep rsync|grep -v grep

## 客户端配置

1.添加用户（管理本地目录）


	# useradd -s /sbin/nologin -M rsync
	# id rsync

2.生成密码文件


	# echo "hkrt" > /etc/rsync.password
	# cat /etc/rsync.password

3.配置密码文件权限

	# chmod 600 /etc/rsync.password

## 同步测试

	# rsync -avz /backup/ rsync_backup@IP::backup/ --password-file=/etc/rsync.password 

## 开机自启动

	# echo "/usr/bin/rsync --daemon" >> /etc/rc.local
	# tail -1 /etc/rc.local

## 使用

共享模块名:在配置文件下，配置的模块名称([XXX]内的名称)

### 本地备份

本地文件系统上实现同步。

	rsync [OPTION] SRC [DEST]

拷贝本地文件。当SRC和DES路径信息都不包含有单个冒号”:”分隔符时就启动这种工作模式。如：rsync -a /data /backup；

### Shell通道

本地主机使用远程shell和远程主机通信。

下载：

	rsync [OPTION] [USER@]HOST:SRC [DEST]

将本地机器的内容拷贝到远程机器。当DST路径地址包含单个冒号”:”分隔符时启动该模式。如：rsync -avz *.c foo:src

上传：

	rsync [OPTION] SRC [USER@]HOST:DEST

将远程机器的内容拷贝到本地机器。当SRC地址路径包含单个冒号”:”分隔符时启动该模式。如：rsync -avz foo:src/bar /data

### Rsync通道
 
本地主机通过网络套接字连接远程主机上的rsync daemon:

下载：

	rsync [OPTION] [USER@]HOST::SRC. [DEST]
	rsync [OPTION] rsync://[USER@]HOST[:PORT]/SRC [DEST]

上传：
	rsync [OPTION] SRC [USER@]HOST::DEST
	rsync [OPTION] SRC rsync://[USER@]HOST[:PORT]/DEST


### 注意事项

1. 如果仅有一个SRC或DEST参数，则将以类似于"ls -l"的方式列出源文件列表(只有一个路径参数，总会认为是源文件)，而不是复制文件。
2. 源路径如果是一个目录的话，带上尾随斜线和不带尾随斜线是不一样的，不带尾随斜线表示的是整个目录包括目录本身，带上尾随斜线表示的是目录中的文件，不包括目录本身。

---

# 附录A rsyncd.conf文件参数说明

## 全局参数

在全局参数部分也可以定义模块参数，这时该参数的值就是所有模块的默认值

|参数|说明|
|:--|:--|
|address|独立运行时，用于指定服务器运行的 IP 地址，默认本地所有IP|
|port|指定 rsync 守护进程监听的端口号，默认873|
|pid file|rsync的守护进程将其 PID 写入指定的文件|
|log file|rsync守护进程的日志文件，而不将日志发送给 syslog|
|syslog facility|指定 rsync 发送日志消息给 syslog 时的消息级别|
|socket options|指定自定义 TCP 选项|
|lockfile|指定rsync的锁文件存放路径|
|timeout|超时时间|

## 模块参数

模块参数主要用于定义 rsync 服务器哪个目录要被同步。模块声明的格式必须为 [module] 形式，这个名字就是在 rsync 客户端看到的名字。服务器真正同步的数据是通过 path 来指定的。

### 模块基本参数
|参数|说明|
|:--|:--|
|path|指定当前模块的同步路径，**必须指定**|
|comment|给模块指定一个描述|

### 模块控制参数

|参数|说明|
|:--|:--|
|use chroot|在服务运行时要不要把他锁定在家目录，默认为 true|
|uid和gid|指定rsync运行用户和用户组，默认nobody|
|use chroot|是否让进程离开工作目录|
|max connections|最大并发连接数，0为不限制|
|lock file|指定支持 max connections的锁文件。默认/var/run/rsyncd.lock|
|list|指定列出模块列表时，该模块是否被列出。默认为 true|
|read only|只读选择，默认true|
|write only|只写选择，不让客户端从服务器上下载文件。默认false|
|ignore errors|忽略IO错误，默认true|
|ignore nonreadable|指定 rysnc 服务器完全忽略那些用户没有访问权限的文件|
|dont compress|用来指定那些在传输之前不进行压缩处理的文件|

### 模块文件筛选参数

|参数|说明|
|:--|:--|
|exclude|由空格隔开的多个文件或目录(相对路径)，并将其添加到 exclude列表中|
|exclude from|指定包含 exclude 规则定义的文件名，服务器从中读取 exclude 列表定义|
|include|指定多个由空格隔开文件或目录(相对路径)，并将其添加到 include 列表中|
|include from|指定一个包含 include 规则定义的文件名，服务器从 中读取 include 列表定义|

### 模块用户认证参数

|参数|说明|
|:--|:--|
|auth users|执行数据同步的用户名，可以设置多个，用英文状态下逗号隔开，默认为匿名方式|
|secrets file|指定一个 rsync 认证口令文件。只有在 auth users 被定义时，该文件才起作用。**文件权限必须是 600**|
|strict modes|表示是否工作在严格模式下，严格检查文件权限等相关信息，默认为true|

### 模块访问控制参数

|参数|说明|
|:--|:--|
|hosts allow|指定哪些主机客户允许连接该模块。默认值为 *|
|hosts deny|指定哪些主机客户不允许连接该模块|

说明：
* 二者都不出现时，默认为允许访问
* 只出现hosts allow，定义白名单，但没有被匹配到的主机由默认规则处理，即为允许
* 只出现hosts deny，定义黑名单，出现在名单中的都被拒绝
* 二者同时出现，先检查hosts allow,如果匹配就allow，否则检查hosts deny，如果匹配则拒绝，如是二者都不匹配，则由默认规则处理，即为允许

### 模块日志参数

|参数|说明|
|:--|:--|
|transfer logging|使 rsync 服务器将传输操作记录到传输日志文件。默认值为false|
|log format|指定传输日志文件的字段。默认为：”%o %h [%a] %m (%u) %f %l”|

注意：
* 设置了”log file”参数时，在日志每行的开始会添加”%t [%p]“；
* 可以使用的日志格式定义符如下所示：

|定义符|意义|定义符|意义|
|:--|:--|:--|:--|
|%o|操作类型：”send” 或 “recv”|%h|远程主机名|
|%a|远程IP地址|%m|模块名|
|%u|证的用户名，匿名时是null|%f|文件名|
|%l|文件长度字符数|%p|该次rsync会话的PID|
|%P|模块路径|%t|当前时间|
|%b|实际传输的字节数|%c|当发送文件时，记录该文件的校验码|


# 附录B rsync指令参数

下面为参数列表，参数含义翻译，可参考<http://www.cnblogs.com/f-ck-need-u/p/7221713.html>

|一级参数|二级参数|描述|
|:--|:--|:--|
|-v|--verbose|increase verbosity|
|-q|--quiet|suppress non-error messages|
|-|--no-motd|suppress daemon-mode MOTD (see manpage caveat)|
|-c|--checksum|skip based on checksum, not mod-time & size|
|-a|--archive|archive mode; equals -rlptgoD (no -H,-A,-X)|
|-|--no-OPTION|turn off an implied OPTION (e.g. --no-D)|
|-r|--recursive|recurse into directories|
|-R|--relative|use relative path names|
|-|--no-implied-dirs|don't send implied dirs with --relative|
|-b|--backup|make backups (see --suffix & --backup-dir)|
|-|--backup-dir=DIR|make backups into hierarchy based in DIR|
|-|--suffix=SUFFIX|set backup suffix (default ~ w/o --backup-dir)|
|-u|--update|skip files that are newer on the receiver|
|-|--inplace|update destination files in-place (SEE MAN PAGE)|
|-|--append|append data onto shorter files|
|-|--append-verify|like --append, but with old data in file checksum|
|-d|--dirs|transfer directories without recursing|
|-l|--links|copy symlinks as symlinks|
|-L|--copy-links|transform symlink into referent file/dir|
|-|--copy-unsafe-links|only "unsafe" symlinks are transformed|
|-|--safe-links|ignore symlinks that point outside the source tree|
|-k|--copy-dirlinks|transform symlink to a dir into referent dir|
|-K|--keep-dirlinks|treat symlinked dir on receiver as dir|
|-H|--hard-links|preserve hard links|
|-p|--perms|preserve permissions|
|-E|--executability|preserve the file's executability|
|-|--chmod=CHMOD|affect file and/or directory permissions
|-A|--acls|preserve ACLs (implies --perms)|
|-o|--owner|preserve owner (super-user only)|
|-g|--group|preserve group|
|-|--devices|preserve device files (super-user only)|
|-|--specials| preserve special files|
|-D|-|same as --devices --specials|
|-t|--times|preserve modification times|
|-O|--omit-dir-times|omit directories from --times|
|-|--super|receiver attempts super-user activities|
|-S|--sparse|handle sparse files efficiently|
|-n|--dry-run|perform a trial run with no changes made|
|-W|--whole-file|copy files whole (without delta-xfer algorithm)|
|-x|--one-file-system|don't cross filesystem boundaries|
|-B|--block-size=SIZE|force a fixed checksum block-size|
|-e|--rsh=COMMAND|specify the remote shell to use|
|-|--rsync-path=PROGRAM|specify the rsync to run on the remote machine|
|-|--existing| skip creating new files on receiver|
|-|--ignore-existing|skip updating files that already exist on receiver|
|-|--remove-source-files|sender removes synchronized files (non-dirs)|
|-|--del|an alias for --delete-during|
|-|--delete|delete extraneous files from destination dirs|
|-|--delete-before|receiver deletes before transfer, not during|
|-|--delete-during|receiver deletes during transfer (default)|
|-|--delete-delay|find deletions during, delete after|
|-|--delete-after| receiver deletes after transfer, not during|
|-|--delete-excluded|also delete excluded files from destination dirs|
|-|--ignore-errors|delete even if there are I/O errors|
|-|--force|force deletion of directories even if not empty|
|-|--max-delete=NUM|don't delete more than NUM files|
|-|--max-size=SIZE|don't transfer any file larger than SIZE|
|-|--min-size=SIZE|don't transfer any file smaller than SIZE|
|-|--partial|keep partially transferred files|
|-|--partial-dir=DIR|put a partially transferred file into DIR|
|-|--delay-updates|put all updated files into place at transfer's end|
|-m|--prune-empty-dirs|prune empty directory chains from the file-list|
|-|--numeric-ids|don't map uid/gid values by user/group name|
|-|--timeout=SECONDS|set I/O timeout in seconds|
|-|--contimeout=SECONDS|set daemon connection timeout in seconds|
|-I|--ignore-times|don't skip files that match in size and mod-time|
|-|--size-only|skip files that match in size|
|-|--modify-window=NUM|compare mod-times with reduced accuracy|
|-T|--temp-dir=DIR|create temporary files in directory DIR|
|-y|--fuzzy|find similar file for basis if no dest file|
|-|--compare-dest=DIR|also compare destination files relative to DIR|
|-|--copy-dest=DIR|... and include copies of unchanged files|
|-|--link-dest=DIR|hardlink to files in DIR when unchanged|
|-z|--compress|compress file data during the transfer|
|-|--compress-level=NUM|explicitly set compression level|
|-|--skip-compress=LIST|kip compressing files with a suffix in LIST|
|-C|--cvs-exclude|auto-ignore files the same way CVS does|
|-f|--filter=RULE|add a file-filtering RULE|
|-F|-|same as --filter='dir-merge /.rsync-filter' repeated: --filter='- .rsync-filter'|
|-|--exclude=PATTERN |exclude files matching PATTERN|
|-|--exclude-from=FILE|read exclude patterns from FILE|
|-|--include=PATTERN|don't exclude files matching PATTERN|
|-|--include-from=FILE|read include patterns from FILE|
|-|--files-from=FILE|read list of source-file names from FILE|
|-0|--from0|all *-from/filter files are delimited by 0s|
|-s|--protect-args|no space-splitting; only wildcard special-chars|
|-|--address=ADDRESS |bind address for outgoing socket to daemon|
|-|--port=PORT|specify double-colon alternate port number|
|-|--sockopts=OPTIONS|specify custom TCP options
|-|--blocking-io|use blocking I/O for the remote shell|
|-|--stats|give some file-transfer stats|
|-8|--8-bit-output|leave high-bit chars unescaped in output|
|-h|--human-readable|output numbers in a human-readable format|
|-|--progress|show progress during transfer|
|-P|-|same as --partial --progress|
|-i|--itemize-changes|output a change-summary for all updates|
|-|--out-format=FORMAT|output updates using the specified FORMAT|
|-|--log-file=FILE|log what we're doing to the specified FILE|
|-|--log-file-format=FMT|log updates using the specified FMT|
|-|--password-file=FILE|read daemon-access password from FILE|
|-|--list-only|list the files instead of copying them|
|-|--bwlimit=KBPS|limit I/O bandwidth; KBytes per second|
|-|--stop-at=y-m-dTh:m|Stop rsync at year-month-dayThour:minute|
|-|--time-limit=MINS|Stop rsync after MINS minutes have elapsed|
|-| --write-batch=FILE|write a batched update to FILE|
|-|--only-write-batch=FILE|like --write-batch but w/o updating destination|
|-|--read-batch=FILE|read a batched update from FILE|
|-|--protocol=NUM |force an older protocol version to be used|
|-|--iconv=CONVERT_SPEC|request charset conversion of filenames|
|-|--tr=BAD/GOOD|transliterate filenames|
|-4|--ipv4|prefer IPv4|
|-6|--ipv6|prefer IPv6|
|-|--version|print version number|
|-h|--help|show this help (-h works with no other options)|

***

<small>2018-09-08 19:32  Cyalud  JinJiang</small>