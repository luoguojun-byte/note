```
show variables like "storage_enine";查看MySQL默认存储引擎信息
show variables like 'have%';查看MySQL支持的存储引擎信息有哪些
insert into mysql.user(host,user,password) values("localhos","examuser",password("000000"));登录到数据库中，在SQL中创建本地的用户，赋予它的权限是本地登录的权限，用户名是examuser，用户密码是000000
use mysql;进入数据库中
select host ,user,password from user;查询数据库中的这些字段
grant select,delete,update,create on *.*  to examuser@"localhost"  identified by "000000";赋予这个用户所有数据库的查询，删除，更新，创建的本地权限。
source /etc/keystone/admin-openrc.sh生效当前的环境变量
openstack user create --domain xiandian password passwork testuser通过openstack这个命令去创建一个用户，domain本身是xiandian，密码是password
openstack user set --disable testuser通过user set这个命令去将它的状态为关闭
openstack user show通过user show 去查看用户的显示信息
glance image -create --name "examimage" --disk-format "qcow2" --container-format bare --progress</opt/........>通过glance去创建一个镜像名称为examimage，它的磁盘的格式为qcow2
openstack image list查看信息，显示当前镜像列表
openstack image set examimage --tag lastone通过openstack set这个命令去打一个标签，先写镜像的名称后面跟上标签的名称
openstack image show examimage通过show去查看我们上传镜像的详细情况
nova quota -class default 通过nova去查看当前默认的配额是多少
nova -class-update --injecte...通过nova去更新下参数
nova quota-class-show default通过nova查看云平台默认每个项目实际配额的数量
cinder create --name volume 2通过cinder去创建一个云硬盘大小”
cinder readonly-mode-update volume true将刚刚创建的云硬盘设为只读模式，只读模式为确定的
cinder show volume查看刚刚创建的云硬盘
lvdisplay查看一下详细信息
cinder extend blockvolume 5对刚刚创建的云硬盘扩容大小为5G
swift capabilities | grep max_file_size通过swift这个命令去查看一下最大参数值，可以通过管道服务去过滤一下单个文件最大值是多少
nova list 查看一下当前云主机
nova show iaas查看云主机详情
ceilometer sample-list -m image通过ceilometer相关命令去查询它的一个镜像的测量值后面跟上 -m的参数再加上image可以看到它的镜像列表信息
ceilometer query-alarm-history可以查询告警的历史更改信息
heat resource-type-list查询到所有资源类型的列表信息
heat resource-type-showOS: :cinder: :Volume通过这个命令去show一下我们要重新要的cinder的详细资源的类型
cinder create --display-name 'unencryption volume' 1通过cinder命令去创建两个卷第一个不加密为 unen...大小为1G
cinder create --display-name 'encryption....'  --volume-type luks 1创建加密类型卷组在加密卷组跟上加密类型参数大小1G
systemctl status openstack-glance*通过systemctl这个命令去查询glance组件的状态信息
neutron agent-list -c binary -c agent_type -c alive通过neutron可以查看网络服务的列表信息在题目当中给出了显示的效果 可以通过-c来调整显示效果并打印列表信息状况
ps -e |grep kvm通过PS -e去查看kvm进程
taskset -cp 0-5 28694通过taskset这个命令将我们的kvm这个进程绑定到我们CPU上面0-5上面去
rabbitmqctl add_user xiandian-admin admin通过rabbitmqctl这个命令去添加一个用户名称为xiandian ，密码为admin
rabbitmqctl set_user_tags xiandian-admin adminstrator通过这个命令使用set将xiandian这个用户配置一个角色名称为admin...
rabbitmqctl list_users查看当前所有用户，可以显示角色的类型
rabbitmqctl set_permissions xiandian-admin ".*"  ".*" ".*"通过这个命令可以去对这个用户进行授权，对本机所以资源可读可写。
rabbitmqctl list_queues name .......通过这个命令查看它的队列信息，可以显示我们想要看的类容
ps aux |prep memcache通过ps aux参数去查到memcache服务的进程信息
vi /etc/sysconfig/memcache打开memcache的配置文件修改参数
systemctl restart memcache重启
cd /var/lib/glance/image/进入到根目录下，是存放所以镜像目录
qemu -img info +ID通过qemu去查看镜像的信息
grep -v "^$" /etc/nova...|grep -v [#]>/etc/nova...通过grep -v将首行为空的字符过滤，然后将首行字符为#进行过滤然后覆盖输出到etc的文件当中
swift post examcontainer 的容器
touch text.txt 创建一个文件
swift upload examcontainer text.txt上传文件到容器里
swift list examcontainer 查看列表容器
heat template -version-list通过heat命令去查询到当前版本的信息
heat template -function-list+版本名称   可以查询最新功能版本列表
curl-s -h "X-Auth-Token: openstack token issue | awk -F '| '  '/\sid\s/{print $3} ' " http://xiandian:9292/v2/image | Python -mjson.tool通过curl这个命令去获取镜像列表的信息需要用到token值，可以通过openstack去获取token值，这里可以将openstack嵌套命令当中。通过openstack token issue 命令在通过awk 进行下过滤，通过管道服务进行下过滤。然后将IE列进行输出就可以拿到它的token值，然后去向openstack端口号发送请求，镜像端口号为9292，请求它的镜像列表
cat /etc/sys/block/sda/queue/scheduler登录到控制节点中去看一下它当前的I/O算法是什么
echo noop > /sys/block/sda/queue/scheduler改成noop的算法，通过echo语句将它输出到文件中
mongo 进入数据库
show  dbs查看当前的数据库信息
use ceilometer进入数据库
show collections查看数据库下面的集合
show user 查看用户信息
use testexam创建testexam数据库
db.id.insert（{"id":"1"})
show  dbs查看当前的数据库信息
```

