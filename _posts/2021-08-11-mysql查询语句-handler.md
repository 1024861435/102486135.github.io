---
layout: post
title: 'mysql查询语句-handler'
subtitle: 'mysql查询语句-handler'
date: 2021-08-11
categories: 技术
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-theme-h2o-postcover.jpg'
tags: jekyll 前端开发 设计
---

### 简介

mysql除可使用select查询表中的数据，也可使用handler语句，这条语句使我们能够一行一行的浏览一个表中的数据，不过handler语句并不具备select语句的所有功能。它是mysql专用的语句，并没有包含到SQL标准中。

HANDLER语句提供通往表的直接通道的存储引擎接口，可以用于MyISAM和InnoDB表。

### 基本语法

handler语句的语法如下：
	HANDLER tbl_name OPEN [ [AS] alias]
	 
	HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,...)
	    [ WHERE where_condition ] [LIMIT ... ]
	HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST }
	    [ WHERE where_condition ] [LIMIT ... ]
	HANDLER tbl_name READ { FIRST | NEXT }
	    [ WHERE where_condition ] [LIMIT ... ]
	 
	HANDLER tbl_name CLOSE

通过HANDLER tbl_name OPEN打开一张表，无返回结果，实际上我们在这里声明了一个名为tb1_name的句柄。

通过HANDLER tbl_name READ FIRST获取句柄的第一行，通过READ NEXT依次获取其它行。最后一行执行之后再执行NEXT会返回一个空的结果。

通过HANDLER tbl_name CLOSE来关闭打开的句柄。

通过索引去查看的话可以按照一定的顺序，获取表中的数据。

通过HANDLER tbl_name READ index_name FIRST，获取句柄第一行（索引最小的一行），NEXT获取下一行，PREV获取前一行，LAST获取最后一行（索引最大的一行）。

通过索引列指定一个值，可以指定从哪一行开始。

通过HANDLER tbl_name READ index_name = value，指定从哪一行开始，通过NEXT继续浏览。

如果我们不想浏览一个表的所有行，可以使用where和limit子句。

### 实例分析

> 创建测试表及测试数据

	create table handler_table(  
	    c1 int,   
	    c2 varchar(10),   
	    c3 int(10) 
	);  
	insert into handler_table values(2, 'name2', 002);  
	insert into handler_table values(5, 'name5', 005);  
	insert into handler_table values(1, 'name1', 001);  
	insert into handler_table values(4, 'name4', 004);  
	insert into handler_table values(3, 'name3', 003);

> 不通过索引打开查看表

打开句柄：

	mysql> handler handler_table open;

查看表数据：

		mysql> handler handler_table read first;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    2 | name2 |    2 |
		+------+-------+------+
		mysql> handler handler_table read next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    5 | name5 |    5 |
		+------+-------+------+
		mysql> handler handler_table read next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    1 | name1 |    1 |
		+------+-------+------+
		mysql> handler handler_table read next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    4 | name4 |    4 |
		+------+-------+------+
		mysql> handler handler_table read next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    3 | name3 |    3 |
		+------+-------+------+
		mysql> handler handler_table read next;
		Empty set (0.00 sec)

关闭句柄：
	
	mysql> handler handler_table close;
	Query OK, 0 rows affected (0.00 sec)

> 通过索引打开查看表（FIRST,NEXT,PREV,LAST）

通过索引查看的话，可以按照索引的升序，从小到大，查看表信息。

创建索引：

	mysql> create index handler_index on handler_table(c1);

打开句柄：

	mysql> handler handler_table open as p;  

查看表数据：

		mysql> handler p read handler_index first;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    1 | name1 |    1 |
		+------+-------+------+
		mysql> handler p read handler_index next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    2 | name2 |    2 |
		+------+-------+------+
		mysql> handler p read handler_index next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    3 | name3 |    3 |
		+------+-------+------+
		mysql> handler p read handler_index next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    4 | name4 |    4 |
		+------+-------+------+
		mysql> handler p read handler_index next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    5 | name5 |    5 |
		+------+-------+------+
		mysql> handler p read handler_index prev;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    4 | name4 |    4 |
		+------+-------+------+
		mysql> handler p read handler_index last;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    5 | name5 |    5 |
		+------+-------+------+

关闭句柄：

	mysql> handler p close;

> 通过索引打开查看表（=,<=,>=,<,>）

从index为2的地方开始

打开句柄：

	mysql> handler handler_table open as p;  

查看表数据：

		mysql> handler p read handler_index = (2);
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    2 | name2 |    2 |
		+------+-------+------+
		mysql> handler p read handler_index next;     
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    3 | name3 |    3 |
		+------+-------+------+
		mysql> handler p read handler_index next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    4 | name4 |    4 |
		+------+-------+------+
		mysql> handler p read handler_index next;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    5 | name5 |    5 |
		+------+-------+------+
		mysql> handler p read handler_index last;
		+------+-------+------+
		| c1   | c2    | c3   |
		+------+-------+------+
		|    5 | name5 |    5 |
		+------+-------+------+

关闭句柄：

	mysql> handler p close;

> 附加：语法实例参考

	handler handler_table open;
	handler handler_table open as p;
	handler handler_table read first;
	handler handler_table read next;
	handler handler_table read first limit 3;
	handler handler_table read next limit 3,3;
	handler handler_table read first where c1 > 2 limit 2;
	handler handler_table read next where c1 >2 limit 1,2;
	 
	create index handler_index on handler_table(c1);
	handler handler_table open;
	handler handler_table read handler_index first;
	handler handler_table read handler_index next limit 3;
	handler handler_table read handler_index PREV limit 3,3;
	handler handler_table read handler_index LAST where c1 > 2 limit 2;
	handler handler_table read handler_index LAST where c1 > 2 limit 1,2;
	handler handler_table read handler_index = (3);
	handler handler_table read handler_index <= (3) limit 2;
	handler handler_table read handler_index >= (3) limit 1,2;
	handler handler_table read handler_index < (4)  where c1 > 0 limit 2;
	handler handler_table read handler_index > (1)  where c1 < 6 limit 2,2;
	handler handler_table close;
	drop index handler_index on handler_table;

### handler与select的比较

4.1 select语句一次返回所有相关行，handler每次返回一行。

4.2 HANDLER涉及的分析较少，比SELECT更快

4.3 没有优化程序或查询校验开销

4.4 在两个管理程序请求之间，不需要锁定表。

### 注意事项

5.1 如果一个应用停止了，所有仍然打开的句柄将自动停止。

5.2 handler不支持分区表。

5.3 执行TRUNCATE TABLE会关闭所有在该表上打开的handler。

5.4 handler打开一个表时不锁表，也不对表进行快照，所以当表中数据实时更新时，handler将会失去指针的位置。


原文地址：[http://blog.csdn.net/jesseyoung/article/details/40785137](http://blog.csdn.net/jesseyoung/article/details/40785137)

博客主页：[http://blog.csdn.net/jesseyoung](http://blog.csdn.net/jesseyoung)