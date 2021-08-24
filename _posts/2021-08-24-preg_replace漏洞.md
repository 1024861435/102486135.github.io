---
layout: post
title: 'preg_replace /e漏洞'
subtitle: 'preg_replace /e 模式下的代码执行'
date: 2021-08-22
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: 网络安全
---

### 前言

本文将深入研究 preg_replace /e 模式下的代码执行问题，其中包括 preg_replace 函数的执行过程分析、正则表达式分析、漏洞触发分析，当中的坑非常多，相信看完本文，你一定会有所收获。下面是 七月火 和 l1nk3r的分析结果。

### 案例

下面先看一个案例，思考如何利用此处的 preg_replace /e 模式，执行代码（可以先不看下文分析，自己思考出 payload 试试）。

	header("Content-Type: text/plain");
	 
	function complexStrtolower( $regex, $value){
	    return preg_replace('/('. $regex.')/ei','strtolower("\1")',$value);
	}
	 
	foreach ($_GET as $regex => $value){
	    echo complexStrtolower($regex, $value)."n";
	}

这个案例实际上很简单，就是 preg_replace 使用了 /e 模式，导致可以代码执行，而且该函数的第一个和第三个参数都是我们可以控制的。我们都知道， preg_replace 函数在匹配到符号正则的字符串时，会将替换字符串（也就是上面代码中 preg_replace 函数的第二个参数）当做代码来执行，然而这里的第二个参数却固定为 'strtolower("\1")' 字符串，那这样要如何执行代码呢？

上面的命令执行，相当于 eval('strtolower("\\1");') 结果，当中的 \\1 实际上就是 \1 ，而 \1 在正则表达式中有自己的含义。我们来看看 W3Cschool 中对其的描述：

### 反向引用：

对一个正则表达式模式或部分模式 两边添加圆括号 将导致相关 匹配存储到一个临时缓冲区 中，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序存储。缓冲区编号从 1 开始，最多可存储 99 个捕获的子表达式。每个缓冲区都可以使用 '\n' 访问，其中 n 为一个标识特定缓冲区的一位或两位十进制数。

所以这里的 \1 实际上指定的是第一个子匹配项，我们拿 ripstech 官方给的 payload 进行分析，方便大家理解。官方 payload 为： /?.*={${phpinfo()}} ，即 GET 方式传入的参数名为 /?.* ，值为 {${phpinfo()}} 。
	
	原先的语句： preg_replace('/(' . $regex . ')/ei', 'strtolower("\1")', $value);
	变成了语句： preg_replace('/(.*)/ei', 'strtolower("\1")', {${phpinfo()}});

上面的 preg_replace 语句如果直接写在程序里面，当然可以成功执行 phpinfo() ，然而我们的 .* 是通过 GET方式传入，你会发现无法执行 phpinfo 函数

我们 var_dump 一下 $_GET 数组，会发现我们传上去的 .* 变成了 _* ，如下图所示：

这是由于在PHP中，对于传入的非法的 $_GET 数组参数名，会将其转换成下划线，这就导致我们正则匹配失效。我们可以 fuzz 一下PHP会将哪些符号替换成下划线，发现有：（这是非法字符不为首字母的情况）

当非法字符为首字母时，只有点号会被替换成下划线：

所以我们要做的就是换一个正则表达式，让其匹配到 {${phpinfo()}} 即可执行 phpinfo 函数。这里我提供一个 payload ： \S*=${phpinfo()} 执行结果如下：

![](https://1024861435.github.io/assets/img/Preg_Replace代码执行漏洞1.png)

下面再说说我们为什么要匹配到 {${phpinfo()}} 或者 ${phpinfo()} ，才能执行 phpinfo 函数，这是一个小坑。这实际上是 PHP可变变量 的原因。在PHP中双引号包裹的字符串中可以解析变量，而单引号则不行。 ${phpinfo()} 中的 phpinfo() 会被当做变量先执行，执行后，即变成 ${1} (phpinfo()成功执行返回true)。如果这个理解了，你就能明白下面这个问题：

	var_dump(phpinfo()); // 结果：布尔 true
	var_dump(strtolower(phpinfo()));// 结果：字符串 '1'
	var_dump(preg_replace('/(.*)/ie','1','{${phpinfo()}}'));// 结果：字符串'11'
	var_dump(preg_replace('/(.*)/ie','strtolower("\1")','{${phpinfo()}}'));// 结果：空字符串''
	var_dump(preg_replace('/(.*)/ie','strtolower("{${phpinfo()}}")','{${phpinfo()}}'));// 结果：空字符串''
	这里的'strtolower("{${phpinfo()}}")'执行后相当于 strtolower("{${1}}") 又相当于 strtolower("{null}") 又相当于 '' 空字符串

### 问题

不过从php5.50 后/e 修饰符已经被弃用了。而7.0.0 不再支持 /e修饰符。PHP 5.5.0 起， 传入 "e" 修饰符的时候，会产生一个 E_DEPRECATED 错误； PHP 7.0.0 起，会产生 E_WARNING 错误，同时 "e" 也无法起效。

php5.5以后的版本要用的话请用 preg_replace_callback() 代替。


原文地址：https://www.cesafe.com/html/6999.html&&https://www.xinyueseo.com/websecurity/158.html