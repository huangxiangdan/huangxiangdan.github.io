---
layout: post
title: "配置munin插件"
date: 2013-10-18 11:50
comments: true
categories: 
---
接上文
##安装ejabberd插件
可参考[http://dev.groupdock.com/2010/10/05/monitoring-ejabberd-with-munin.html](http://dev.groupdock.com/2010/10/05/monitoring-ejabberd-with-munin.html)
具体步骤是：

1. 创建具体监控的软链接
		
		sudo ln -s /usr/share/munin/plugins/ejabberd_ /etc/munin/plugins/ejabberd_users

2. 配置ejabberd配置文件(/etc/munin/plugin-conf.d/munin-node.conf)

		[ejabberd_users]
		user root
		
		[ejabberd_*]
		env.vhosts yourhost.com
3. 可通过sudo munin-run ejabberd_users（因为你的users是root，所以必须用sudo）进行测试
4. 理论上这样就可以搞定了，但因为插件的更新速度更不上ejabberd的更新速度，所有很有可能不能用。以ejabberd_users为例，实际上munin只需要你运行/sbin/ejabberdctl connected_users_number，然后把值传给它，所以如果碰到问题，可以查看ejabberd_的代码，修改成合适的样子。
##监控某个端口的连接数
		ln -s /usr/share/munin/plugins/port_ /etc/munin/plugins/port_5222

##自定义munin插件
可参考[http://munin-monitoring.org/wiki/HowToWritePlugins](http://munin-monitoring.org/wiki/HowToWritePlugins)

就像下面这个样子（你可以用任意语言来写），你就可以搞定了
		
	#!/bin/sh

    case $1 in
       config)
            cat <<'EOM'
    graph_title Load average
    graph_vlabel load
    load.label load
    EOM
            exit 0;;
    esac

    echo -n "load.value "
    cut -d' ' -f2  /proc/loadavg
    
如果有权限问题，需要用root运行的话，可以在/etc/munin/plugin-conf.d/munin-node.conf中配置好所需要的用户，比如

	[ejabberd_users]
	user root
	timeout 60

如果你的命令需要很长时间的话可以加上timeout 60这样的设置（参考[http://munin-monitoring.org/wiki/plugin-conf.d](http://munin-monitoring.org/wiki/plugin-conf.d)）

	




