---
layout:     post
title:      "Centos增加管理员账户" 
subtitle:   ""  
date:       2014-01-21 20:00:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 运维
---

1.添加普通用户

	useradd lyre
	
2.修改/etc/sudoers，找到如下行：

	## Allow root to run any commands anywhere 
	root    ALL=(ALL)       ALL

在后面添加一行：

	lyre      ALL=(ALL)       ALL
	
修改完成，通过lyre账号登录，用sudo即可获取root权限做一些操作
