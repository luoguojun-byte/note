一、安装环境准备

#### 1.修改网卡ip

```
X视情况而定
网段10.0.0.X/24 172.129.X.0/24
网关10.0.0.254 172.129.X.254
controller 10.0.0.35
		   172.129.35.1
		   
compute	   10.0.0.36
		   172.129.35.2
```

#### 2.修改主机名，并添加hosts映射 /etc/hosts

```
[root@控制 ~]# hostnamectl set-hostname controller
[root@计算 ~]# hostnamectl set-hostname compute
[root@计算&控制 ~]# vi /etc/hosts
#在最后添加
10.0.0.35 controller
10.0.0.36 compute
```

#### 3.关闭防火墙Selinux

```
[root@计算&控制 ~]# iptables -F
[root@计算&控制 ~]# iptables -X
[root@计算&控制 ~]# iptables -Z
[root@计算&控制 ~]# setenforce 0
[root@计算&控制 ~]# vi /etc/selinux/config
#将SELINUX=enforcing更改为SELINUX=Permissive
```

#### 4.上传镜像

通过CRT上传镜像至/root目录下

#### 5.配置本地yum源

```
[root@计算&控制 ~]# rm -rf /etc/yum.repos.d/*
[root@控制 ~]# mkdir /opt/openstack
[root@控制 ~]# mkdir /opt/centos
[root@控制 ~]# mount CentOS-7-x86_64-DVD-1804.iso /opt/centos/
[root@控制 ~]# mount openstack-queens-7.5.iso /opt/openstack/
[root@控制 ~]# vi /etc/yum.repo.d/ftp.repo
#添加如下
[openstack]
name=openstack
baseurl=file:///opt/openstack
gpgcheck=0
enabled=1

[centos]
name=centos
baseurl=file:///opt/centos
gpgcheck=0
enabled=1
[root@计算 ~]# vi /etc/yum.repo.d/ftp.repo
#添加如下
[openstack]
name=openstack
baseurl=ftp://source/openstack
gpgcheck=0
enabled=1

[centos]
name=centos
baseurl=ftp://source/centos
gpgcheck=0
enabled=1
```





# 二、Openstack基础环境安装

#### 1.在控制节点和计算节点安装ntp服务

```
[root@计算&控制 ~]# yum -y install chrony
[root@控制 ~]# vi /etc/chrony.conf
```

编辑配置文件/etc/chrony.conf增加以下内容

```
server 172.129.35.0/24
local stratum 10
```

```
[root@控制 ~]# systemctl restart chronyd

[root@控制 ~]# systemctl enable chron=yd
[root@计算 ~]# vi /etc/chrony.conf
#server 0.centos.pool.ntp.org iburst  //注释掉这一行
#server 1.centos.pool.ntp.org iburst  //注释掉这一行
#server 2.centos.pool.ntp.org iburst  //注释掉这一行
#server 3.centos.pool.ntp.org iburst  //注释掉这一行
server 172.129.35.1 iburst  //添加

[root@计算 ~]# systemctl restart chronyd
[root@计算 ~]# systemctl enable chronyd
```

#### 2.创建磁盘分区

```
[root@计算 ~]# fdisk -l  //查看磁盘
[root@计算 ~]# fdisk /dev/md126    //对md126进行分区，按实际情况
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x89a2c7f8.

Command (m for help): n             //输入n表示new一个分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p               //输入p，p表示主分区，e表示扩展分区
Partition number (1-4, default 1): 1     //输入主分区号
First sector (2048-207615999, default 2048): Enter  //起始分区，回车即可
Last sector, +sectors or +size{K,M,G} (20-2076, default 207615999): +100G   //输入+100G表示分区大小为100
Partition 1 of type Linux and of size 10 GiB is set

Command (m for help): p    //打印输入当前分区

Disk /dev/sda2: 106.3 GB, 106299392000 bytes, 207616000 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x89a2c7f8

     Device Boot      Start         End      Blocks   Id  System
/dev/md126p1        20971520    41943039    10485760   83  Linux

Command (m for help): w  //对结果进行保存
```



#### 3.Openstack client安装

##### 离线安装

在controller和compute节点上分别安装Openstack client。

```
[root@计算&控制 ~]# yum clean all
[root@计算&控制 ~]# yum upgrade
[root@计算&控制 ~]# yum install python-openstackclient -y
[root@计算&控制 ~]# yum install openstack-selinux -y
```

