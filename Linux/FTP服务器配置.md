``` powershell
[root@T1 vsftpd]# service vsftpd restart
[root@T1 vsftpd]# grep -v "#" vsftpd.conf >vsftpd.conf1
[root@T1 vsftpd]# vi vsftpd.conf1
chroot_local_user=YES  #不能在访问系统文件了
anonymous_enable=NO  ##no的话是必须要账号密码才能访问FTP即匿名用户关闭


#自定义里匿名用户配置
anon_root=/var/ftp/pub/  #匿名用户的根目录
anon_upload_enable=YES	#匿名用户的上传配置
anon_mkdir_write_enable=YES	#匿名用户的创建文件夹的权限
anon_other_write_enable=NO #匿名用户的写入权限
anon_max_rate=50000	#匿名用户所上传的带宽权限 即5000字节

#自定义本地用户配置
chroot_list_enable=YES #根目录锁定
chroot_list_file=/etc/vsftpd/chroot_list #
vi chroot_list #在这里输入你要锁定的用户
userlist_enable=YES #用户的黑白名单
userlist_deny=NO  #用户的白名单在列表里面的用户才能登陆
vi user_list #编辑列表里面的人才能成为用户
userlist_file=/etc/vsftpd/user_list

#自定义链接配置
max_clients=100 #只能接受100客户端连接
max_per_ip=5  #每一专业只允许5个连接
idle_seession_timeout=300  #超时时间为300秒
~


[root@T1 ~]# chmod 777 -R /var/ftp/pub ##文件夹权限
[root@T1 ~]# vi /etc/selinux/config  ##关闭Selinux
[root@T1 ~]# service iptables stop ##关闭防火墙


修改默认自定义用户的目录可参考：https://www.cnblogs.com/feiyuanxing/p/5309627.html

```

![image-20201110084424183](E:%5C%E7%AC%94%E8%AE%B0%5CLinux%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20201110084424183.png)

![image-20201117042139942](E:%5C%E7%AC%94%E8%AE%B0%5CLinux%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20201117042139942.png)