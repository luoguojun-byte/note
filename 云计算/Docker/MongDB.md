```
FROM centos:centos7.5.1804
MAINTAINER Chinaskill
RUN mkdir -p /root/MongDB
RUN yum -y install net-snmp
ADD mongodb-4.4.5 /root/MongDB/mongodb-4.4.5
RUN mkdir /root/MongDB/mongodb-4.4.5/{data,logs}
ENV PATH /root/MongDB/mongodb-4.4.5/bin:$PATH
WORKDIR /root/MongDB/mongodb-4.4.5/bin/
ADD mongdb.conf .
EXPOSE 27017
CMD ["mongod","-f"," mongdb.conf"]
```

![image-20210419184051670](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210419184051670.png)

## 准备东西

```
vi mongdb.conf
port=27017
bind_ip=0.0.0.0
dbpath=/root/MongDB/mongodb-4.4.5/
logpath=/root/MongDB/mongodb-4.4.5/logs/mongodb.log
fork=true

```

![image-20210420090950467](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210420090950467.png)

