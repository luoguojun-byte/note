## 使用脚本安装docker

- 在 root 用户下执行安装脚本

  ```
  在network里面加上DNS=8.8.8.8
  curl -fsSL https://get.docker.com -o get-docker.sh
  sh get-docker.sh
  # cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
  
  # yum install –y wget
  # wget /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
  
  ```

- 如果出现以下错误就在网络增加DNS

  ```
  curl: (6) Could not resolve host: get.docker.com; Unknown error
  或者
  安装时候卡退
  ```

- 启动docker

  ```
  systemctl start docker
  systemctl status docker
  ```

- 安装好了后执行第一个测试镜像

  ```
  docker pull hello-world
  docker run hello-world
  docker images 
  ```

- 如果出现拉取慢可以去配置加速器

  ```
  https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
  #增加 /etc/docker/daemon.json配置文件
  tee /etc/docker/daemon.json <<-'EOF'
  {
    "registry-mirrors": ["https://vszcl2h8.mirror.aliyuncs.com"]
  }
  EOF
  systemctl daemon-reload
  systemctl restart docker
  ```

## 安装docker-compose

- 下载二进制文件可以更改版本

  ```
  curl -L https://github.com/docker/compose/releases/download/1.29.1/\
  docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  如果安装失败出现curl（6）可以试试下面这种方法
  wget https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m) -O /usr/local/bin/docker-compose
  ```

- 增加可执行权限

  ```
  chmod +x /usr/local/bin/docker-compose
  ```

- 测试版本

  ```
  docker-compose --version
  ```

## 安装Harbor

- 下载&&可以自己改版本

  ```
  wget https://github.com/goharbor/harbor/releases/download/v1.9.2/harbor-offline-installer-v1.9.2.tgz
  ```

- 解压

  ```
  tar -zxvf harbor-offline-installer-v1.9.2.tgz -C /usr/local/
  ```

- 修改配置文件

  ```
  cd /usr/local/harbor/
  ！！！这里要把原来的harbor.yml.tmpl名字改成 harbor.yml
  vim harbor.yml
  # 修改主机名 与 端口号
  hostname: harbor
  port:86
  ```

- 安装并开启 hlem charts 功能

  ```
  ./install.sh  --with-chartmuseum
  ```

- 启动和重启

  ```
  # 停止Harbor
  docker-compose down -v
   
  # 重启Harbor
  docker-compose up -d
  ```

  

