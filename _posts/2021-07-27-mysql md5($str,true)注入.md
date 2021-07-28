---
layout: post
title: 'mysql md5($str,true)注入'
subtitle: 'mysql md5($str,true)注入'
date: 2021-07-27
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: jekyll 前端开发 设计
---

### mysql md5($str,true)注入
"select * from admin where passwd = ' " .md5. " ' ";

这样一个SQL语句其实可以注入

先来说一下md5这个函数

md5($str,raw) raw 可选，默认为false

true:返回16字符2进制格式

false:返回32字符16进制格式

简单来说就是 true将16进制的md5转化为字符了,如果某一字符串的md5恰好能够产生如’or ’之类的注入语句，就可以进行注入了.

提供一个字符串：ffifdyop

md5后，276f722736c95d99e921722cf9ed621c

转成字符串后： 'or'6<trash>