---
layout: post
title: "setup munin and plugins"
date: 2013-08-27 05:18
comments: true
categories: 
---
#系统环境
Ubuntu 12.04
#安装munin
##安装munin-node
sudo apt-get install munin-node
##安装munin server
sudo apt-get install munin
#基础配置
##munin-node 配置
###配置文件地址
/etc/munin/munin-node.conf
###需要修改的地方
配置文件为 /etc/munin/munin-node.conf 添加上master的ip地址，添加字段为如下，添加地方为 allow ^127\.0\.0\.1$ 的后面。allow支持正则表达式

	allow ^124\.232\.156\.224$
##munin server 配置
###配置文件地址
/etc/munin/munin.conf
###需要修改的地方
添加如下客户端的信息,内容如下,如果有多个node，加入多个node的相关信息即可。web;web1代表的是web组，web1服务器
	
	[web;web1]
    address 127.0.0.1
    use_node_name yes

	[lvs;lvs1]
    address 192.168.1.30
    use_node_name yes
###为munin server配置nginx路径
	server {
      listen 80;
      server_name munin.yitu.me;
      expires off;
      auth_basic "Munin";
      auth_basic_user_file /var/cache/munin/htpasswd;
      root /var/cache/munin/www;
      }
先使用
	
	htpasswd -c /var/cache/munin/htpasswd admin 
命令生成htpasswd文件，如果不存在htpasswd命令，先安装

	apt-get install mini-httpd
打开地址 http://munin.xx.xx(自己在nginx里配置的地址，前提是先把dns转过来) 就可以看到被监控节点的信息了
#munin服务启动
##munin-node
	
	service munin-node start
##munin server
munin server 不需要启动，是通过cron job的方式去抓取信息的，安装成功之后就已经有了，不需要启动。可以通过
	
	munin-check
来检查配置是否正确
#munin 基础知识
1. munin会使用munin这个用户运行munin相关的命令，所以有时候可以通过切换到munin用户debug
		
		su munin --shell=/bin/bash
2. munin用到的路径地址

		# dbdir /var/lib/munin
		# htmldir /var/cache/munin/www
		# logdir /var/log/munin
		# rundir  /var/run/munin
		# plugins /var/etc/munin/plugins



#插件配置
##基础配置增强
	sudo apt-get install munin-plugins-extra
##安装passenger插件
	mkdir munin && cd munin && git clone https://github.com/huangxiangdan/munin-plugins-rails.git
	cd munin-plugins-rails/ && gem install munin-plugins-rails-0.2.12.gem
	rvmsudo request-log-analyzer-munin install
	rvmsudo request-log-analyzer-munin add <app_name> <path_to_log_file>
