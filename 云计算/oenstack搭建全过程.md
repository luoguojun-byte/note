# 环境部署

## 两个节点改名

```
[root@localhost ~]hostnamectl set-hostname controller
[root@localhost ~]hostnamectl set-hostname compute
改完重启就看得到名字变化
```



## 配置controller、compute主机映射

```
[root@controller ~]vi /etc/hosts
[root@controller ~]10.0.0.60 controller
[root@controller ~]10.0.0.61 compute
```

## 关闭防火墙和安全策略

```
##compute节点一样，这里就不打了
[root@controller ~]vi /etc/selinux/config
[root@controller ~]SELINUX=disabled

关闭防火墙
[root@controller ~]systemctl stop firewalld.service
[root@controller ~]systemctl disable firewalld.service

清空iptables规则
[root@controller ~]iptables -F
[root@controller ~]iptables -X
[root@controller ~]iptables -Z
[root@controller ~]iptables-save
```

## 创建文件夹并挂载资源

```
[root@controller ~]# mkdir /opt/openstack
[root@controller ~]# mkdir /opt/centos
[root@controller ~]# mount CentOS-7-x86_64-DVD-1804.iso /opt/centos/
[root@controller ~]# mount openstack-queens-7.5.iso /opt/openstack/
```

## 配置本地源

```
##controller节点
[root@controller ~]# rm -rf /etc/yum.repos.d/*
[root@controller ~]# vi /etc/yum.repo.d/ftp.repo
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

##compute节点
[root@compute ~]# rm -rf /etc/yum.repos.d/*
[root@compute ~]# vi /etc/yum.repo.d/ftp.repo
[openstack]
name=openstack
baseurl=ftp://controller/openstack
gpgcheck=0
enabled=1

[centos]
name=centos
baseurl=ftp://controller/centos
gpgcheck=0
enabled=1
```

## 在controller安装vsftpd

```
[root@controller ~]# yum -y install vsftpd
[root@controller ~]# vi /etc/vsftpd/vsftpd.conf
anon_root=/opt
[root@controller ~]# systemctl restart vsftpd
[root@controller ~]# systemctl enable vsftpd
```

## 在控制和计算节点安装ntp服务

```
##控制节点
[root@controller ~]# yum -y install chrony
[root@controller ~]# vi /etc/chrony.conf
server controller iburst 
#将其他的server注释
allow 10.0.0.0/24
local stratum 10

[root@controller ~]# systemctl restart chronyd
[root@controller ~]# systemctl enable chronyd
[root@controller ~] date 和计算节点同时输入

##计算节点
[root@compute ~]# yum -y install chrony
[root@compute ~]# vi /etc/chrony.conf
server controller iburst
#将其他的server注释
[root@compute ~]# systemctl restart chronyd
[root@compute ~]# systemctl enable chronyd
[root@compute ~] chronyd sources
[root@compute ~] date 和控制节点同时输入
```

## 在计算节点创建磁盘分区

```
[root@compute ~]# fdisk -l  //查看磁盘
[root@compute ~]# fdisk /dev/分区名字
n->p->1->回车->回车->w ## n：添加一个分区 P：主分区 两个回车指是开始和结束的磁盘扇区大小 w：写入磁盘
[root@compute ~]# fdisk -l //查看刚刚创建的分区
```

## 双节点安装Q版opeenstack（queens）

```
##controller节点
[root@controller ~]# yum install -y centos-release-openstack-queens
[root@controller ~]# yum install -y python-openstackclient
[root@controller ~]# yum install -y openstack-selinux
[root@compute ~] 同上面
```

## 控制节点mariadb数据库

```
[root@controller ~]# yum install -y mariadb mariadb-server python2-PyMySQL
[root@controller ~]# vi /etc/my.cnf.d/mariadb-server.cnf
bind-address = 0.0.0.0
[root@controller ~]# chown mysql:mysql -R /var/lib/mysql/
[root@controller ~]# systemctl enable mariadb.service
[root@controller ~]# systemctl start mariadb.service
[root@controller ~]# mysql_secure_installation #数据库初始化
MariaDB [(none)]>  GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '000000';
```

