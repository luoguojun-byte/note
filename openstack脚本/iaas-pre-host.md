#!/bin/bash
source /etc/xiandian/openrc.sh

#selinux
sed -i 's/SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
setenforce 0

#firewalld
systemctl stop firewalld
systemctl disable firewalld  >> /dev/null 2>&1

#NetworkManager
systemctl stop NetworkManager >> /dev/null 2>&1
systemctl disable NetworkManager >> /dev/null 2>&1
yum remove -y NetworkManager firewalld
systemctl restart network

#iptables
yum install  iptables-services  -y 
if [ 0  -ne  $? ]; then
	echo -e "\033[31mThe installation source configuration errors\033[0m"
	exit 1
fi
systemctl restart iptables
iptables -F
iptables -X
iptables -Z 
/usr/sbin/iptables-save
systemctl stop iptables
systemctl disable iptables

# install package 
sed -i -e 's/#UseDNS yes/UseDNS no/g' -e 's/GSSAPIAuthentication yes/GSSAPIAuthentication no/g' /etc/ssh/sshd_config
yum upgrade -y
yum install python-openstackclient openstack-selinux openstack-utils crudini expect -y

#hosts
if [[ `ip a |grep -w $HOST_IP ` != '' ]];then 
    hostnamectl set-hostname $HOST_NAME
elif [[ `ip a |grep -w $HOST_IP_NODE ` != '' ]];then 
    hostnamectl set-hostname $HOST_NAME_NODE
else
    hostnamectl set-hostname $HOST_NAME
fi
sed -i -e "/$HOST_NAME/d" -e "/$HOST_NAME_NODE/d" /etc/hosts
echo "$HOST_IP $HOST_NAME" >> /etc/hosts
echo "$HOST_IP_NODE $HOST_NAME_NODE" >> /etc/hosts

#ssh
if [[ ! -s ~/.ssh/id_rsa.pub ]];then
    ssh-keygen  -t rsa -N '' -f ~/.ssh/id_rsa -q -b 2048
fi
name=`hostname`
if [[ $name == $HOST_NAME ]];then
expect -c "set timeout -1;
               spawn ssh-copy-id  -i /root/.ssh/id_rsa $HOST_NAME_NODE;
               expect {
                   *password:* {send -- $HOST_PASS_NODE\r;
                        expect {
                            *denied* {exit 2;}
                            eof}
                    }
                   *(yes/no)* {send -- yes\r;exp_continue;}
                   eof         {exit 1;}
               }
               "
else
expect -c "set timeout -1;
               spawn ssh-copy-id  -i /root/.ssh/id_rsa $HOST_NAME;
               expect {
                   *password:* {send -- $HOST_PASS\r;
                        expect {
                            *denied* {exit 2;}
                            eof}
                    }
                   *(yes/no)* {send -- yes\r;exp_continue;}
                   eof         {exit 1;}
               }
               "
fi

#chrony
yum install -y chrony
if [[ $name == $HOST_NAME ]];then
        sed -i '3,6s/^/#/g' /etc/chrony.conf
        sed -i '7s/^/server controller iburst/g' /etc/chrony.conf
        echo "allow $network_segment_IP" >> /etc/chrony.conf
        echo "local stratum 10" >> /etc/chrony.conf
else
        sed -i '3,6s/^/#/g' /etc/chrony.conf
        sed -i '7s/^/server controller iburst/g' /etc/chrony.conf
fi

systemctl restart chronyd
systemctl enable chronyd

#DNS
if [[ $name == $HOST_NAME ]];then
yum install bind -y
sed -i -e '13,14s/^/\/\//g' \
-e '19s/^/\/\//g' \
-e '37,42s/^/\/\//g' \
-e 's/recursion yes/recursion no/g' \
-e 's/dnssec-enable yes/dnssec-enable no/g' \
-e 's/dnssec-validation yes/dnssec-validation no/g' /etc/named.conf 
systemctl start named.service
systemctl enable named.service
fi
printf "\033[35mPlease Reboot or Reconnect the terminal\n\033[0m"
