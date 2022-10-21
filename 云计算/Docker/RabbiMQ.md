```
FROM centos:centos7.5.1804
MAINTAINER Chinaskill
RUN mkdir /opt/rabbitmq/
ADD rabbitmq-repo.tar.gz /opt/rabbitmq/
ADD ftp.repo /etc/yum.repos.d/local.repo
RUN yum clean all
RUN yum -y install rabbitmq-server
EXPOSE 5672 15672
CMD ["/usr/sbin/rabbitmq-server"]

```

![image-20210419180327723](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210419180327723.png)