## 创建Dockerfie

```
mkdir mysql
cd mysql
vim Dockerfile
将软件包考入当前目录：mysql-5.6.26.tar.gz
软件包地址
http://mirrors.sohu.com/mysql/MySQL-5.6/
```

## 具体类容

```
FROM centos:7
MAINTAINER Luo<3109252978@qq.com>
RUN yum -y update && yum -y install gcc gcc-c++ make pcre-devel expat-devel perl ncurses ncurses-devel bison cmake autoconf
ADD mysql-5.6.39.tar.gz /opt/
RUN useradd -s /sbin/nologin mysql
#进入源码包，执行cmake文件，指定工作目录
WORKDIR /opt/mysql-5.6.39
RUN cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DEXTRA_CHARSETS=all \
-DSYSCONFIDIR=/etc \
-DMYSQL_DATADIR=/home/mysql/ \
-DMYSQL_UNIX_ADDR=/home/mysql/mysql.sock   
#make编译
RUN make && make install
#复制默认配置文件到/etc目录下
RUN cp -f support-files/my-default.cnf /etc/my.cnf
RUN mkdir -p /usr/local/mysql/
RUN chown -R mysql:mysql /usr/local/mysql/
#配置mysql环境变量
ENV PATH /usr/local/mysql/bin:$PATH
#初始化数据库
RUN /usr/local/mysql/scripts/mysql_install_db \
--user=mysql \
--ldata=/var/lib/mysql \
--basedir=/usr/local/mysql \
--datadir=/home/mysql
#暴露服务端口
EXPOSE 3306
#启动
RUN cp support-files/mysql.server /etc/init.d/mysqld
RUN chmod 755 /etc/init.d/mysqld
RUN sed -i '/^basedir=/s#basedir=#basedir=/usr/local/mysql#' /etc/init.d/mysqld  
RUN sed -i '/^datadir=/s#datadir=#basedir=/home/mysql#' /etc/init.d/mysqld
#建立软链接
RUN ln -s /var/lib/mysql/mysql.sock /home/mysql/mysql.sock
ENTRYPOINT ["/usr/local/mysql/bin/mysqld_safe"]
```

## 容器运行测试

```
#容器创建
docker build -t luo:1.3 .
#容器运行
docker run -d -P centos7:mysql --privileged
#进入容器
docker exec -it 容器ID bash
```

```
#登录MySQL
mysql -u root -p 
#grant授权
grant all privileges on *.* to 'root'@'%' identified by '000000';    
grant all privileges on *.* to 'root'@'localhost' identified by '000000';
#创建wordpress数据库
create database  wordpress;
create user wordpressuser@localhost identified by '000000';
grant all  privileges on wordpress.* to wordpressuser@localhost identified by '000000';
#刷新权限
flush privileges; 
```

