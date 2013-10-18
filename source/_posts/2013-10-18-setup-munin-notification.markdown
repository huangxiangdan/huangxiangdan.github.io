---
layout: post
title: "setup munin notification"
date: 2013-10-18 12:45
comments: true
categories: 
---
可参考[http://munin-monitoring.org/wiki/HowToContact"](http://munin-monitoring.org/wiki/HowToContact)

在munin.conf中添加
	
	contact.me.command mail -s "Munin-notification for ${var:group} :: ${var:host}" xd_huang1986@163.com
	contact.me.always_send warning critical

可修改邮件的内容，添加上下面所列的即可

	contact.me.text  <munin group="${var:group}" host="${var:host}"\
	graph_category="${var:graph_category}" graph_title="${var:graph_title}" >\
	${loop< >:wfields <warning label="${var:label}" value="${var:value}"\
    w="${var:wrange}" c="${var:crange}" extra="${var:extinfo}" /> }\
    ${loop< >:cfields <critical label="${var:label}" value="${var:value}"\
    w="${var:wrange}" c="${var:crange}" extra="${var:extinfo}" /> }\
    ${loop< >:ufields <unknown label="${var:label}" value="${var:value}"\
    w="${var:wrange}" c="${var:crange}" extra="${var:extinfo}" /> }\
    </munin>
    
可以在某一个host上添加上
	
	cpu.user.warning 20
	
进行测试，这样一旦cpu达到20，就会触发报警信息。可在每一个host下加上预警的数值

	[chat;web2]
    address 192.168.1.3
    use_node_name yes
    load.load.warning 6
    load.load.critical 15
    
如果想添加短信提醒，可以开通短信服务，比如[http://ipyy.net/](http://ipyy.net/)。然后在配置文件中添加

	contact.sms.command curl "http://ipyy.net/WS/Send.aspx?xxxxxxxx"
	
xxxxxx可参考短信服务提供的接口。