---
layout: post
title: 'flask session伪造'
subtitle: '[HCTF 2018]admin 1'
date: 2021-07-27
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: jekyll 前端开发 设计
---

### [HCTF 2018]admin 1

> 打开题目

![](https://1024861435.github.io/assets/img/flask session伪造1.png)

> 查看源码

![](https://1024861435.github.io/assets/img/flask session伪造2.png)

>发现登录，注册界面，进入登录先试着登录看看（admin，123）！！！发现直接得到flag。

![](https://1024861435.github.io/assets/img/flask session伪造3.png)

以为是一道爆破题，去看了大佬的wp，才知道是flask session伪造,于是按照大佬的思路重新开始

> 注册账号登录，并查看每个界面的源码，在改密码的界面发现有源码文件

![](https://1024861435.github.io/assets/img/flask session伪造4.png)

> 下载源码,代码审计,发现admin.html,知道令session[name]='admin'才能得到flag,于是想到session伪造

![](https://1024861435.github.io/assets/img/flask session伪造5.png)

> 抓到得到session,用GitHub的exp解码,解密需要SECRET_KEY,查阅质料知道SECRET_KEY在config.py

![](https://1024861435.github.io/assets/img/flask session伪造7.png)

![](https://1024861435.github.io/assets/img/flask session伪造6.png)

![](https://1024861435.github.io/assets/img/flask session伪造8.png)

> 把name的值改成admin进行加密

![](https://1024861435.github.io/assets/img/flask session伪造9.png)

> 抓包改session,得到flag

![](https://1024861435.github.io/assets/img/flask session伪造1.png)












