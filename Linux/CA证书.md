``` shell
rpm –qa|grep openssl       //检查是否已安装 OpenSSL 软件
rpm –qa|grep mod_ssl       //检查是否已安装 mod_ssl 模块
 cd /etc/pki/tls 
 cp openssl.cnf openssl.cnf.bak    //备份 OpenSSL 主配置文件 
 vim openssl.cnf        //编辑 OpenSSL 主配置文件
 找到[ policy_match ] ,前三行修改为
 countryName     = optional 
 stateOrProvinceName   = optional 
 organizationName   = optional
 找到[ req_distinguished_name ] ，修改下面几行
 countryName_default    = CN 
 stateOrProvinceName_default  = ZJ 
 localityName_default   = HZ 
 0.organizationName_default  = xinyuan 
 ##创建证书数据库和序列号
 cd /etc/pki/CA 
 touch index.txt       //创建空文件 index.txt 
 echo "01"  >serial       //创建 serial 文件并置内容为“01” 
 ll          //该命令等同于“ls -l”命令
 ##生成CA服务器和私钥证书文件
 /etc/pki/CA 
 openssl genrsa 1024 > private/cakey.pem  //显示下列信息则表明私钥已生成
 openssl req –new –key private/cakey.pem –x509 –out cacert.pem –days 3650
 //根据自己的私钥文件生成证书文件，其中-x509 是专用于 CA 生成自签名证书的选项(如   
 //果不是自签名证书则无须该选项)，显示以下信息以及提示用户确认或输入信息
 #修改权限
 /etc/pki/CA 
 chmod 600 cacert.pem       //修改证书文件的权限为 600 
 chmod 600 private/cakey.pem     //修改私钥文件的权限为 600 
 ##为web颁发证书
 cd /etc/httpd
 mkdir certs 
 cd certs          //使当前目录为/etc/httpd/certs 
 openssl genrsa 1024 > httpd.key 
 /etc/httpd/certs
 openssl req –new –key httpd.key –out httpd.csr   
 //根据私钥生成证书请求文件，显示以下信息以及提示用户确认或输入信息 
  openssl ca –in httpd.csr –out httpd.cert 
  ##将Web站点要求为SSl访问
  cd /etc/httpd/conf.d 
  cp ssl.conf ssl.conf.bak
  vim ssl.conf 
  #找到SSLCertificateFile /etc/pki/tls/certs/localhost.crt 和 
  SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
  SSLCertificateChainFile /etc/pki/tls/certs/server-chain.crt
  修改成 
  SSLCertificateFile /etc/httpd/certs/httpd.cert 
  SSLCertificateKeyFile /etc/httpd/certs/httpd.key 
  SSLCertificateChainFile /etc/pki/CA/cacert.pem 
  ##最后重启
  service httpd restart 
```

