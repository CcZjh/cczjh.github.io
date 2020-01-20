---
title: Linux iptables防火墙开放端口不生效
date: 2020-01-10 14:43:59
tags: ["Linux"]
categories: ["运维"]
---

### iptables常用命令

----------

    service iptables start // 开启iptables
	service iptables stop // 关闭iptables
	service iptables restart // 重启
	service iptables status // iptables状态
	service iptables save // 保存iptables配置文件

	/etc/sysconfig/iptables // iptables规则存放文件
	

### iptables开放端口
#### 一.

----------

	// 先使用此命令将需要开放的端口添加进规则内
    iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
	iptables -A OUTPUT -p tcp --sport 8080 -j ACCEPT
	
	// 保存规则
	service iptables save

	// 重启
	service iptables restart

#### 二.

----------

	// 编辑iptables规则文件
	vim /etc/sysconfig/iptables
	
	// 添加如下
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT

	// 重启
	service iptables restart

### 注:
有时候当进行如上操作时会发现接口任不可访问, 是因为用以上的方式添加接口规则时, 接口默认排在最后, 此时接口规则并不生效, 所以我们应该用iptables -I进行添加, 这样就会将8080接口规则放在首位

	iptables -I INPUT -p tcp --dport 8080 -j ACCEPT

![](1.png)