## 控制节点安装消息队列Memcached服务

```
[root@controller ~]# yum -y install memcached python-meemcached
[root@controller ~]# vi /etc/sysconfig/memcached 
#主要增加controller
OPTIONS="-l 127.0.0.1,::1,controller"
[root@controller ~]# systemctl start mecached
[root@controller ~]# systemctl enable memcached
```

## 控制节点安装etcd服务

```
[root@controller ~]# yum -y install etcd
[root@controller ~]# vi /etc/etcd/etcd.conf
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://10.0.0.60:2380"
ETCD_LISTEN_CLIENT_URLS="http://10.0.0.60:2379"
ETCD_NAME="controller"
ETCD_ADVERTISE_CLIENT_URLS="http://10.0.0.60:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.0.60:2380"
ETCD_INITIAL_CLUSTER="controller=http://10.0.0.60:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
[root@controller ~] systemctl enable etcd;systemctl start etcd
[root@controller ~] systemctl start etcd
```

# Openstack-Q版 Keystone安装

## 控制节点基础配置

```
[root@controller ~]# mysql -u root -p
MariaDB [(none)]> CREATE DATABASE keystone; ##c创建数据库
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '000000'; #给keystone用户赋予密码
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '000000';
[root@controller ~]# yum -y install openstack-keystone httpd mod_wsgi 
[root@controller ~]# vi /etc/keystone/keystone.conf
connection = mysql+pymysql://keystone:000000@controller/keystone #数据库连接
provider = fernet 
driver=memcached
[root@controller ~]# su -s /bin/sh -c "keystone-manage db_sync" keystone #同步数据库
[root@controller ~]# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone #初始化Fernet密钥存储库
[root@controller ~]# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
[root@controller ~]# keystone-manage bootstrap --bootstrap-password 000000 \
--bootstrap-admin-url http://controller:5000/v3/ \
--bootstrap-internal-url http://controller:5000/v3/ \
--bootstrap-public-url http://controller:5000/v3/ \
--bootstrap-region-id RegionOne
```

## 配置Apach HTTP服务

```
[root@controller ~]# vi /etc/httpd/conf/httpd.conf
ServerName controller
[root@controller ~]# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/ #添加软连接
[root@controller ~]# systemctl enable httpd.service
[root@controller ~]# systemctl start httpd.service
[root@controller ~]#  export OS_USERNAME=admin
					export OS_PASSWORD=000000
					export OS_PROJECT_NAME=admin
					export OS_USER_DOMAIN_NAME=Default
					export OS_PROJECT_DOMAIN_NAME=Default
					export OS_AUTH_URL=http://controller:35357/v3
					export OS_IDENTITY_API_VERSION=3
[root@controller ~]# netstat -ntlp #查看5000和3537端口是否正常启动
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      928/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1130/master         
tcp6       0      0 :::5000                 :::*                    LISTEN      1886/httpd          
tcp6       0      0 :::80                   :::*                    LISTEN      1886/httpd          
tcp6       0      0 :::22                   :::*                    LISTEN      928/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1130/master         

tcp6       0      0 :::35357                :::*                    LISTEN      1886/httpd  
！如果缺少一个就去增加配置,少那个就去增加那个端口的配置，别忘了重启
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
[root@controller ~]# curl 172.129.35.1:5000 #验证刚刚的操作是否正确
{"versions": {"values": [{"status": "stable", "updated": "2019-07-19T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.13", "links": [{"href": "http://10.0.0.100:5000/v3/", "rel": "self"}]}]}}
[root@controller ~]# curl 172.129.35.1:35357
{"versions": {"values": [{"status": "stable", "updated": "2019-07-19T00:00:00Z", "media-types": [{"base": "application/json", "type": "application/vnd.openstack.identity-v3+json"}], "id": "v3.13", "links": [{"href": "http://10.0.0.100:5000/v3/", "rel": "self"}]}]}}
```

