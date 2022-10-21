```shell
wget  https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-ddevel readline-devel tk-devel python-devel mysql-devel gcc make
./cnfigure
make
make install
进入python的源目录替换为刚刚安装的
cd /usr/bin/
备份原python mv python python.bak
找到你安装的版本 which python版本
软连接  ln /usr/local/bin/python3.6 python
安装 MYSQL yum -y install mysql-server 
!!!注意 这里版本可能不一样之前修改过yum的下面也要修改
 vi /usr/libexec/urlgrabber-ext-down 
 
```

![image-20201130181730883](E:%5C%E7%AC%94%E8%AE%B0%5CLinux%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20201130181730883.png)

``` shell
启动mysql服务
mysql -u root
为root设置密码 SET PASSWORD FOR root@localhost=password('19510129');
ln /usr/local/bin/pip3 pip3.0
pip3.0 install mysqlclient
pip3.0 install Django==2.0.6
django-admin startproject Test #创建Django文件
修改 setting配置文件中的数据库片段并且更改时区

```

![image-20201201084202826](E:%5C%E7%AC%94%E8%AE%B0%5CLinux%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20201201084202826.png)

![image-20201201084811093](E:%5C%E7%AC%94%E8%AE%B0%5CLinux%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20201201084811093.png)

```powershell
关闭DEBUG和TEMPLATE_DEBUG
```

![image-20201201093328840](E:%5C%E7%AC%94%E8%AE%B0%5CLinux%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20201201093328840.png)