# Linux-Home-Gateway
这是一个基于Linux系统打造的综合家庭网关，也可以称之为软路由，目的在于取代传统的路由器甚至光猫，并且承担NAS文件存储，科学上网梯子等一系列附加应用。

因日常工作就是与路由器交换机以及Linux等企业级产品打交道，故有了改造家庭网络的念头。（注：此处的路由器与家里常见上网用的“无线路由器”几乎不划等号，掌握一定的Linux和网络基础对快速理解本文内容有很大帮助。）

得益于x86架构相比传统路由器平台更强大的性能，以及Linux系统近乎无限的扩展性，结合我自身的需求，最终硬件使用1台HPE MICROSERVER GEN10 PLUS服务器，加一块PCIE千兆I350光口网卡搭配PON STICK光猫模块，系统使用CentOS 8最新发行版。

# 我的需求，或者说要实现的功能
设备本身至少带4个电口做桥接

设备至少带一个PCIE扩展槽用来插光口网卡

PPPoE拨号获取公网IP

内网NAT让所有设备上网

内网设备获取IPv6全球地址

通过组播观看IPTV

科学上网分流国内外流量

动态DNS绑定公网IP

OpenVPN实现从外网访问到家庭内网

SMB文件共享

PT下载挂机

（想到再补充）

# 软路由系统选型
大多数常见的软路由方案都是在宿主机系统上通过虚拟机安装软路由系统，但是经过我的测试，或多或少都有些问题，无法完全满足我的需求。再加上我稍微有点强迫症倾向，对于虚拟机里运行软路由这事总有种影响性能的担忧（实际上家庭宽带这点负载根本不会有肉眼可见的影响），所以最终决定抛弃虚拟化方案，直接装Linux来实现所有的功能。Linux本身的网络性能是非常够用的，现网中如Arista的交换机就是基于Linux系统开发而成。以下罗列一些我测试过的系统以及他们存在的一些问题。

RouterOS，IPTV看不了。ROS默认使用IGMPv3，联通局端使用IGMPv2，按照协议的设计当客户端发出IGMPv3请求后，服务端应该回应一个报文告知客户端改用IGMPv2，客户端再以IGMPv2重新发起请求。而联通那边似乎做了某种策略直接丢掉了IGMPv3报文，导致双方无法协商正常使用IGMPv2。而ROS内置的igmp-proxy无法修改强制使用IGMPv2。

OpenWRT，内置的igmp-proxy会在进程启动的时候将已经配置好的上下游接口搞错，并且OpenWRT的静态路由下一跳无法写接口只能写IP，家用宽带PPPoE每次拨号获取的IP都不固定，将来做一些路由策略的时候可能会有麻烦。

psSense，主打防火墙功能，优点是极其稳定，公司办公网NAT出口用的就是这货，已经连续稳定运行1500多天未关过机。缺点是体积过于臃肿，基于FreeBSD有些需要用到的软件需要自己编译。