##### 在线安装

```
cd /etc/pki/rpm-gpg
yum install -y wget
wget https://archive.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-7
yum install -y https://rdoproject.org/repos/rdo-release.rpm
yum install -y centos-release-openstack-queens
yum update -y
yum install -y python-openstackclient
yum install openstack-selinux -y
```



#### 4.安装mariadb服务

在controller节点上安装mariadb服务并完成配置

```
[root@控制 ~]# yum install mariadb mariadb-server python2-PyMySQL -y
```

创建并编辑文件/etc/my.cnf.d/openstack.cnf增加如下内容

```
[mysqld]
bind-address=172.129.35.1
default-storage-engine=innodb
innodb_file_per_table=on
max_connections=4096
collation-server=utf8_general_ci
character-set-server=utf8
```

设置数据库服务开机启动，并启动

```
[root@控制 ~]# systemctl restart mariadb
[root@控制 ~]# systemctl enable mariadb
```

初始化数据库

```
[root@controller ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): Enter
OK, successfully used password, moving on...

Set root password? [Y/n] y
New password: 000000            //默认输入不会显示
Re-enter new password: 000000 
Password updated successfully!
 ... Success!

Remove anonymous users? [Y/n] y
 ... Success!

Disallow root login remotely? [Y/n] n
 
Remove test database and access to it? [Y/n] y

Reload privilege tables now? [Y/n] y
 ... Success!

Thanks for using MariaDB!
```



#### 5.安装rabbitmq消息队列

```
[root@控制 ~]# yum -y install rabbitmq-server
[root@控制 ~]# systemctl restart rabbitmq-server
[root@控制 ~]# systemctl enable rabbitmq-server
```

创建openstack用户、授权和角色设置

```
[root@controller ~]# rabbitmqctl add_user openstack 000000 //用户创建
Creating user "openstack" ... 
[root@controller ~]# rabbitmqctl set_user_tags openstack administrator  //角色赋予
Setting tags for user "openstack" to [administrator] ...   

对何种资源具有配置、写、读的权限通过正则表达式来匹配，具体命令如下： 
set_permissions [-p <vhostpath>] <user> <conf> <write> <read> 
其中，<conf> <write> <read>的位置分别用正则表达式来匹配特定的资源，如'^(amq\.gen.*|amq\.default)$'可以匹配server生成的和默认的exchange，'^$'不匹配任何资源 

[root@controller ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"  //权限赋予
Setting permissions for user "openstack" in vhost "/" ... 
[root@controller ~]# rabbitmqctl list_user_permissions user_admin   //查看权限
[root@controller ~]# rabbitmqctl list_users    //查看确认
```



#### 6.安装memcached服务

在控制节点安装memcached服务

```
[root@controller ~]# yum install memcached python-memcached -y
```

编辑/etc/sysconfig/memcached文件设置一下内容

```  
[root@controller ~]# vi /etc/sysconfig/memcached 
OPTIONS="-l 127.0.0.1,::1" //原内容
OPTIONS="-l 127.0.0.1,::1,controller"  //添加
[root@controller ~]# systemctl restart memcached
[root@controller ~]# systemctl enable memcached
```

#### 7.安装etcd服务

```
[root@controller ~]# yum -y install etcd
```

编辑文件/etc/etcd/etcd.conf设置以下内容

```
[root@controller ~]# vi /etc/etcd/etcd.conf 

#ETCD_LISTEN_PEER_URLS="http://localhost:2380"  //取消注释
#ETCD_LISTEN_CLIENT_URLS="http://localhost:2379" //取消注释
ETCD_NAME="controller"
#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380" //取消注释
#ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379" //取消注释
#ETCD_INITIAL_CLUSTER="default=http://localhost:2380" //取消注释
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster" //取消注释
#ETCD_INITIAL_CLUSTER_STATE="new" //取消注释

[root@controller ~]# systemctl restart etcd
[root@controller ~]# systemctl enable etcd
```



#### 8.安装Keystone服务

在mariadb上为keystone创建管理数据库