## 配置keystone连接时的环境变量

```
[root@controller ~]# export OS_USERNAME=admin 
export OS_PASSWORD=000000
export OS_PROJECT_NAME=admin 
export OS_USER_DOMAIN_NAME=Default 
export OS_PROJECT_DOMAIN_NAME=Default 
export OS_AUTH_URL=http://10.0.0.60:35357/v3 
export OS_IDENTITY_API_VERSION=3
```



## openstack域、项目、用户、服务

```
[root@controller ~]# openstack domain create --description "demo" example #创建域
[root@controller ~]# openstack project create --domain default --description "Service Project" service #创建服务项目
[root@controller ~]# openstack project create --domain default --description "Demo Project" demo #创建Demo Project
[root@controller ~]# openstack user create --domain default --password-prompt keystone #创建keystone 用户
[root@controller ~]# openstack role create keystone #创建keystone 用户
[root@controller ~]# openstack role add --project demo --user DOMAIN_NAME user ##创建角色并且将角色添加到demo项目和用户 
```

## 创建脚本环境并验证

```
#创建admin-openrc脚本
[root@controller ~]# vi /root/admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=000000
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
#创建demo-openrc脚本
[root@controller ~]# vi /root/demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=000000
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

## 验证admin和demo返回的token

```
[root@controller ~]# unset OS_AUTH_URL OS_PASSWORD #取消临时环境变量
[root@controller keystone]# openstack --os-auth-url http://10.0.0.60:35357/v3 --os-project-domain-name Default --os-user-domain-name Default --os-project-name admin --os-username admin token issue
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
[root@controller ~]# openstack --os-auth-url http://10.0.0.60:5000/v3 \
> --os-project-domain-name Default --os-user-domain-name Default \
> --os-project-name demo --os-username demo token issue
```

# OpenStack—Q Glance安装

## 基础配置

```
[root@controller ~]# mysql -u root -p
MariaDB [(none)]> create database glance;
MariaDB [(none)]> grant all privileges on glance.* to 'glance'@'localhost' identified by '000000';
MariaDB [(none)]> grant all privileges on glance.* to 'glance'@'%' identified by '000000';
[root@controller ~]# . /root/admin-openrc #获取admin用户的环境变量，并创建服务认证
[root@controller ~]# openstack user create --domain default --password-prompt glance  #创建glance用户
[root@controller ~]# openstack role add --project service --user glance admin #把admin用户添加到glance用户和项目中
[root@controller ~]# openstack service create --name glance --description "OpenStack Image" image #创建glance服务
[root@controller ~]# openstack endpoint create --region RegionOne image public http://controller:9292 #创建API端点
[root@controller ~]# openstack endpoint create --region RegionOne image internal http://controller:9292
[root@controller ~]# openstack endpoint create --region RegionOne image internal http://controller:9292
```

## 控制节点安装glance组件

```
[root@controller ~]# yum install -y openstack-glance 
[root@controller ~]# vi /etc/glance/glance-api.conf
在[database]选项，配置数据库链接
[database]
connection = mysql+pymysql://glance:000000@controller/glance

