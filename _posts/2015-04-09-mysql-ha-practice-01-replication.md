---
layout:     post
title:      "MySQL HA 实践一" 
subtitle:   "MySQL 主从复制"  
date:       2015-04-09 20:00:00
author:     "lyremelody"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - MySQL
    - HA
---

## 环境配置
<table border="1" cellpadding="10">
<tr>
	<th>主机</th>
	<th>操作系统版本</th>
	<th>MySQL版本</th>
	<th>主机IP</th>
</tr>
<tr>
	<td>master</td>
	<td>CentOS release 6.5 (Final)</td>
	<td>MySQL 5.6.4-m7</td>
	<td>192.168.100.42</td>
</tr>
<tr>
	<td>slave</td>
	<td>CentOS release 6.5 (Final)</td>
	<td>MySQL 5.6.4-m7</td>
	<td>192.168.100.43</td>
</tr>
</table>

## 主从复制配置的基本步骤
<ol>
<li>配置一个服务器作为Master</li>
<li>配置一个服务器作为Slave</li>
<li>将Slave连接到Master</li>
</ol>

### 配置Master
修改Master主机的MySQL配置文件（my.cnf），在“[mysqld]”段添加如下内容：

	server-id=1
	log-bin=mysql-bin

进入MySQL命令行，创建用于Slave复制的用户：

	[root@master home]# mysql
	...
	mysql> grant replication slave on *.* to 'repl_user'@'192.168.100.43' identified by 'repl_passwd'

### 配置Slave
修改Slave主机的MySQL配置文件（my.cnf），在“[mysqld]”段添加如下内容：

	server-id=2
	log-bin=mysql-bin
	relay-log=mysql-relay-bin
	replicate-wild-ignore-table=mysql.%
	replicate-wild-ignore-table=test.%
	replicate-wild-ignore-table=information_schema.%

注意：servier-id 必须是唯一的，不能相同

### 将Slave连接到Master
在Slave主机进入MySQL命令行，建立连接：
	
	[root@slave home]# mysql
	...
	mysql> change master to \
		-> master_host='192.168.100.42', \
		-> master_port=3306, \
		-> master_user='repl_user', \
		-> master_password='repl_passwd';

在Slave上启动slave服务：
	
	mysql> start slave;

MySQL主从复制配置完成。在Master上写数据，会自动同步到Slave。

<b>本文所用工具安装包可<a href="https://github.com/lyremelody/package-repositories" target="_blank">点击此处下载</a></b>

