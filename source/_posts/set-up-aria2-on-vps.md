title: aria2+yaaw离线下载
date: 2016-09-13 00:49:21
tags: ubuntu
categories: 网络
---
充分利用VPS大硬盘、大流量进行BT/磁力/HTTP下载，并实现web资源管理器：文件上传下载，一举两得。
<!-- more -->
> System: ubuntu 16.04 x64
> RAM: 256M
> Reverse proxy: nginx 1.10.1

## aria2

最简单的apt

	apt install aria2
	
新建一个拥有者为www-data的目录，用于存放aria2的设置，初始化某些文件，这里选用/var/www/downloads作为下载默认目录
	
	mkdir /home/www-data
	mkdir /home/www-data/aria2
	mkdir /var/www/downloads
	
	cd /home/www-data/aria2
	touch aria2.session
	
为了安全起见，不用root权限运行aria2。使用以下配置文件启动，将文件内容保为```aria2.conf```

	# HTTPS证书。如果不需要https，可以删掉下面三行
	rpc-certificate=/home/www-data/aria2/signed.crt
	rpc-private-key=/home/www-data/aria2/domain.key
	rpc-secure

	## log默认值输出到stdout
	log=/dev/null

	## 文件保存相关 ##

	# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
	# 请先创建该文件夹：mkdir 
	dir=/var/www/downloads
	# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
	disk-cache=32M
	# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
	# 预分配所需时间: none < falloc ? trunc < prealloc
	# falloc和trunc则需要文件系统和内核支持
	# NTFS建议使用falloc, EXT3/4建议trunc, SSD需要指定none
	file-allocation=none
	# 断点续传
	continue=true

	## 下载连接相关 ##

	# 最大同时下载任务数, 运行时可修改, 默认:5
	max-concurrent-downloads=5
	# 同一服务器连接数, 添加时可指定, 默认:1
	max-connection-per-server=2
	# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
	# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
	min-split-size=10M
	# 单个任务最大线程数, 添加时可指定, 默认:5
	split=8
	# 整体下载速度限制, 运行时可修改, 默认:0
	#max-overall-download-limit=0
	# 单个任务下载速度限制, 默认:0
	#max-download-limit=0
	# 整体上传速度限制, 运行时可修改, 默认:0
	#max-overall-upload-limit=0
	# 单个任务上传速度限制, 默认:0
	#max-upload-limit=0
	# 禁用IPv6, 默认:false
	disable-ipv6=false

	## 进度保存相关 ##

	# 从会话文件中读取下载任务
	input-file=/home/www-data/aria2/aria2.session
	# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
	save-session=/home/www-data/aria2/aria2.session
	# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
	save-session-interval=10

	## RPC相关设置 ##

	# 启用RPC, 默认:false
	enable-rpc=true
	# 允许所有来源, 默认:false
	rpc-allow-origin-all=true
	# 允许非外部访问, 默认:false
	rpc-listen-all=true
	# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
	event-poll=select
	# RPC监听端口, 端口被占用时可以修改, 默认:6800
	# rpc-listen-port=6800
	# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
	# rpc-secret=<token>
	# 设置的RPC访问用户名, 此选项新版已废弃, 建议改用 --rpc-secret 选项
	#rpc-user=<USER>
	# 设置的RPC访问密码, 此选项新版已废弃, 建议改用 --rpc-secret 选项
	#rpc-passwd=<PASSWD>

	## BT/PT下载相关 ##

	# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
	follow-torrent=true
	# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
	listen-port=51413
	# 单个种子最大连接数, 默认:55
	#bt-max-peers=55
	# 打开DHT功能, PT需要禁用, 默认:true
	enable-dht=false
	# 打开IPv6 DHT功能, PT需要禁用
	#enable-dht6=false
	# DHT网络监听端口, 默认:6881-6999
	#dht-listen-port=6881-6999
	# 本地节点查找, PT需要禁用, 默认:false
	#bt-enable-lpd=false
	# 种子交换, PT需要禁用, 默认:true
	enable-peer-exchange=false
	# 每个种子限速, 对少种的PT很有用, 默认:50K
	#bt-request-peer-speed-limit=50K
	# 客户端伪装, PT需要
	peer-id-prefix=-TR2770-
	user-agent=Transmission/2.77
	# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
	seed-ratio=0.2
	# 强制保存会话, 即使任务已经完成, 默认:false
	# 较新的版本开启后会在任务完成后依然保留.aria2文件
	force-save=true
	# BT校验相关, 默认:true
	#bt-hash-check-seed=true
	# 继续之前的BT任务时, 无需再次校验, 默认:false
	#bt-seed-unverified=true
	# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
	bt-save-metadata=true


修改完后更改拥有者www-data

	chown -R www-data:www-data /home/www-data/aria2
	chown -R www-data:www-data /var/www/downloads
	
测试一下使用该文件能否启动

	aria2c --conf-path=/home/www-data/aria2/aria2.conf
	
![](/images/aria2/aria2.png)
	
