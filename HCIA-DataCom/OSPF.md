# OSPF概述

### 链路状态协议——LSA泛洪

- 不在通告路由信息，而是LSA
- LSA描述了路由器的接口的状态信息，列如接口的开销，连接的对象等

### 链路状态协议——LSDB

- 路由器将LSA存放在lSDB中
- LSDB汇总了网络中路由器对于自己接口的描述
- LSDB包含全网拓扑的描述

### 链路状态协议——SPF计算

- 每台路由器都会计算出一棵以自己为根的,无环的，拥有最短路径的“树”

# OSPF协议工作原理

### OSPF路由器之间的关系

- OSPF路由器之间的关系分为：邻居关系和邻接关系
- 在双方互联接口上激活OSPF，路由器发送侦听的Hello报文，然后这两台路由器便形成了邻居关系
- DD、LSR、LSU、LSACK等，当两台路由器LSDB同步完成，并开始独立计算路由时，两台路由器便形成了邻接关系
- OSPF完成邻接关系的四个步骤：建立邻居关系、协商主/从（DD报文），交互LSDB信息、同步LSDB

### OSPF协议报文类型

| 报文名称                    | 报文类型                                                     |
| --------------------------- | ------------------------------------------------------------ |
| Hello                       | 周期性发送，用来发现和维护OSPF邻居关系                       |
| Database Description （DD） | 描述本地LSDB的摘要信息，用于两台设备进行数据库同步           |
| Link State Request （LSR）  | 请求所需要的LSA，设备在OSPF邻居双方成功交换DD报文才会向对方发出LSR报文 |
| Link State Update  （LSU）  | 用于向对方发送其所需要的LSA                                  |
| Link State ACK （LSA）      | 用来对收到的LSA进行确认                                      |



### OSPF邻接关系建立流程

![image-20210321193217905](E:%5C%E7%AC%94%E8%AE%B0%5CHCIA-DataCom%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20210321193217905.png)

![image-20210321193251103](E:%5C%E7%AC%94%E8%AE%B0%5CHCIA-DataCom%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20210321193251103.png)

![image-20210321193807387](E:%5C%E7%AC%94%E8%AE%B0%5CHCIA-DataCom%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20210321193807387.png)

# OSPF协议典型配置

### OSPF的有四种网络类型

- Broadcast（广播）
- NBMA（非广播的多点接入）
- P2MP（点到多点）
- P2P （点到点）

### DR与BDR的背景

- OSPF指定三种OSPF路由器身份，DR（指定路由器）、BDR（备份指定路由器）、DRother路由器
- 只允许DR、BDR与其他OSPF路由器建立邻接关系，DRother之间不会建立毗邻的OSPF邻接关系，双方停滞在2-way状态
- BDR会监控DR的状态，并会在DR发生故障时代替其角色

### OSPF多区域

- 将一个OSPF区域划分多个区域，可以使OSPF支撑更大规模组网
- OSPF多区域的设计减小了LSA泛洪的范围，有效的拓扑变化的影响限制在区域内，达到优化网络的目的
- 在区域的边界可以做路由汇总，减小路由表的规模，提高了网络拓展性，有利于组建大规模的网络

### OSPF路由器类型

- 区域内路由器（IR）
- 区域边界路由器（ABR）
- 骨干区域路由器（Area 0）
- 自治系统边界路由器（ASBR）在一个路由器使用了多个协议比如RIP需要路由引入

### OSPF的基础配置命令

- [Huawei]ospf 【procsee-id】  router-id  默认的进程号为1，router-id为手工指定的设备ID；例如：ospf 100 1.1.1.1
- [Huawei-ospf]area area-id
- [Huawei-ospf-1-area0.0.0.0]network network-address wildcard-mask network-address为要宣告的网段，wildcard-mask为反向子网掩码
- [Huawei-GE0/0/1]ospf cost cost 配置OSPF接口开销
- [Huawei-ospf-1]bandwidth-reference value 设置OSPF带宽参考值，缺省是100Mbit/s
- [Huawei-G0/0/1]ospf dr-priority priority 设置接口在选举DR时的优先级，值越大越优先，范围是0~255，默认是1

