#!/bin/bash
source /etc/xiandian/openrc.sh
ping $HOST_IP -c 4 >> /dev/null 2>&1
if [ 0  -ne  $? ]; then
        echo -e "\033[31m Warning\nPlease make sure the network configuration is correct!\033[0m"
        exit 1
fi

#  MariaDB
yum install -y mariadb mariadb-server python2-PyMySQL
sed -i  "/^symbolic-links/a\default-storage-engine = innodb\ninnodb_file_per_table\ncollation-server = utf8_general_ci\ninit-connect = 'SET NAMES utf8'\ncharacter-set-server = utf8\nmax_connections=10000" /etc/my.cnf
sed -i 's/plugin-load-add*/#plugin-load-add/g' /etc/my.cnf.d/auth_gssapi.cnf  
crudini --set /usr/lib/systemd/system/mariadb.service Service LimitNOFILE 10000
crudini --set /usr/lib/systemd/system/mariadb.service Service LimitNPROC 10000
systemctl daemon-reload
systemctl enable mariadb.service
systemctl restart mariadb.service

expect -c "
spawn /usr/bin/mysql_secure_installation
expect \"Enter current password for root (enter for none):\"
send \"\r\"
expect \"Set root password?\"
send \"y\r\"
expect \"New password:\"
send \"$DB_PASS\r\"
expect \"Re-enter new password:\"
send \"$DB_PASS\r\"
expect \"Remove anonymous users?\"
send \"y\r\"
expect \"Disallow root login remotely?\"
send \"n\r\"
expect \"Remove test database and access to it?\"
send \"y\r\"
expect \"Reload privilege tables now?\"
send \"y\r\"
expect eof
"

# RabbitMQ
yum install rabbitmq-server -y
systemctl start rabbitmq-server.service
systemctl enable rabbitmq-server.service

rabbitmqctl add_user $RABBIT_USER $RABBIT_PASS
rabbitmqctl set_permissions $RABBIT_USER ".*" ".*" ".*"

# Memcache
yum install memcached python-memcached -y
sed -i  -e 's/OPTIONS.*/OPTIONS="-l 127.0.0.1,::1,'$HOST_NAME'"/g' /etc/sysconfig/memcached

systemctl start memcached.service
systemctl enable memcached.service


# ETCD
yum install etcd -y
sed -i -e 's/#ETCD_LISTEN_PEER_URLS.*/ETCD_LISTEN_PEER_URLS="http:\/\/'$HOST_IP':2380"/g' \
-e 's/^ETCD_LISTEN_CLIENT_URLS.*/ETCD_LISTEN_CLIENT_URLS="http:\/\/'$HOST_IP':2379"/g' \
-e 's/^ETCD_NAME="default"/ETCD_NAME="'$HOST_NAME'"/g' \
-e 's/#ETCD_INITIAL_ADVERTISE_PEER_URLS.*/ETCD_INITIAL_ADVERTISE_PEER_URLS="http:\/\/'$HOST_IP':2380"/g' \
-e 's/^ETCD_ADVERTISE_CLIENT_URLS.*/ETCD_ADVERTISE_CLIENT_URLS="http:\/\/'$HOST_IP':2379"/g' \
-e 's/#ETCD_INITIAL_CLUSTER=.*/ETCD_INITIAL_CLUSTER="'$HOST_NAME'=http:\/\/'$HOST_IP':2380"/g' \
-e 's/#ETCD_INITIAL_CLUSTER_TOKEN.*/ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"/g' \
-e 's/#ETCD_INITIAL_CLUSTER_STATE.*/ETCD_INITIAL_CLUSTER_STATE="new"/g' /etc/etcd/etcd.conf
systemctl start etcd
systemctl enable etcd
