``` shell
rpm -qa |grep sendmail 
rpm -qa |grep m4
rpm -qa |grep dovecot

vim /var/named/xinyuan.com.zone      //编辑域名正向解析资源文件
$TTL  86400 @   1D IN SOA  dns.xinyuan.com. admin.xinyuan.com. (   
						42   ; serial (d. adams) 
   						3H   ; refresh     
						15M  ; retry     
						1W   ; expiry     
						1D ) ; minimum         
						IN NS  dns.xinyuan.com.     
						IN MX 5 mail.xinyuan.com. //添加邮件交换记录 
mail   					 IN A  192.168.0.210 //添加 E-mail 服务器主机记录
dns    					 IN A  192.168.0.200 
www    					 IN A  192.168.0.200 
ftp    					 IN A  192.168.0.200 //修改后保存并退出 

##创建用户并且输入密码

##为用户创建邮箱目录
mkdir -p /home/chf/mail/.imap/INBOX   //使用-p 选项可逐级自动创建目录，即上一级目录不存在就自动创建它，要注意大小写  
//也可以使用下面的方法来创建 chf 用户的邮箱目录：
su - chf         //临时切换到 chf 用户身份 
$ mkdir -p mail/.imap/INBOX     //此时当前目录在 chf 用户主目录下 
$ exit          //退出 chf 用户身份返回 root 身份 
##每次创建 一个新的用户时能自动为其创建邮箱目录
 vim /etc/skel/.bash_profile
 …  //文件内容略，在文件中添加以下代码 
if [ ! -d ~/mail/.imap/INBOX ];then  
mkdir -p ~/mail/.imap/INBOX 
fi 


##创建邮箱用户别名
vim /etc/aliases       //编辑/etc/aliases 文件 
…  //已有内容略，在文件最后输入 chf 和 wjm 两个用户名与别名 
Person who shoukl get root’s mail 
root:  marc 
chf:  chf 
wjm:  wjm            //输入完毕，保存并退出   
//也可以使用以下命令直接把 chf 和 wjm 两个用户名及别名追加到 aliases 文件末尾 
echo ”chf: chf” >>/etc/aliases  
echo ”wjm: wjm” >>/etc/aliases 
##生成别名数据库文件。Sendmail 并不直接读取/etc/aliases 文本文件，而是使 用该文件的 DBM 数据库格式文件/etc/aliases.db。
因此，还需要通过 newaliases 命令，根据 文本文件/etc/aliases 的内容来自动生成/etc/aliases.db 数据库格式文件，命令如下
newaliases
##修改 Sendmail 主配置文件/etc/mail/sendmail.cf，删去“DaemonPortOptions” 配置选项的行首注释符#而使其生效，
并将“Addr=127.0.0.1”中的 IP 地址改为 0.0.0.0，以 允许中继来自 Internet 或其他任何网络传入的邮件。 
vim /etc/mail/sendmail.cf 
…  //找到以下一行内容(CentOS 6.5 原始的默认文件中为第 261 行) 
# O DaemonPortOptions=Port=smtp,Addr=127.0.0.1, Name=MTA   
//删去行首的注释符#，并修改其中的 IP 地址为 0.0.0.0，即： 
O DaemonPortOptions=Port=smtp,Addr=0.0.0.0, Name=MTA     //修改后保存退出 
## 如果 E-mail 服务器仅供企业内部员工之间发送邮件，无须中继其他网络传入 的邮件，
则“DaemonPortOptions”配置选项后“Addr=127.0.0.1”中的 IP 地 址也可改为 E-mail 服务器自身的 IP 地址 192.168.1.3。
由于 Sendmail 主配置 文件 sendmail.cf 的语法深奥难懂，多数管理员不去直接修改该文件，
而是通 过修改另一个相对容易理解的宏配置文件/etc/mail/sendmail.mc，
然后使用 m4 宏处理程序依据 sendmail.mc 文件来生成 sendmail.cf 文件。以下给出这种方 法的操作步骤，有关宏配置文件 sendmail.mc 详解可参阅附录 B。
vim /etc/mail/sendmail.mc 
…  //找到以下一行内容(CentOS 6.5 原始的默认文件中为第 116 行)
dnl  # DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl   
	//删去行首的注释符“dnl #”，并修改语句中的 IP 地址为 0.0.0.0，即改为如下 
DAEMON_OPTIONS(`Port=smtp,Addr=0.0.0.0, Name=MTA')dnl
m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf   
//根据宏配置文件 sendmail.mc 生成 Sendmail 主配置文件 sendmail.cf
##access 文件中定义允许访问的该 E-mail 服务器的域、IP 地址和访问类型，然后执行 makemap 命 令将其转换生成 access.db 数据库文件，具体操作如下。 
vim /etc/mail/access   //在以下文件内容的最后添加两行内容，设置中继域和网络 
Connect:xinyuan.com       RELAY 
Connect:192.168.1.3       RELAY
//添加后保存并退出，执行以下命令 
makemap –r hash /etc/mail/access.db < /etc/mail/access    //生成 access.db 数据库文件 
##修改 Sendmail 接收邮件的主机列表文件/etc/mail/local-host-names,其内容为指定收发邮件的主机域名信息，
也就是说 Sendmail 将所有允许中继的 本地域或主机名称放在该文件中
vim /etc/mail/local-host-names 	 
local-host-names  -  include all aliases for your machine here.   
//该文件默认仅有以上一行注释，在此后添加 E-mail 服务器本地域名称 
xinyuan.com
##通过修改 sendmail.mc 文件来指 定 E-mail 服务器域名的方法
vim /etc/mail/sendmail.mc 
…  
//找到以下一行内容(CentOS 6.5 原始的默认文件中为第 155 行) 

