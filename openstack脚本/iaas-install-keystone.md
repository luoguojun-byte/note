#!/bin/bash
source /etc/xiandian/openrc.sh
#keystone mysql
mysql -uroot -p$DB_PASS -e "create database IF NOT EXISTS keystone ;"
mysql -uroot -p$DB_PASS -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '$KEYSTONE_DBPASS' ;"
mysql -uroot -p$DB_PASS -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '$KEYSTONE_DBPASS' ;"

#install keystone
yum install openstack-keystone httpd mod_wsgi -y

#/etc/keystone/keystone.conf
crudini --set /etc/keystone/keystone.conf database connection  mysql+pymysql://keystone:$KEYSTONE_DBPASS@$HOST_NAME/keystone
crudini --set /etc/keystone/keystone.conf token provider  fernet
ADMIN_TOKEN=$(openssl rand -hex 10)
crudini --set /etc/keystone/keystone.conf DEFAULT admin_token $ADMIN_TOKEN
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
sed -i "s/#ServerName www.example.com:80/ServerName $HOST_NAME/g" /etc/httpd/conf/httpd.conf 
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/  
systemctl enable httpd.service
systemctl restart httpd.service

export OS_TOKEN=$ADMIN_TOKEN
export OS_URL=http://$HOST_NAME:35357/v3
export OS_IDENTITY_API_VERSION=3

openstack service create --name keystone --description "OpenStack Identity" identity
openstack endpoint create --region RegionOne identity public http://$HOST_NAME:5000/v3 
openstack endpoint create --region RegionOne identity internal http://$HOST_NAME:5000/v3
openstack endpoint create --region RegionOne identity admin http://$HOST_NAME:35357/v3

openstack domain create --description "Default Domain" $DOMAIN_NAME
openstack project create --domain $DOMAIN_NAME --description "Admin Project" admin
openstack user create --domain $DOMAIN_NAME --password $ADMIN_PASS admin

openstack role create admin
openstack role add --project admin --user admin admin

openstack project create --domain $DOMAIN_NAME --description "Service Project" service
openstack project create --domain $DOMAIN_NAME --description "Demo Project" demo

openstack user create --domain $DOMAIN_NAME --password $DEMO_PASS demo
openstack role create user
openstack role add --project demo --user demo user

unset OS_TOKEN OS_URL

cat > /etc/keystone/admin-openrc.sh <<-EOF
export OS_PROJECT_DOMAIN_NAME=$DOMAIN_NAME
export OS_USER_DOMAIN_NAME=$DOMAIN_NAME
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=$ADMIN_PASS
export OS_AUTH_URL=http://$HOST_NAME:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

cat > /etc/keystone/demo-openrc.sh <<-EOF
export OS_PROJECT_DOMAIN_NAME=$DOMAIN_NAME
export OS_USER_DOMAIN_NAME=$DOMAIN_NAME
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=$DEMO_PASS
export OS_AUTH_URL=http://$HOST_NAME:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF

source /etc/keystone/admin-openrc.sh 
openstack token issue
