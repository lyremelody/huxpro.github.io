---
layout:     post
title:      "MySQL HA 概览" 
subtitle:   ""  
date:       2015-04-09 19:00:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - MySQL
    - HA
---


高可用性需要解决的两个主要问题：

    1. 如何实现数据共享或数据同步
	2. 如何处理失败转移（failover）


MySQL高可用方案(实践后不断补充)：

    1. <a href="http://blog.lyremelody.org/2015/04/09/mysql-ha-practice-01-replication/" target="_blank">MySQL主从复制</a>
    2. <a href="http://blog.lyremelody.org/2015/04/09/mysql-ha-practice-02-keepalived-master-master/" target="_blank">MySQL主主复制 + Keepalived</a>

