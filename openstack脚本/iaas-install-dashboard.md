#!/bin/bash
source /etc/xiandian/openrc.sh

#dashboard install
yum install openstack-dashboard -y 

#/etc/openstack-dashboard/local_settings
sed -i -e "s/^ALLOWED_HOSTS.*/ALLOWED_HOSTS = ['*', 'two.example.com']/g" \
-e 's/^OPENSTACK_HOST.*/OPENSTACK_HOST = "'$HOST_NAME'"/g' \
-e 's/#OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT.*/OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True/g' \
-e 's/^OPENSTACK_KEYSTONE_URL .*/OPENSTACK_KEYSTONE_URL = "http:\/\/%s:5000\/v3" % OPENSTACK_HOST/g' \
-e 's/#OPENSTACK_KEYSTONE_DEFAULT_DOMAIN.*/OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"/g' \
-e 's/^OPENSTACK_KEYSTONE_DEFAULT_ROLE.*/OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"/g' /etc/openstack-dashboard/local_settings
echo "SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': '$HOST_NAME:11211',
    }
}
OPENSTACK_API_VERSIONS = {
    "\"identity"\": 3,
    "\"image"\": 2,
    "\"volume"\": 2,
}" >> /etc/openstack-dashboard/local_settings

#/etc/httpd/conf.d/openstack-dashboard.conf
sed -i -e 'N;4aWSGIApplicationGroup %{GLOBAL}' /etc/httpd/conf.d/openstack-dashboard.conf
systemctl restart httpd.service memcached.service
