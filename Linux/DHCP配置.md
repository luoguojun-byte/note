## 配置实例

``` powershell
 [root@T1 ~]# cp /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample /etc/dhcp/dhcpd.conf 拷贝模板
 [root@T1 ~]# cd /etc/dhcp/dhcpd.conf  进入dhcp编辑配置文件
 ## 具体配置如下
 option domain-name "example.org";
 option domain-name-servers ns1.example.org, ns2.example.org;
 ddns-update-style interim;
 ignore client-updates;
 subnet 192.168.0.0 netmask 255.255.255.0 {       ## 这是我自己做的时候虚拟机设置的IP
 option routers 192.168.0.1;				   ## 路由的IP地址
 option subnet-mask 255.255.255.0;		        ##	
 option domain-name-servers 192.168.0.200;		## DHCP服务器的地址
 option time-offset -18000;					   ## 关闭时间
 range dynamic-bootp 192.168.0.190 192.168.0.199; ## 分配dhcp地址池
 default-lease-time 600;						   ## 最小时间
 max-lease-time 7200;						   ## 最大时间
 }
 "/etc/sysconfig/dhcpd" 2L, 43C written
 [root@T1 ~]# vi /etc/dhcp/dhcpd.conf             #设置所要应用的接口
 DHCPDARGS=eth0                                  #仅在eth0上提供dhcp服务，多网卡写成 DHCPDARGS=eth0 eth1
 [root@T1 ~]# service dhcpd restart			    #重启dhcp服务
 Shutting down dhcpd: [  OK  ]
 Starting dhcpd: [  OK  ]
 [root@T1 ~]# /var/lib/dhcpd/dhcpd.leases        #记录着DHCP服务器向DHCP客户机提供租用的每个IP地址的信息
```

