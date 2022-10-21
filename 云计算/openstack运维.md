# 基本环境安装

## 配置网络、主机名

```
修改和添加/etc/sysconfig/network-scripts/ifcfg-enp*（具体的网口）文件。
（1）controller节点
配置网络：
enp8s0: 192.168.100.10
DEVICE=enp8s0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.100.10
PREFIX=24
GATEWAY=192.168.100.1

enp9s0: 192.168.200.10
DEVICE=enp9s0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.200.10
PREFIX=24
配置主机名：
# hostnamectl set-hostname controller
按ctrl+d 退出  重新登陆

（2）compute 节点
配置网络：
enp8s0: 192.168.100.20
DEVICE=enp8s0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.100.20
PREFIX=24
GATEWAY=192.168.100.1

enp9s0: 192.168.200.20
DEVICE=enp9s0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=no
BOOTPROTO=static
IPADDR=192.168.200.20
PREFIX=24

配置主机名：
# hostnamectl set-hostname compute
按ctrl+d 退出  重新登陆
```

## 配置yum源

```
#Controller和compute节点
（1）yum源备份
#mv /etc/yum.repos.d/*  /opt/
（2）创建repo文件
【controller】
在/etc/yum.repos.d创建centos.repo源文件
[centos]
name=centos
baseurl=file:///opt/centos
gpgcheck=0
enabled=1
[iaas]
name=iaas
baseurl=file:///opt/iaas-repo
gpgcheck=0
enabled=1

【compute】
在/etc/yum.repos.d创建centos.repo源文件
[centos]
name=centos
baseurl=ftp://192.168.100.10/centos
gpgcheck=0
enabled=1
[iaas]
name=iaas
baseurl=ftp://192.168.100.10/iaas-repo
gpgcheck=0
enabled=1

（3）挂载iso文件
【挂载CentOS-7-x86_64-DVD-1804.iso】
[root@controller ~]# mount -o loop CentOS-7-x86_64-DVD-1804.iso  /mnt/
[root@controller ~]# mkdir /opt/centos
[root@controller ~]# cp -rvf /mnt/* /opt/centos/
[root@controller ~]# umount  /mnt/

【挂载XianDian-IaaS-v2.4.iso】
[root@controller ~]# mount -o loop XianDian-IaaS-v2.4.iso  /mnt/
[root@controller ~]# cp -rvf /mnt/* /opt/
[root@controller ~]# umount  /mnt/

（4）搭建ftp服务器，开启并设置自启
[root@controller ~]# yum install vsftpd -y
[root@controller ~]# vi /etc/vsftpd/vsftpd.conf
添加anon_root=/opt/
保存退出

[root@controller ~]# systemctl start vsftpd
[root@controller ~]# systemctl enable vsftpd

（5）配置防火墙和Selinux
【controller/compute】
编辑selinux文件
# vi /etc/selinux/config
SELINUX=permissive
关闭防火墙并设置开机不自启
# systemctl stop firewalld.service
# systemctl disable firewalld.service
# yum remove -y NetworkManager firewalld
# yum -y install iptables-services
# systemctl enable iptables
# systemctl restart iptables
# iptables -F
# iptables -X
# iptables -Z
# service iptables save

（6）清除缓存，验证yum源
【controller/compute】
# yum clean all
# yum list
```

## 编辑环境变量

```
controller和compute节点
# yum install iaas-xiandian -y
编辑scp装过程中的各项参数，根据每项参数上一行的说明及服务器实际情况进行配置。
HOST_IP=192.168.100.10
HOST_PASS=000000
HOST_NAME=controller
HOST_IP_NODE=192.168.100.20
HOST_PASS_NODE=000000
HOST_NAME_NODE=compute
network_segment_IP=192.168.100.0/24
RABBIT_USER=openstack
RABBIT_PASS=000000
DB_PASS=000000
DOMAIN_NAME=demo
ADMIN_PASS=000000
DEMO_PASS=000000
KEYSTONE_DBPASS=000000
GLANCE_DBPASS=000000
GLANCE_PASS=000000
NOVA_DBPASS=000000
NOVA_PASS=000000
NEUTRON_DBPASS=000000
NEUTRON_PASS=000000
METADATA_SECRET=000000
INTERFACE_IP=192.168.100.10/192.168.100.20（controllerIP/computeIP）
INTERFACE_NAME=enp9s0 （外部网络网卡名称）
Physical_NAME=provider （外部网络适配器名称）
minvlan=101 （vlan网络范围的第一个vlanID）
maxvlan=200 （vlan网络范围的最后一个vlanID）
CINDER_DBPASS=000000
CINDER_PASS=000000
BLOCK_DISK=md126p4 （空白分区）
SWIFT_PASS=000000
OBJECT_DISK=md126p5 （空白分区）
STORAGE_LOCAL_NET_IP=192.168.100.20
HEAT_DBPASS=000000
HEAT_PASS=000000
ZUN_DBPASS=000000
ZUN_PASS=000000
KURYR_DBPASS=000000
KURYR_PASS=000000
CEILOMETER_DBPASS=000000
CEILOMETER_PASS=000000
AODH_DBPASS=000000
AODH_PASS=000000
```

