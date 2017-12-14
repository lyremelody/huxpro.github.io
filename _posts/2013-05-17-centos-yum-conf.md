---
layout:     post
title:      "centos不使用yum更新系统内核"
subtitle:   ""
date:       2013-05-17 12:00:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 运维
---


  在CentOS下使用yum安装和更新软件，但是不想更新系统内核，可参照如下方法：

    1. 编辑/etc/yum.conf
    2. 在"[main]"的最后添加一行"exclude=kernel*"

  那么以后再执行 yum upgrade 就不会自动更新系统内核了。  
  如果之前安装了多个版本的系统内核，但又希望删除其他的系统内核，可参照如下方法：

    执行 yum remove kernel

  它会删除当前运行内核以外的系统内核文件
