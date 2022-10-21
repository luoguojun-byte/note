```
FROM centos:centos7.5.1804
MAINTAINER Chinaskill
RUN curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
RUN yum -y install redis
RUN sed -i -e 's@bind 127.0.0.1@bind 0.0.0.0@g' /etc/redis.conf
RUN sed -i -e 's@protected-mode yes@protected-mode no@g' /etc/redis.conf
EXPOSE 6379
#ENTRYPOINT redis-server /etc/redis.conf
#CMD ["redis-server"]
CMD ["/usr/bin/redis-server"]
##这两种方法都可以

```

![image-20210419180226147](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210419180226147.png)