## 通过脚本安装服务

```
Controller节点和Compute节点
执行脚本iaas-pre-host.sh进行安装
```

## 配置域名解析

```
（1）controller 节点
192.168.100.10   controller
192.168.100.20   compute
（2）compute 节点
192.168.100.10   controller
192.168.100.20   compute
```

## 安装chrony服务

```
（2）配置controller节点
编辑/etc/chrony.conf文件
添加以下内容（删除默认sever规则）
server controller iburst
allow 192.168.100.0/24
local stratum 10
启动ntp服务器
# systemctl restart chronyd
# systemctl enable chronyd
（3）配置compute节点
编辑/etc/chrony.conf文件
添加以下内容（删除默认sever规则）
server controller iburst
启动ntp服务器
# systemctl restart chronyd
# systemctl enable chronyd
```

## 通过脚本安装数据库服务

```
Controller节点
执行脚本iaas-install-mysql.sh进行安装
```

# 安装Keystone认证服务

##  通过脚本安装keystone服务

```
Controller节点
执行脚本iaas-install-keystone.sh进行安装。
```

# 安装Glance镜像服务

## 通过脚本安装glance服务

```
Controller 节点
执行脚本iaas-install-glance.sh进行安装
```

# 安装Nova计算服务

## 通过脚本安装nova服务

```
#Controller节点
执行脚本iaas-install-nova-controller.sh进行安装
#Compute节点
执行脚本iaas-install-nova-compute.sh进行安装
```

# 安装Neutron网络服务

## 通过脚本安装neutron服务

```
#Controller节点
执行脚本iaas-install-neutron-controller.sh进行安装
#Compute节点
执行脚本iaas-install-neutron-compute.sh进行安装
```

# 安装Dashboard服务

## 通过脚本安装dashboard服务

```
#Controller
执行脚本iaas-install-dashboard.sh进行安装
```

# 安装Cinder块存储服务

## 通过脚本安装Cinder服务

```
#Controller
执行脚本iaas-install-cinder-controller.sh进行安装
#Compute节点
执行脚本iaas-install-cinder-compute.sh进行安装
```

# 安装Swift对象存储服务

## 通过脚本安装Swift服务

```
#Controller
执行脚本iaas-install-swift-controller.sh进行安装
#Compute节点
执行脚本iaas-install-swift-compute.sh进行安装
```

```

```

```

```

```

```

```

```

## openstack 私 有 云 平 台 上 ， 基 于 cirros-0.3.4-x86_64-disk.img 镜 像 ， 使 用 命 令 创 建 一 个 名 为cirros 的镜像。完成后提交控制节点的用户名、密码和 IP 地址到答 题框

```
 openstack image create "crrors" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 \
> --container-format bare --public
## 参考
glance image-create --name "cirros" --disk-format qcow2 --container-format bare --progress <cirros-0.3.4-x86_64-disk.img 

+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | ee1eca47dc88f4879d8a229cc70a07c6                     |
| container_format | bare                                                 |
| created_at       | 2021-04-20T12:38:55Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/608e1017-723b-4cf5-a6f6-940048a609f0/file |
| id               | 608e1017-723b-4cf5-a6f6-940048a609f0                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | crrors                                               |
| owner            | ce29f19c83064165831e0224adcc7c3e                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13287936                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2021-04-20T12:38:55Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
openstack image show crrors
```

## 在 openstack 私有云平台上，使用命令创建一个名为 Fmin，ID 为 1，内存为 1024 MB，磁盘为 10 GB，vcpu 数量为 1 的云主机类型

```
nova flavor-create Fmin  1 1024 10 1
+----+------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
| ID | Name | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public | Description |
+----+------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
| 1  | Fmin | 1024      | 10   | 0         |      | 1     | 1.0         | True      | -           |
+----+------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
```

```
 openstack network create --project admin --provider-network-type flat --provider-physical-network provider --external  extnet
 openstack subnet create --network extnet --subnet-range 192.168.200.0/24 --gateway 192.168.200.1 --allocation-pool start=192.168.200.100,end=192.168.200.200 --dhcp extsubnet

```