```
[root@controller ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9

MariaDB [(none)]> create database keystone;

MariaDB [(none)]> grant all privileges on keystone.* to 'keysont'@'localhost' identified by '000000';

MariaDB [(none)]> grant all privileges on keystone.* to 'keysont'@'%' identified by '000000';

[root@controller ~]# yum install openstack-keystone httpd mod_wsgi -y
```

编辑文件/etc/keystone/keystone.conf修改如下配置

```
[root@controller ~]# ADMIN_TOKEN=$(openssl rand -hex 10)
[root@controller ~]# echo $ADMIN_TOKEN
8b6c120fab18b64f493
[root@controller ~]# cp /etc/keystone/keystone.conf /etc/keystone/keystone.conf.bak 
//过滤掉空格和注释，然后导入配置文件
[root@controller ~]# grep -Ev '^$|#' /etc/keystone/keystone.conf.bak >/etc/keystone/keystone.conf
[root@controller ~]# vi /etc/keystone/keystone.conf

[DEFAULT]
...
admin_token = 8b6c120fab18b64f493 //echo $ADMIN_TOKEN生成的值

[database]
...
connection = mysql+pymysql://keystone:这里用密码替换@controller/keystone

[token]
...
provider = fernet
driver = memcache

[cache]
memcache_servers = 172.129.35.1:11211
```

在keystone上同步认证服务的数据库：

```
[root@controller ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone
[root@controller ~]# ll /var/log/keystone/
//正常情况显示
- rw-rw---- 1 root keystone 24812 May 10 15:25 keystone.log 
检查数据库连接：
[root@controller ~]# mysql -h 172.129.35.1 -ukeystone -p000000 -e "use keystone;show tables;"   //正常无返回
```

初始化keystone Fernet keys密匙：

```
[root@controller ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
[root@controller ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

[root@controller keystone]# ll /etc/keystone/ #初始化后创建fernet-keys目录，里面有KEY文件 
drwx------ 2 keystone keystone     24 May 14 16:25 credential-keys 
-rw-r----- 1 root     keystone   2303 Feb 28 19:28 default_catalog.templates 
drwx------ 2 keystone keystone     24 May 14 16:25 fernet-keys 
-rw-r----- 1 root     keystone 122660 May 14 16:24 keystone.conf 
-rw-r----- 1 root     keystone 122684 May 14 14:23 keystone.conf.rpmsave 
-rw-r----- 1 root     keystone   2493 Feb 28 19:28 keystone-paste.ini 
-rw-r----- 1 root     keystone   1046 Feb 28 19:28 logging.conf 
-rw-r----- 1 root     keystone      3 Feb 28 21:02 policy.json 
-rw-r----- 1 keystone keystone    665 Feb 28 19:28 sso_callback_template.html 
```

配置keystone引导身份服务

```
keystone-manage bootstrap \
--bootstrap-password 000000 \
--bootstrap-admin-url http://10.0.0.60:35357/v3/ \
--bootstrap-internal-url http://10.0.0.60:5000/v3/ \
--bootstrap-public-url http://10.0.0.60:5000/v3/ \
--bootstrap-region-id RegionOne

```

在keystone上配置 Apache HTTP 服务器：

编辑/etc/httpd/conf/httpd.conf 文件，配置`ServerName` 选项为控制节点

```
[root@controller ~]# vi /etc/httpd/conf/httpd.conf
ServerName controller
```

在apache目录下创建keystone配置文件，将keystone在apache中的配置文件软链接到apache目录下

```
[root@controller ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

重启httpd服务

```
[root@controller ~]# systemctl enable httpd
[root@controller ~]# systemctl restart httpd
```

查看5000和35357端口服务是否正常启动

```
[root@controller ~]# yum -y install net-tools
[root@controller1 conf.d]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      928/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1130/master         
tcp6       0      0 :::5000                 :::*                    LISTEN      1886/httpd          
tcp6       0      0 :::80                   :::*                    LISTEN      1886/httpd          
tcp6       0      0 :::22                   :::*                    LISTEN      928/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1130/master         

tcp6       0      0 :::35357                :::*                    LISTEN      1886/httpd    
```

注意！！！若出现35357端口没启动则按以下步骤,缺什么加什么

```
[root@controller ~]# vi /usr/share/keystone/wsgi-keystone.conf 
Listen 5000
Listen 35357
<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    LimitRequestBody 114688
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/httpd/keystone.log
    CustomLog /var/log/httpd/keystone_access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    LimitRequestBody 114688
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/httpd/keystone.log
    CustomLog /var/log/httpd/keystone_access.log combined
<Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

Alias /identity /usr/bin/keystone-wsgi-public
<Location /identity>
    SetHandler wsgi-script
    Options +ExecCGI

    WSGIProcessGroup keystone-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
</Location>

Alias /identity_admin /usr/bin/keystone-wsgi-admin
<Location /identity_admin>
    SetHandler wsgi-script
    Options +ExecCGI

    WSGIProcessGroup keystone-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
</Location>
```

curl验证是否正确

```
[root@controller ~]# curl 172.129.35.1:5000
{"versions": {"values": [{"status": "stable", "updated": "2019-07-19T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.13", "links": [{"href": "http://10.0.0.100:5000/v3/", "rel": "self"}]}]}}[root@controller keystone]# 

[root@controller ~]# curl 172.129.35.1:35357
{"versions": {"values": [{"status": "stable", "updated": "2019-07-19T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.13", "links": [{"href": "http://10.0.0.100:5000/v3/", "rel": "self"}]}]}}[root@controller keystone]# 
```

连接到keystone

配置keystone连接的环境变量

```
[root@controller ~]# export OS_USERNAME=admin 
export OS_PASSWORD=000000
export OS_PROJECT_NAME=admin 
export OS_USER_DOMAIN_NAME=Default 
export OS_PROJECT_DOMAIN_NAME=Default 
export OS_AUTH_URL=http://10.0.0.60:35357/v3 
export OS_IDENTITY_API_VERSION=3

```

下面的操作都将按这些环境变量中的参数进行设定，记住admin用户和密码

创建名为service的服务

```
[root@controller]# openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 078bc842002c4ba2a005e0bb4ab4114b |
| is_domain   | False                            |
| name        | service                          |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

创建平台demo项目

```
[root@controller]# openstack project create --domain default --description "Demo Project" demo
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 580791afda7142e3a7edea67c639a2b1 |
| is_domain   | False                            |
| name        | demo                             |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

创建demo用户

```
[root@controller]# openstack user create --domain default  --password-prompt demo
User Password: 000000   //默认输入不显示
Repeat User Password: 000000
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | f2465f8034ac4f9aa6d411e3413fe05e |
| name                | demo                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

创建用户角色

```
[root@controller]# openstack role create user
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| domain_id   | None                             |
| id          | df3ed49e53dc48c4acb233c5080c6703 |
| name        | user                             |
| options     | {}                               |
+-------------+----------------------------------+

```

添加用户角色，给demo用户增加user权限

```
[root@controller# openstack role add --project demo --user demo user
//说明:此条命令执行成功后不返回参数
```

**验证操作**

先取消环境变量

```
[root@controller# unset OS_AUTH_URL OS_PASSWORD
```

使用export|grep OS_ 查看环境变量是否取消

```
[root@controller]# export|grep OS_
declare -x OS_IDENTITY_API_VERSION="3"
declare -x OS_PROJECT_DOMAIN_NAME="Default"
declare -x OS_PROJECT_NAME="admin"
declare -x OS_USERNAME="admin"
declare -x OS_USER_DOMAIN_NAME="Default"
```

admin用户返回的认证token

```
[root@controller keystone]# openstack --os-auth-url http://172.129.35.1:35357/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
Password: 000000   //默认不显示
Password: 000000
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2021-04-15T06:21:04+0000                                                                                                                                                                |
| id         | gAAAAABgd81A0yPfSXXvt0hMcyFWxoPJKeY7ArYXhpTAkvS11UXcGa3peUO-4N4lSUkXZVq3MuMbI2xXeH5MUceAL1bQHoZG9VU3YAKjzrE2HaXGqO-7S0XEdSd6MzNW9mnPoxPZuNo7bXwGjKZrBUzWzr-UdVFnkDuYLmih97emgPqnOTrRec0 |
| project_id | 8381345527d94cfc9991f8fd5a8d9e5b                                                                                                                                                        |
| user_id    | d3f93386317e437bbea9240ce6133df5                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

demo用户返回的认证token

```
openstack --os-auth-url http://172.16.70.201:5000/v3 \
> --os-project-domain-name Default --os-user-domain-name Default \
> --os-project-name demo --os-username demo token issue
```

