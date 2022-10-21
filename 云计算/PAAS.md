```
netstat -ntlp查看当前仓库监听服务的端口号
lsof -i：5000查看5000端口号的进程
yum - y  install losf 安装losf进程
top -p 12446查看12446pid使用情况
docker ps -a 查看所运行的容器
docker inspect -f '{{.State.pid}}' fa7f...看一下进程号 
mkdir /var/run/netns创建一个命名空间
ln - s /proc/13788/ns/net /var/run/netns/rancher-server用软连接的方式将刚才查询到的ID进行连接
ip netns list通过这个命令可以看到刚刚连接的
ip netns exec rancher-server ip a查看一下容器的IP地址
mount -t cgroup 直接查看cgroup挂载的情况
mkdir /sys/fs/cgroup/memory/xiandian创建一个cgroup名称为xiandian
echo $$输出当前的进程号
sudo sh -c 'echo $$>> /sys/fs/cgroup/xiandian/tasks'将当前的进程号重新输出到刚刚创建的cgroup当中
cat /proc/21661/cgroup通过cat命令去查看cgroup的进程Id
mkdir /opt/xiandian创建一个xiandian目录
docker run -d -it -p --name xiandian-dir -v /opt/xiandian/nginx:latest /bin/bash启动一个名称为xiandian-dir   -v是指定创建的目录为启动卷目录
docker inspect -f '{{.confing.volumes}}' xiandian-dir使用docker inspect -f固定格式去查看刚刚创建的xiandian-dir这个名字
mkdir /opt/xiandian-ro创建一个xiandian-ro目录
docker run -t -it -p --name xiandian -v /opt/xiandian-ro：/opt：ro nginx：latest /bin/bash运行下容器名称为xiandian 通过-v指定目录opt，成为运行容器的opt目录，设置为只读，镜像采用nginx
docker inspect -f '{{.HostConfig.Binds}}'xiandian运行docker ，指定查看hostconfig内的binds
docker run -d -it --name mysqldb -p mysql：8.0 /bin/bash运行下容器建立一个名称为mysql 通过-p指定镜像
docker run -t -it --name nginxweb -p --link mysqldb：db nginx：latest /bin/bash 运行下nginx外部的容器让连接到刚刚运行的mysqldb中的db，使用nginx镜像，再通过docker inspect -f查看连接内容的字段
yum -y install bridge-utils 安装命令，查看网桥
brctl show 查看网桥列表信息
brctl addbr xd_br通过这命令去添加一个 xd_br这个网桥
ip addr 192.168.2.1/24 dev xd_br在通过ip add将ip添加到网桥上
ip link set xd_br up将刚刚创建的网桥启动
brctl show 看到网桥
ifconfig xd_br去查一下网桥的详细信息
docker run -d -it --net=none nginx：latest /bin/bash运行下一个无网络环境的容器
docker ps -a查看容器
docker run -it -d -p MYSQL_ROOT_PASSWORD=000000 mysql：8.0运行下数据库容器，数据库密码设置为000000镜像使用mysql
docker ps -a查看容器
docker export +ID > /media/mysql_container.tar通过export命令将容器导出，导出名称为mysql.....放在目录/media下
ll/media查看目录
docker stop +ID将运行的容器停止，然后再查看通过docker pa-a查看容器状态
docker exec +ID netstat -ntlp查看网络套接字的连接情况
docker top +ID查看容器进程
docker commit +ID mysql_new：latest通过commit命令将容器创建一个为镜像，镜像名称为my....
docker images查看镜像
docker network list列举所有的网络
docker network create --subnet=192.168.3.0/24 --ip-range=192.168.3.0/24 --gateway=192.168.3.1 xd_net创建一个网络设置它的网络配置，网络名称为xd_net
docker network ls查看网络列表
docker stats +ID查看容器的统计信息
docker run -itdp --net=xd_net --name centos centos：latest运行容器网络选用刚刚创建的网络，名称为centos 镜像centos：latest
docker port +ID查看容器端口状态
docker run centos：latest /bin/echo "Hello Word"运行镜像，输出打印hello word
docker cp +ID：/opt /  /media/拷贝容器内的opt到宿主机的media下
docker cp /etc/yum.repos.d/local.repo+ID  :/opt将当前系统的操作系统yum源拷贝到容器内opt目录下
docker exec -it +ID /bin/ls /opt通过exec命令去查询容器内opt目录下所有文件列表
docker volume ls 查看当前系统使用的卷组信息
docker volume create --xd_volume创建一个卷组名称为xd...
docker run -itd -v xd_volume：/root/ centos：latest运行容器将刚刚创建的卷组挂载到root分区
docker build -t 172.30.11.4：5000/http：v1.0通过bulid命令去构建镜像，镜像名称为http：1.0
docker image 查看镜像列表
curl -X GET http：//localhost：2375/info查看系统信息
curl -X GET http://localhost:2375/version查看docker版本
curl -X GET http://localhost:2375/containers/json/查看docker内所有容器
```

