# SSH配置

[AR1]interface GigabitEthernet 0/0/0

[AR1-GigabitEthernet0/0/0]ip address 100.1.1.1 30

在AR2做同样的配置

[AR1]rsa local-key-pair ？ 创建本地密钥

[AR1]stelnet server enable 启动服务器

AR1]user-interface vty 0 4 进入虚拟用户接口

[AR1-ui-vty0-4]protocol inbound ? 设置ssh协议

[AR1-ui-vty0-4]authentication-mode aaa 认证

[AR1]aaa 进入aaa的配置模式

[AR1-aaa]local-user ? 设置用户名

[AR1-aaa]local-user user1 password cipher admin@huawei 设置用户和密码

[AR1-aaa]local-user user1 service-type ? 设置服务类型

[AR1-aaa]local-user user1 privilege level ? 设置用户等级

[AR1]ssh user user1 authentication-type password 设置用户认证类型

[AR2]ssh client first-time enable 将首次访问启动起来

[AR1]stelnet server enable 开启AR1服务器

[AR1]display ssh server status 查看AR1的状态