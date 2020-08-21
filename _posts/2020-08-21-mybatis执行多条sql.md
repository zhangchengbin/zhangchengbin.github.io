---
layout: post
title:  mybatis执行多条sql
date:   2020-08-21 11:40:00 +0800
categories: myabtis
tag: 
---

* content
{:toc}


1. 在mybatis中写好语句, 多条sql之间用分号分隔, 最后一个也要加上分号  

![](http://source.zhangcb.site/img/20200821114313.png)  

2. 需要在连接字符串中增加, allowMultiQueries=true  

![](http://source.zhangcb.site/img/20200821114445.png)  
