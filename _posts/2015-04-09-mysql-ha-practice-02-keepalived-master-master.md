---
layout:     post
title:      "MySQL HA 实践二" 
subtitle:   "Keepalived + MySQL 主主复制"   
date:       2015-04-09 21:00:00
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
	<th>Keepalive版本</th>
	<th>主机IP</th>
</tr>
<tr>
	<td>Host1</td>
	<td>CentOS release 6.5 (Final)</td>
	<td>MySQL 5.6.4-m7</td>
	<td>Keepalived 1.2.16</td>
	<td>192.168.100.42</td>
</tr>
<tr>
	<td>Host2</td>
	<td>CentOS release 6.5 (Final)</td>
	<td>MySQL 5.6.4-m7</td>
	<td>Keepalived 1.2.16</td>
	<td>192.168.100.43</td>
</tr>
</table>
虚拟IP： 192.168.100.45

## 基本步骤
<ol>
<li>配置MySQL主主</li>
<li>写MySQL Slave状态检测脚本</li>
<li>配置Keepalived，实现虚拟IP的漂移</li>
</ol>

### 配置MySQL主主
配置MySQL主主，实际上就将两台机器配置配置成对方机器的从节点，即配置两次主从。

### 配置Host1
修改Host1主机的MySQL配置文件（my.cnf），在“[mysqld]”段添加如下内容：

	server-id=1
	log-bin=mysql-bin
	relay-log=mysql-relay-bin
	replicate-wild-ignore-table=mysql.%
	replicate-wild-ignore-table=test.%
	replicate-wild-ignore-table=information_schema.%

进入MySQL命令行，创建用于复制的用户：

	[root@master home]# mysql
	...
	mysql> grant replication slave on *.* to 'repl_user'@'192.168.100.43' identified by 'repl_passwd'
	
### 配置Host2
Host2的配置方法基本与Host1的一样，除了server-id和IP地址等需要变更。

修改Host2主机的MySQL配置文件（my.cnf），在“[mysqld]”段添加如下内容：

	server-id=2
	log-bin=mysql-bin
	relay-log=mysql-relay-bin
	replicate-wild-ignore-table=mysql.%
	replicate-wild-ignore-table=test.%
	replicate-wild-ignore-table=information_schema.%

注意：servier-id 必须是唯一的，不能相同

进入MySQL命令行，创建用于复制的用户：

	[root@master home]# mysql
	...
	mysql> grant replication slave on *.* to 'repl_user'@'192.168.100.42' identified by 'repl_passwd'

	
### 将Host1连接到Host2
在Host1主机进入MySQL命令行，建立连接：
	
	[root@master home]# mysql
	...
	mysql> change master to \
		-> master_host='192.168.100.43', \
		-> master_port=3306, \
		-> master_user='repl_user', \
		-> master_password='repl_passwd';

在Host1上启动slave服务：
	
	mysql> start slave;
	
### 将Host2连接到Host1
在Host2主机进入MySQL命令行，建立连接：
	
	[root@slave home]# mysql
	...
	mysql> change master to \
		-> master_host='192.168.100.42', \
		-> master_port=3306, \
		-> master_user='repl_user', \
		-> master_password='repl_passwd';

在Host2上启动slave服务：
	
	mysql> start slave;

MySQL主主复制配置完成。在任意一台机器的MySQL上写数据，会自动同步到另一台的MySQL。


## MySQL Slave 监控脚本
<b>监控脚本 mysql_ha_check_slave.py <a href="https://github.com/lyremelody/lyremelody.github.com/blob/master/codes/mysql_ha_check_slave.py" target="_blank">点击此处查看</a></b>

## 配置Keepalived
将MySQL Slave监控脚本分别复制到Host1和Host2的keepalived安装目录下 (如：/usr/local/keepalived/etc/mysql_ha_check_slave.py)，并赋执行权限：

	chmod a+x /usr/local/keepalived/etc/mysql_ha_check_slave.py

修改Host1的keepalived的配置文件：（如 /etc/keepalived/keepalived.conf）

	! Configuration File for keepalived

	global_defs {
	    router_id host1
	}

	vrrp_script check_mysqld {
		script "/usr/local/keepalived/etc/mysql_ha_check_slave.py"
		interval 2
		weight 21
	}

	vrrp_instance HA_1 {
		state BACKUP
		interface eth0
		virtual_router_id 80
		priority 100
		advert_int 2
		nopreemtp

		authentication {
       		auth_type PASS
       		auth_pass 1111
    	}

    	track_script {
       		check_mysqld
    	}

    	virtual_ipaddress {
       		192.168.100.45/24 dev eth0
    	}
    }

修改Host2的keepalived的配置文件：（如 /etc/keepalived/keepalived.conf）

	! Configuration File for keepalived

	global_defs {
	    router_id host2
	}

	vrrp_script check_mysqld {
		script "/usr/local/keepalived/etc/mysql_ha_check_slave.py"
		interval 2
		weight 21
	}

	vrrp_instance HA_1 {
		state BACKUP
		interface eth0
		virtual_router_id 80
		priority 90
		advert_int 2
		nopreemtp

		authentication {
       		auth_type PASS
       		auth_pass 1111
    	}

    	track_script {
       		check_mysqld
    	}

    	virtual_ipaddress {
       		192.168.100.45/24 dev eth0
    	}
    }

分别重启Host1、Host2的Keepalived服务：

	service keepalived restart
	
在Host1上可查看虚拟IP状态：
	
	[root@master ~]# ip a
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
   		link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    	inet 127.0.0.1/8 scope host lo
    	inet6 ::1/128 scope host 
       		valid_lft forever preferred_lft forever
	2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    	link/ether 00:50:56:b2:24:37 brd ff:ff:ff:ff:ff:ff
    	inet 192.168.100.42/24 brd 192.168.100.255 scope global eth0
    	inet 192.168.100.45/24 scope global secondary eth0
    	inet6 fe80::250:56ff:feb2:2437/64 scope link 
       		valid_lft forever preferred_lft forever
	3: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN 
    	link/ether 52:54:00:4d:5c:0b brd ff:ff:ff:ff:ff:ff
    	inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
	4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 500
    	link/ether 52:54:00:4d:5c:0b brd ff:ff:ff:ff:ff:ff

MySQL主主复制 + Keepalived 配置完成。

通过访问虚拟IP访问Host1写数据，会自动同步到Host2。当Host1的数据库服务出现故障（服务停掉或者复制线程出现问题）时，虚拟IP会自动漂移到Host2上，后面通过虚拟IP访问会访问到Host2的数据库。

<b>本文所用工具安装包可<a href="https://github.com/lyremelody/package-repositories" target="_blank">点击此处下载</a></b>

