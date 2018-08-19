---
layout:     post
title:      "Windows下用脚本简单实现守护进程"
subtitle:   ""
date:       2018-08-17 18:36:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 2018
    - Windows
---


今天有个同事有个需求： 
1. 需要对Windows下的某个服务进行监控，如果这个服务被停止了，需要自动启动起来； 
2. 这个监控程序要在后台跑。 

针对这个问题，由于是一个非正式的需求，就想想简单的办法满足。 
自然想到了Windows的批处理脚本。 
分两步实现： 
1. 实现服务监控和启动 
2. 隐藏cmd窗口 

## 服务监控和启动
要实现服务监控和启动问题，逻辑上比较简单： 
1. 查找这个服务的进程 
2. 如果不存在，则调用启动服务 
3. 如果存在，则过5秒再检测 

通过tasklist就能实现查找。具体用法在cmd下执行 tasklist /? 
启动服务可以通过sc。具体用法在cmd下执行 sc /? 

基于批处理脚本的检测和启动服务的主要实现可以如下（以监控和启动Teamviewer服务为例），文件名为service_monitor.bat： 
```
@echo off 

set process_name=teamviewer 
set service_name=teamviewer 

:check_service 
tasklist -v | findstr /i %process_name% > NUL 
if ErrorLevel 1 ( 
	goto start_service 
) else ( 
	timeout /t 5 /nobreak > NUL 
	goto check_service 
) 

:start_service 
sc start %service_name% 
goto check_service 
```

### 问题
可能遇到的问题，没有权限，那么需要用管理员权限运行批处理脚本，比如右键，以管理员权限运行。 

## 隐藏执行窗口
剩下的就是隐藏执行窗口了。 

可以使用vbs实现。具体如下，文件名为service_monitor.vbs： 
```
CreateObject(“WScript.Shell”).Run “cmd /c d:/service_monitor/service_monitor.bat”,0
```

把这两个文件放到D盘下的service_monitor目录下，执行service_monitor.vbs文件即可。 

### 问题
可能遇到的问题，还是没有权限。 
右键service_monitor.vbs，没有以管理员权限运行的选项，怎么办？ 
以管理员权限打开cmd，在cmd下切换到vbs脚本所在路径，执行即可。 

问题解决。 

