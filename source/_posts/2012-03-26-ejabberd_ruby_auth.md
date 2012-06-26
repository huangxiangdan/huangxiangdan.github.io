---
layout: post
title: "ejabberd使用ruby做用户验证（extauth_program）"
description: ""
category: 
tags: []
---
##背景
使用ejabberd做聊天服务，最大的问题是怎么跟现有的系统对接，其中最重要的就是用户系统如何对接。而这一点，ejabberd已经为我们考虑到了，为了方便不会使用erlang的开发者使用自己的用户系统，ejabberd提供了**extauth_program**这个配置，从而可以使用任何可执行的脚本作为用户系统的接口。其中perl语言的demo在ejabberd srouce的example里就有。而更多的配置可以参考[Authentication Scripts](http://www.ejabberd.im/extauth)，你可以使用非常多的语言和数据库，包括perl,python,甚至php，c#。其实你用任何语言都没有关系，因为毕竟只有短短几十行代码，你唯一要做的就是设置好你的数据库参数。

##我的尝试步骤
但是不幸的是官方给的参考里是没有ruby脚本的（有脚本，但是页面已经失效），于是乎我在网上狂搜extauth ruby的实现，还好找到一个据说有点问题的ruby代码[ejabberd not exec my script ](http://www.ejabberd.im/node/919)，那人就是因为执行不了所以才发帖求助的。于是

1. 我copy了他的代码，然后稍作了修改，配置好extauth。果然执行不了，没有任何反应，连我设置的log都没有输出来。而ejabberd的log是**extauth 脚本意外终止，原因是normal**
2. 于是我做了个测试，将所有的代码都删掉，只保留log的部分，结果还是没有log，而我用ruby执行是有log
3. 这让我一度怀疑ejabberd不支持ruby，于是我换用ejabberd source里的perl脚本，结果一切工作正常。
4. 我误认为ejabberd不支持ruby，于是我放弃ruby的方式，在网上down了[Authenticate Against MySQL with Perl](http://www.ejabberd.im/check_mysql_perl)，修改后run，结果发现跟1的结果是一样，无法运行。
5. 既然有一个正确的，一个错误的相同语言的脚本，那我就可以比对哪里导致运行失败了。比对的结果是4下载的sql没有执行权限，于是我chmod a+x，4运行正常了，但我仍然不能正确登录。
6. 在perl console环境中一步步执行4的脚本，发现原因是我缺少sql lib的支持。于是准备换用python脚本。
7. 在修改python脚本支持的时候，我意外发现所有的文件开头都有一行#!/usr/bin/perl这样的注释，我突然明白为什么我的ruby脚本不能执行了，因为它不知道怎么去执行，于是在ruby文件开头加上了#!/usr/bin/env ruby，这个世界顿时清净了。后来想起我好像在鸟哥的私房菜里边见过这些东西，后悔看得太草了。

##要点
1. 你的script本身要能正确执行，即有执行权限，而且是可执行的（第一行有类似#!/usr/bin/env ruby的注释）
2. 正确配置extauth。如下： 
     {auth_method, external}.%%这是要点
     {extauth_program, "/var/lib/ejabberd/db_auth.rb”}.
3. 没有第三了。

##下载
[a ejabberd extauth script write by ruby](https://gist.github.com/2495010)


##参考资料
1. [extauth list](http://www.ejabberd.im/extauth)
2. [ejabberd not exec my script ](http://www.ejabberd.im/node/919)
3. [ejabberd extauth using erlang escript](http://stackoverflow.com/questions/6127284/ejabberd-extauth-using-erlang-escript)
