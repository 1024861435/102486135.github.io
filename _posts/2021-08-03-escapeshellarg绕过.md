---
layout: post
title: 'escapeshellarg绕过'
subtitle: 'DASCTF cat flag'
date: 2021-08-03
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: 网络安全 buu刷题
---
### DASCTF cat flag

> 1.打开题目

![](https://1024861435.github.io/assets/img/escapeshellarg1.png)

发现通过cmd传参可以访问文件，但是正则过滤了'flag'

看到提示

![](https://1024861435.github.io/assets/img/escapeshellarg2.png)

于是想到访问日志文件/var/log/nginx/access.log

> 访问日志文件发现flag的文件名

![](https://1024861435.github.io/assets/img/escapeshellarg3.png)

> 于是构造poc

	?cmd=/this_is_final_fla*_e2a457126032b42d.php

 发现通配符不能绕过escapeshellarg

> 了解escapeshellarg函数

发现escapeshellarg会把cmd的传参加上单引号

为什么通配符不行？

	cat flag ->可以访问
	cat fla* ->可以访问
	cat 'flag' -> 可以访问
	cat ‘fla*' -> 不能访问

查看https://www.php.net/manual/es/function.escapeshellarg.php

![](https://1024861435.github.io/assets/img/escapeshellarg3.png)

发现只有加入不是ascll码的字符就能绕过

> fla和g中间加一个ascii码大于127的字符，进行url编码后，就可绕过正则，然后在遇到escapeshellarg后会自动去除，所以可以正常读取文件，最终payload：

	?cmd=./this_is_final_fla%8ag_e2a457126032b42d.php

