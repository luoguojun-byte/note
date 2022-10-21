```shell
 ###软件检查
//以下命令查询系统中是否已安装了 ppp 软件包  
rpm -qa |grep ppp 
rp-pppoe-3.10-10.el6.i686 ppp-2.4.5-5.el6.i686   //有该软件包显示表明已安装，无显示则未安装 
//以下命令检查系统内核是否支持 MPPE 补丁 
modprobe ppp-compress-18 && echo 'ok!!! ' ok!!!     //显示“ok!!!”则表示支持，否则表示不支持   
//以下命令检查 ppp 是否支持 MPPE 
strings '/usr/sbin/pppd'|grep -i mppe|wc –l 42      //显示≥30 的数字表示支持，显示为 0 表示不支持   
//以下命令检查系统是否开启 ppp 支持 
cat /dev/ppp 
cat: /dev/ppp: No such device or address   //有该行文本显示表明通过   
//以下命令检查系统是否开启 TUN/TAP 支持 
cat /dev/net/tun 
cat: /dev/net/tun: File descriptor in bad state //有该行文本显示表明通过 
##DKMS 软件包必须在 MPPE 内核补丁包之前安装

1．修改 VPN 服务器主配置文件
grep -v "#" /etc/pptpd.conf.bak > /etc/pptpd.conf 
vim /etc/pptpd.conf     //编辑 VPN 服务器主配置文件
#ppp /usr/sbin/pppd      //指定 pppd 程序的路径
option /etc/ppp/options.pptpd   //指定 PPP 选项文件(PPTP 加密和认证选项配置文件)的路径，该文件的内容作为 pptpd   		
debug   
stimeout 10      
noipparam	 
logwtmp   	
bcrelay eth1 	
delegate     						
connections 100
localip 192.168.0.1  //在隧道中为 VPN 服务器设置一个虚拟 IP 地址
remoteip 192.168.0.234-238,192.168.0.245  	//在隧道中指定自动分配给 VPN 客户端的虚拟 IP 地址范围
或者
localip 192.168.0.234-238, 192.168.0.245 
remoteip 192.168.1.234-238,192.168.1.245 

vim /etc/pptpd.conf   
//以下为默认配置文件中有变动的行，其它内容保持不变 
ppp /usr/sbin/pppd 
connections 100 
localip 192.168.1.221 
remoteip 192.168.1.222-253

2．修改 PPP 选项文件 

去掉 vim /etc/ppp/options.pptpd 的配置说明，并且最后所需要的配置如下
name pptpd 
refuse-pap 
refuse-chap 
refuse-mschap 
require-mschap-v2 
require-mppe-128 
proxyarp 
auth            //新增的配置选项 
debug            //该选项去掉行首的#注释符 
lock 
nobsdcomp 
novj 
novjccomp 
nologfd

3．创建 VPN 用户和密码 

vim /etc/ppp/chap-secrets       // 编辑安全认证文件
#client     server    secret      IP addresses
 "csvpn"    pptpd     "123456"    *   //此处为VPN连接测试的用户
 
 4．开启 IP 转发功能
 vim /etc/sysctl.conf 
 		//…其他内容略，仅将以下配置参数的值改为 1，即 
 net.ipv4.ip_forward = 1 
         //修改后保存并退出 
 sysctl -p           //使配置的内核参数生效 
 net.ipv4.ip_forward = 1 
 net.ipv4.conf.default.rp_filter = 1 
 net.ipv4.conf.default.accept_source_route = 0 
 kernel.sysrq = 0 
 kernel.core_uses_pid = 1 
 net.ipv4.tcp_syncookies = 1 
 net.bridge.bridge-nf-call-ip6tables = 0 
 net.bridge.bridge-nf-call-iptables = 0 
 net.bridge.bridge-nf-call-arptables = 0 
 kernel.msgmnb = 65536
 kernel.msgmax = 65536 
 kernel.shmmax = 4294967295 
 kernel.shmall = 268435456 # 
 
 5．设置防火墙转发规则并打开 PPTP 端口 
 service iptables start       //启动 iptables 服务
 iptables -F -t filter 
 iptables -F -t nat 
 iptables -P INPUT ACCEPT 
 iptables -P OUTPUT ACCEPT 
 iptables -P FORWARD ACCEPT 
 iptables -t nat -P PREROUTING ACCEPT 
 iptables -t nat -P POSTROUTING ACCEPT 
 iptables -t nat -P OUTPUT ACCEPT 
 iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT    //注① 
 iptables -A INPUT -p tcp --dport 1723 -j ACCEPT   //注② 
 iptables -A INPUT -p gre -j ACCEPT      //注③ 
 iptables -t nat -A POSTROUTING -o ppp0 -s 192.168.1.0/24 -j MASQUERADE 
 service iptables save        //保存防火墙 iptables 规则 
 
 6．启动 pptpd 服务 
 service pptpd start       //启动 pptpd 服务，或者使用命令 
 netstat -anp |grep pptpd      //查看 pptpd 监听的端口 
 tcp   0 0 0.0.0.0:1723  0.0.0.0:*  LISTEN  5837/pptpd
 unix  2   [ ]   DGRAM    21354 5837/pptpd 
 
 
```

