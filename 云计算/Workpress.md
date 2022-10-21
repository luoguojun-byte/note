## 安装Apache Web服务器

```

yum -y install httpd
systemctl start httpd.service
systemctl enable httpd.service
```

## 在浏览器访问

```
http://IP地址
如果出错就去防火墙放行
iptables -I INPUT -p TCP --dport 80 -j ACCEPT
```

![image-20210417180023063](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210417180023063.png)

## 安装Mariadb数据库

```
yum -y install mariadb-server mariadb
systemctl start mariad
初始化数据库1
mysql_secure_installation
In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
Enter current password for root (enter for none): //按下回车
Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.
Set root password? [Y/n] Y
New password: ///你的密码
Re-enter new password: //重复密码
Password updated successfully!
Reloading privilege tables..
 ... Success!
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.
Remove anonymous users? [Y/n] Y
 ... Success!
By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.
Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
Reload privilege tables now? [Y/n] Y
 ... Success!
Cleaning up...
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!

systemctl enable mariadb.service ##开机自启动
```

## 安装PHP

```
rpm  -Uvh   https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm  -Uvh   https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
yum -y install ph72w php-mysql
yum  -y  install  php72w-cli  php72w-common  php72w-devel
yum  install  yum-utils –y
yum  install  php-mysqli
yum  install  php72w-fpm
systemctl enable php-fpm.service
systemctl start php-fpm.service
vi /var/www/html/info.php ##创建一个php来看我们php的安装情况
<?php phpinfo(); ?> ##添加hph信息
systemctl restart httpd.service ##完成重启http
访问 http://IP地址/info.php
```

![image-20210417221418016](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210417221418016.png)

## 安装Wordpress&&创建Workprss数据库

```
mysql -u root -p
CREATE DATABASE wordpress;
#创建数据库用户和密码
CREATE USER wordpressuser@localhost IDENTIFIED BY '000000';
#设置wordpressuser访问wordpress数据库权限
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost IDENTIFIED BY '000000';
#刷新数据库设置
FLUSH PRIVILEGES;
#退出数据库
exit
wget http://wordpress.org/latest.tar.gz ##下载Workpreess

然后解压到/var/www/html/wordpress目录 ##没有目录就去创建
tar -xzvf latest.tar.gz -C /var/www/html/wordpreess/

编辑wp-config.php文件
先备份一份
cp wp-config-sample.php wp-config-sample.php.bak
mv  

define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wordpressuser' );

/** MySQL database password */
define( 'DB_PASSWORD', '000000' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

![image-20210417221109571](C:\Users\luo\AppData\Roaming\Typora\typora-user-images\image-20210417221109571.png)

