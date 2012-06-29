---
layout: post
title: "Facebook Menu的Android实现"
date: 2012-06-27 18:20
comments: true
categories: 
---
这段时间一直再找类似path menu（不是圆形的那种）的android 实现，但是搜了一圈都没有。今天天放提醒，那个原来叫Facebook Menu，立马去搜，果然google第一页就有[答案](http://stackoverflow.com/questions/8657894/android-facebook-style-slide)。但答案了有三个demo，我一一下载，测试。发现只有[korovyansk/android-fb-like-slideout-navigation](https://github.com/korovyansk/android-fb-like-slideout-navigation)版本是我想要的。而另外就显得太简陋了，只是实现了滑动的效果，把menu的部分和内容的部分全都放在了一个Activiy里面。而我想要的是

1. 符合Android UI结构的，即通过menu点击之后应该是切换到不同的activity，而不是切换view。
2. Menu是独立的，不应该让我在每一个activity里面都加入menu的那一段东西。

于是便fork了[korovyansk/android-fb-like-slideout-navigation](https://github.com/korovyansk/android-fb-like-slideout-navigation)代码，在他的基础上又添加了几个东西：
1. 支持包含titlebar的布局（ Support titlebar layout ）
2. 支持多个activity切换 (Support multiple activities)

不多说，看图，看源码
{% img http://i.stack.imgur.com/hbZD7.jpg %}
===
###源码地址
github：[https://github.com/huangxiangdan/android-fb-like-slideout-navigation](https://github.com/huangxiangdan/android-fb-like-slideout-navigation)