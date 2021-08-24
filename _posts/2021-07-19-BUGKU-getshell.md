---
layout: post
title: 'BUGKU-getshell'
subtitle: 'BUGKU-getshell'
date: 2021-07-19
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: 网络安全 bugku刷题
---
### BUGKU-getshell

> 打開題目

發現php混淆類代碼

![](https://1024861435.github.io/assets/img/getshell0.png)

> 獲得源碼

方法：不斷取最後eval的值echo 最終規得到源代碼

![](https://1024861435.github.io/assets/img/getshell1.png)

![](https://1024861435.github.io/assets/img/getshell2.png)

![](https://1024861435.github.io/assets/img/getshell3.png)

![](https://1024861435.github.io/assets/img/getshell4.png)

發現shell文件

> 用蟻劍鏈接

![](https://1024861435.github.io/assets/img/getshell5.png)

發現沒有權限訪問其他目錄

![](https://1024861435.github.io/assets/img/getshell6.png)

> 利用蚁剑的"绕过disable_functions"插件检测一下函数

![](https://1024861435.github.io/assets/img/getshell7.png)

发现putenv没有被禁用，linux主机，putenv=on，用LD_PRELOAD方法绕过

![](https://1024861435.github.io/assets/img/getshell8.png)

上次.antproxy.php成功

> 鏈接url/.antproxy.php,訪問目錄

![](https://1024861435.github.io/assets/img/getshell9.png)

![](https://1024861435.github.io/assets/img/getshell10.png)

在根目錄下發現flag文件