LOCAL_DOMAIN(`localhost.localdomain')dnl  
	 //如有行首注释符“dnl”将其删去，并修改为如下 
LOCAL_DOMAIN(`xinyuan.com')dnl           //修改后保存并退出 
m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf   
	//执行该命令来生成 Sendmail 主配置文件/etc/mail/sendmail.cf 
##配置 POP3 服务
vim /etc/dovecot/dovecot.conf 
…  //找到以下一行内容(CentOS 6.5 原始的默认文件中为第 20 行) 
protocols = imap pop3 lmtp   
	//删去行首的注释符#，即改为以下配置行，或者在原注释行后直接添加以下配置行 
protocols = imap pop3 lmtp    
	 //也可删去其他协议仅保留 POP3 …  
	//其他配置内容略，文件最后有一个 include 语句，注意以感叹号(!)开头 
!include conf.d/*.conf      //包含 conf.d 目录下的所有配置文件
  	//修改后保存并退出 
##修改/etc/dovecot/conf.d/10-auth.conf 文件，该文件负责设置 dovecot 所使用的 sasl 验证方法
vim /etc/dovecot/conf.d/10-auth.conf 
…  
	//找到以下一行内容 
disable_plaintext_auth = yes   
	//删去行首的注释符#，并将选项值 yes 改为 no，也可在此注释行后直接添加以下行 
disable_plaintext_auth = no 
##修改/etc/dovecot/conf.d/10-ssl.conf 文件，该文件主要负责 dovecot 的 SSL 认证 相关的配置
vim /etc/dovecot/conf.d/10-ssl.conf 
…  
	//找到以下一行内容 
ssl = yes   
	//删去行首的注释符#，并将其值 yes 改为 no，也可在此注释行后直接添加以下行 
ssl = no 
##修改/etc/dovecot/conf.d/10-mail.conf 文件，该文件主要定义邮件用户存储相关 信息的位置
vim /etc/dovecot/conf.d/10-mail.conf 
…  
	//找到以下一行被注释的用于指定用户邮箱目录位置的配置选项    
#mail_location = mbox:~/mail:INBOX=/var/mail/%u   
	//只需删去行首的注释符#，即改为： 
mail_location = mbox:~/mail:INBOX=/var/mail/%u
##启动 sendmail 和 dovecot 服务
netstat -antulp | grep :110    //查看服务是否启动成功 
tcp  0  0 0.0.0.0:110    0.0.0.0:*  LISTEN 9778/dovecot 
tcp  0  0 :::110         :::*       LISTEN 9778/dovecot   
//有上述显示表明 dovecot 服务已成功启动  
telnet localhost 110      //测试连接本机 110 端口 
```

