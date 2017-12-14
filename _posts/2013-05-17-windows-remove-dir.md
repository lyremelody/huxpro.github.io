---
layout:     post
title:      "Windows下快速删除目录"
subtitle:   ""
date:       2013-05-17 20:00:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 运维
---


  在Windows下，如果一个目录下的子文件或者子目录过多，比如有一万多个，那么直接在界面上操作会需要等很久；它会先计算要删多少文件，然后再删除。
  用命令行就会快很多，打开cmd，输入：

    rd xxx(目录路径) /S /Q

  很快就删除掉了。
