---
layout: post
title: 'mysql之突破secure_file_priv写shell'
subtitle: ''
date: 2021-05-25
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: 网络安全
---
### mysql之突破secure_file_priv写shell
在某些情况下，当我们进入了一个网站的phpMyAdmin时，想通过select into outfile来写shell，但是通常都会报错。
![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell1.png)
这是因为在mysql 5.6.34版本以后 secure_file_priv的值默认为NULL。并且无法用sql语句对其进行修改，只能够通过以下方式修改

windows下:

修改mysql.ini 文件，在[mysqld] 下添加条目: secure_file_priv =

保存，重启mysql。

Linux下:

在/etc/my.cnf的[mysqld]下面添加local-infile=0选项。

 

但是如果这样的话我们就不能够写shell了吗？答案是否定的。

> 日志的利用
在php文件包含漏洞中我们有过这种利用方法:攻击时进行一次附带代码的url请求,在日志记录这次请求之后，通过文件包含漏洞来包含这个日志文件，从而执行请求中的代码。

 

mysql也具有日志，我们也可以通过日志来getshell。

mysql日志主要包含:错误日志、查询日志、慢查询日志、事务日志，日志的详细情况参考mysql日志详细解析

我们主要利用慢查询日志来写shell,步骤大致分为三步:

> 1.设置slow_query_log=1.即启用慢查询日志(默认禁用)。

set global slow_query_log=1;

![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell2.png)

> 2.伪造(修改)slow_query_log_file日志文件的绝对路径以及文件名

![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell3.png)

3.向日志文件写入shell

![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell4.png)

执行成功
![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell5.png)
![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell6.png)

对慢查询日志的补充:
因为是用的慢查询日志，所以说只有当查询语句执行的时间要超过系统默认的时间时,该语句才会被记入进慢查询日志。

时间默认超过多少的称为慢查询日志？

一般都是通过long_query_time选项来设置这个时间值，时间以秒为单位，可以精确到微秒。如果查询时间超过了这个时间值（默认为10秒），这个查询语句将被记录到慢查询日志中。查看服务器默认时间值方式如下：

show global variables like '%long_query_time%'

![](https://1024861435.github.io/assets/img/mysql之突破secure_file_priv写shell7.png)

通常情况下执行sql语句时的执行时间一般不会超过10s，所以说这个日志文件应该是比较小的，而且默认也是禁用状态，不会引起管理员的察觉。

参考文章：

[mysql日志详细解析](http://blog.51cto.com/pangge/1319304)

[MySQL慢查询日志总结](http://www.cnblogs.com/kerrycode/p/5593204.html)