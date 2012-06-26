---
layout: post
title: "ejabberd部署"
description: ""
category: 
comments: true
tags: []
---
我们选用ejabberd作为我们聊天的Server，原因是可以1、erlang性能高2、用得人多。 
经过两天的折腾，我终于在三台不同的系统上都装上了ejabberd，分别讲讲这三个平台的安装经历。

##Mac
ejabberd有Mac下的dmg安装包，有可以通过source安装。

1. 我的第一次尝试是直接使用了ejabberd的dmg安装包，结果执行postinstall脚本的时候出错，服务起不来。没有具体的错误提示。
2. 所以换用source安装的方式。先装的erlang 15B01，然后安装ejabberd 2.1.10版本。结果使用/sbin/ejabberdctl start 没有任何反应。通过netstat看发现没有5280端口。于是换ejabberdctl live命令，发现提示是端口冲突。后来想起来我装过openfire，于是卸载掉，重新启动，还是出错。
3. 后来发现erlang 15B01出的比ejabberd2.1.10还晚，怀疑是版本问题。于是卸载erlang 15B01，换用erlang 15B。
4. 装好之后，重新安装ejabberd，好像就没什么大问题了。
5. 启动访问http://localhost:5280/admin。是否能出来页面。

##Cent OS
直接用source安装的，安装完之后没什么问题，有问题可以参考下面的资料。

##Ubuntu  10.10

1. 我首先是直接用apt-get安装的erlang，结果一看，版本是5.9.1就是15B01，所以就卸载掉了。
2. 后来手动安装erlang失败，发现缺少依赖库
3. 所以必须先安装下面的东西
4. sudoapt-get -yinstall\
        build-essential m4 libncurses5-dev \
        libssh-dev unixodbc-dev libgmp3-dev \
        libwxgtk2.8-dev libglu1-mesa-dev \
        fop xsltproc default-jdk
5. 安装完erlang后，又手动安装了ejabberd，结果启动总是不成功。
6. 后又换apt-get安装ejabberd，启动时提示几个文件有权限问题，我直接chmod 777就搞定了。
5. ubuntu中启动ejabberd是这样启动的，/etc/init.d/ejabberd start。不要用ejabberdctl，会失败。

##排错指南
1. 在start之前检查下epmd, beam, beam.smp等是否启动了，如果启动着，先把他们杀掉，他们是erlang相关的东西。
2. 检查是否有应用占用了5222,5223,5280等端口，5222端口是xmpp协议本身规定的，所以默认是用此端口。如果之前装过其他的XMPP server请停止或卸载，比如OpenFire
3. ejabberd有很重的erlang依赖关系，所以一定要对好版本，参考下面的资料。
4. 如果不需要配置，也是可以启动服务的，所以无法启动不是配置的问题。
5. 如果start没什么错误提示，可以改用status或者live方式启动，这样错误会明显一点，如果还是没有错误提示，只能到/var/log/ejabberd底下去看了。

##参考资料

1. [ejabberd 简明配置](http://blog.sina.com.cn/s/blog_6d2cab390100zop7.html)
2. [ejabberd 简明配置2](http://blog.sina.com.cn/s/blog_6d2cab390100zop7.html)
3. [ejabberd清理](http://usenrong.iteye.com/blog/855430)
3. [ejabberd清理2](http://www.mail-archive.com/server-devel@lists.laptop.org/msg02776.html)
4. [
Installing Erlang/OTP R14B04 on Ubuntu 11.10 from source tarball ](http://boris.muehmer.de/2011/11/09/installing-erlangotp-r14b04-on-ubuntu-11-10-from-source-tarball/)
5. [
E: Sub-process /usr/bin/dpkg returned an error code 解决办法](http://forum.ubuntu.org.cn/viewtopic.php?f=86&t=90547)
6. [odbc : ODBC library – link check failed](http://blog.csdn.net/summerhust/article/details/7201298)
7. [
Debian package difference](http://www.ejabberd.im/node/4917)
8. [installing-erlang-on-mac-os-x](http://digitalsanctum.com/2009/10/01/installing-erlang-on-mac-os-x/)
9. [ejabberd与erlang版本对应关系(重要)](http://www.ejabberd.im/erlang)
10. [ejabberd安装配置指南(不太有用)](http://wiki.jabbercn.org/Ejabberd2:%E5%AE%89%E8%A3%85%E5%92%8C%E6%93%8D%E4%BD%9C%E6%8C%87%E5%8D%97)
11. [
Debugging ejabberd](http://lboynton.com/tag/ejabberd/)