写入init.d脚本，位置为/etc/init.d/aria2

	#!/bin/sh
	# https://gist.github.com/jereksel/8217470
	
	USER="www-data"
	DAEMON=/usr/bin/aria2c
	CONF="/home/www-data/aria2/aria2.conf"

	start() {
		if [ -f $CONF ]; then
			echo "Starting aria2 daemon..."
			start-stop-daemon -S -c $USER:$USER -x $DAEMON -- -D --conf-path=$CONF || echo "start fail"
			pid=`pgrep -fu $USER $DAEMON`
			echo "pid=$pid"
		else
			echo "$CONF was not found"
		fi
	}

	stop() {
		echo  "Stopping..."
		start-stop-daemon -K -c $USER:$USER -x $DAEMON
		if [ $? = "0" ];then
			sleep 1
			echo "stop ok"
		else
			echo "stop fail"
		fi
	}

	case "$1" in
		start)
			start;;
		stop)
			stop;;
		restart)
			stop
			start;;
		*)
			echo "Usage: /etc/init.d/aria2 {start|stop|restart}"
			exit 1
	esac

添加可执行权限

	chmod a+x /etc/init.d/aria2
	
测试一下能否启动

	killall aria2c
	/etc/init.d/aria2 start
	
可以加入到系统自动启动

	vi /etc/rc.local
	# 添加 /etc/init.d/aria2 start
	
## Yaaw

这个是配合aria2实现web前端控制，纯静态html页面，在客户端javscript实现所有功能。

	cd /var/www/
	git clone https://github.com/binux/yaaw
	
在nginx.conf中的server标签中添加合适的location，指向index.html。实现访问xxx.com/yaaw即可打开页面

	location ^~ /yaaw {
		alias /var/www/yaaw/;
	}

重载nginx，测试一下网站能否打开

	nginx -s reload
	
![](/images/aria2/yaaw.png)
	
第一次会出现Internal server error，正常，因为没有设置RPC地址。在网站右侧“设置”修改jsonrpc为正确的地址，正确值应该是

	http://xxx.com:6800/jsonrpc

修改正确就可以登陆服务器实现下载文件到vps了，更多设置参考[yaaw设置帮助](http://aria2c.com/usage.html)
	
## eXtplorer

光有下载还不行，还需要文件管理。实现web界面的删除文件/下载/上传等基本功能。

	apt install php php-cgi php-json php-xml php-mbstring

下载[eXtplorer](http://extplorer.net/projects/extplorer/files)，解压到/var/www/eXtplorer目录

同样为了安全起见，使用较低权限的www-data

	chown -R www-data:www-data /var/www/eXtplorer

在nginx添加一个virtual-server

	vi nginx.conf
	# http标签内增加:
	
	fastcgi_buffers 8 16k;
	fastcgi_buffer_size 32k;
	fastcgi_connect_timeout 300;
	fastcgi_send_timeout 300;
	fastcgi_read_timeout 300;
	

	vi sites-enabled/defualt
	# server标签增加:
	
	root /var/www/eXtplorer/;
	index index.php index.html index.htm;

	location / {
		try_files $uri $uri/ /index.html;
	}

	location ~ \.php$ {
		fastcgi_pass unix:/run/php/php7.0-fpm.sock;
		fastcgi_index index.php;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
	
遇到的各种502 Bad Gateway错误、404错误请自己解决。一般都是权限问题，只要保证```php-fpm的执行者```与```nginx的执行者```都是```www-data```就不会出错。

重启nginx服务，打开浏览器看看能不能进入界面。这样就可以实网页端管理文件。

貌似第一次进入的密码、帐号都是admin，及时更改密码，如无法更改密码请核实eXtplorer拥有者权限。

![](/images/aria2/eXtplorer.png)

把aria2下载的东西全部在这里eXtplorer下载到自己电脑，还是很不错的

至于上传文件功能，eXtplorer已经集成了。最好设置一下默认最大允许上传文件大小，否则上传超大文件提示413错误:```Request Entity Too Large```

nginx部份

	vi /etc/nginx/nginx.conf
	# Content-Length的最大值限制，默认1MB，这里改为1GB
	client_max_body_size 1024m;

php部份

	vi /etc/php/7.0/fpm/php.ini
	# 是否允许通过HTTP上传文件的开关。默认为ON即是开
	file_uploads = on
	# 允许上传文件大小的最大值。默认为2M
	upload_max_filesize = 100m 望文生意，即
	# 表单POST给PHP的所能接收的最大值，包括表单里的所有值。默认为8M
	post_max_size = 8m
	# 传至服务器上存储临时文件的地方，如果没指定就会用系统默认的临时文件夹
	upload_tmp_dir = /home/tmp_upload

一般来说，设置好上述四个参数后，在网络正常的情况下，上传<=8M的文件是不成问题的

但如果要上传>8M的大文件的话，只设置上述四项还不一定能行的通。除非你的网络真有100Mbps的上传高速，否则你还得继续设置下面的参数。

	# 每个PHP页面运行的最大时间值(秒)，默认30秒
	max_execution_time 600
	# 每个PHP页面接收数据所需的最大时间，默认60秒
	max_input_time 600
	# 每个PHP页面所吃掉的最大内存，默认8M
	memory_limit 8m

这样就可以使用自己的私有云了！远离国产云存储服务！