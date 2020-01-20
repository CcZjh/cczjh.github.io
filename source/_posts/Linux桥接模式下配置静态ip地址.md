---
title: Linux桥接模式下配置静态ip地址
date: 2020-01-15 09:59:44
tags: ["Linux"]
categories: ["后端","运维"]
---
### 桥接模式下配置静态ip
#### 一、涉及三个文件的配置： 

1. /etc/sysconfig/network 
2. /etc/sysconfig/network-scripts/ifcfg-eth0 
3. /etc/resolv.conf

#### 二、具体配置过程 

1. ipconfig查看本机ip地址 

![](1.png)

----------

    即本机ip地址为：192.168.1.100
	网关为：192.168.1.1 
	子网掩码为：255.255.255.0

2. 根据本机ip地址配置linux的静态ip，如下：


----------

	// 进入配置
    vim /etc/sysconfig/network-scripts/ifcfg-eth0
  
	DEVICE=eth0        #虚拟机网卡名称。
	TYPE=Ethernet
	ONBOOT=yes　　      #开机启用网络配置。
	NM_CONTROLLED=yes
	BOOTPROTO=static      #static，静态ip，而不是dhcp，自动获取ip地址。
	IPADDR=192.168.1.101　　#设置我想用的静态ip地址，要和物理主机在同一网段，但又不能相同。
	NETMASK=255.255.255.0  #子网掩码，和物理主机一样就可以了。
	GETWAY=192.168.1.1     #和本机一样
	DNS1=8.8.8.8　　　　　　#DNS，写谷歌的地址(免费)。
	HWADDR=00:oc:29:A5:0D:38
	IPV6INIT=no
	USERCTL=no

3. 在网络配置文件 /etc/sysconfig/network 中添加网关地址

----------

	NETWORKING=yes
	HOSTNAME=localhost.localdemain
	GATEWAY=192.168.1.1   #网关地址，同物理主机的网关地址

4. 配置 /etc/resolv.conf

----------

	nameserver 8.8.8.8

5. 重启网卡 

----------

	service network restart 

#### 三、测试

1. 测试与本机互ping

虚拟机ping主机，

----------

	ping 192.168.100
 
![](2.png)

主机ping虚拟机 

![](3.png)

虚拟机ping外网 

![](4.png)

#### 四、遇到的问题

1. 按照上述步骤可实现ping通外网和主机ping同虚拟机，但存在虚拟机ping不通主机，主要问题是物理本机的防火墙没有关闭，关闭即可。
2. 自己配置遇到问题的参考博客： vmware下centOs设置静态IP和无法本地ping通centOs的ip解决方法([http://blog.csdn.net/shaonaozu/article/details/12869185](http://blog.csdn.net/shaonaozu/article/details/12869185))