在[[keystone_authtoken]和[paste_deploy]选项，配置身份服务访问
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
##auth_type这里别输入你的密码，应为password
auth_type = password    
project_domain_name = Default
user_domain_name = Default
project_name = service
##username为刚刚所创建的glance用户名
username = glance    
##password为刚刚创建glance用户设置的密码
password = 000000   

[paste_deploy]
flavor = keystone

在[glance_store]选项，配置本地文件存储和本地镜像文件的位置

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images
[root@controller ~]# vi /etc/glance/glance-registry.conf
在[database]选项，配置数据库链接
[database]
connection = mysql+pymysql://glance:000000@controller/glance

在[[keystone_authtoken]和[paste_deploy]选项，配置身份服务访问

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password          
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 000000

[paste_deploy]
flavor = keystone

[root@controller ~]# su -s /bin/sh -c "glance-manage db_sync" glance #同步数据库成功
[root@controller ~]# systemctl enable openstack-glance-api.service openstack-glance-registry.service
[root@controller ~]# systemctl restart openstack-glance-api.service openstack-glance-registry.service
```

## 验证glance服务

```
[root@controller ~]# . /root/admin-openrc
[root@controller ~]#  yum install -y wget #这里取下载一个镜像去验证我们做的glancee服务
[root@controller ~]# wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img 
[root@controller ~]# openstack image create "cirros" --file cirros-0.4.0-x86_64-disk.img \
--disk-format qcow2 --container-format bare --public
[root@controller ~]# openstack image list
```

# OpenStack—Q Nova安装

## 控制节点基础配置

```
[root@controller ~]# mysql -uroot -p
MariaDB [(none)]> create database nova_api;
MariaDB [(none)]> grant all privileges on nova_api.* to 'nova'@'localhost' identified by '000000'; 
MariaDB [(none)]> grant all privileges on nova_api.* to 'nova'@'%' identified by '000000';

MariaDB [(none)]> create database nova;
MariaDB [(none)]> grant all privileges on nova.* to 'nova'@'localhost' identified by '000000';
MariaDB [(none)]> grant all privileges on nova.* to 'nova'@'%' identified by '000000';

MariaDB [(none)]> create database nova_cell0;
MariaDB [(none)]> grant all privileges on nova_cell0.* to 'nova'@'localhost' identified by '000000';
MariaDB [(none)]> grant all privileges on nova_cell0.* to 'nova'@'%' identified by '000000';

[root@controller ~]#  . /root/admin-openrc #获取admin凭证以获得对admin-only CLI命令的访问
[root@controller ~]# openstack user create --domain default --password-prompt nova #创建nova用户
[root@controller ~]# openstack role add --project service --user nova admin #将admin角色添加到nova用户中
[root@controller ~]# openstack service create --name nova --description "OpenStack Compute" compute #创建实体
#创建Compute API服务端点
[root@controller ~]# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
[root@controller ~]# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
[root@controller ~]# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
#使用您选择的placementpass创建一个Placement service用户
[root@controller ~]# openstack user create --domain default --password-prompt placement 
#将Placement用户添加到服务项目中，并具有admin角色
[root@controller ~]# openstack role add --project service --user placement admin 
在服务目录中创建Place API入口
[root@controller ~]# openstack service create --name placement --description "Placement API" placement
创建Placement API服务端点
[root@controller ~]# openstack endpoint create --region RegionOne placement public http://controller:8778
[root@controller ~]# openstack endpoint create --region RegionOne placement internal http://controller:8778
[root@controller ~]# openstack endpoint create --region RegionOne placement admin http://controller:8778
```

## 控制节点安装和配置组件

```
[root@controller ~]# yum install -y openstack-nova-api openstack-nova-conductor \
openstack-nova-console openstack-nova-novncproxy \
openstack-nova-scheduler openstack-nova-placement-api
[root@controller ~]# vi /etc/nova/nova.conf
[DEFAULT]
enabled_apis = osapi_compute,metadata
transport_url = rabbit://openstack:000000@controller
my_ip = 10.0.0.60
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[api_database]
connection = mysql+pymysql://nova:000000@controller/nova_api

[database]
connection = mysql+pymysql://nova:000000@controller/nova

[api]
auth_strategy = keystone

[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 000000

[vnc]
enabled = true
server_listen = $my_ip
server_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292

[oslo_concurrency]
locak_path = /var/lib/nova/tmp

[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 000000
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```

```
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```

```
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```

```
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```

```
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```

```
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```



```
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
[root@controller ~]# 
```

