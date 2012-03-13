---
layout: post
title: 计算机网络拾遗笔记
tags: 网络
key : cnn20200414
---

虽然工作已近一年（实际8个月），但我对于网络技术的掌握依然十分**浅薄**，工作中遇见问题也常常无从下手。

![Title Photo](https://www.nationalgeographic.com/content/dam/news/2015/11/25/01qajoelsartore.jpg)

我写这篇笔记的目的，不仅仅在于“**写**”这个行为。更多的是为了以期巩固网络技术基础，增强对网络技术原理的理解；同时结合工作遇见的问题，对一些网络技术加以实例化的解释。在完成这篇笔记的过程中，从互联网中攫取了一些前人总结的“精华”，如果没有明确标出出处，请联系我更改。

<!--more-->

# 0. 修订记录

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-14 | 完成初步框架搭建； | JiangDi |
| 2020-04-27 | 补充文章框架； | JiangDi |
| 2020-04-30 | 完善端口聚合部分； | JiangDi |
| 2020-04-30 | 完善VLAN部分； | JiangDi |
| 2020-04-30 | 完善NAT，端口映射，静态域名部分； | JiangDi |
| 2020-08-07 | 完善链路层部分； | JiangDi |
| 2020-08-08 | 完善链路层部分； | JiangDi |
| 2020-08-09 | 完善网络层部分； | JiangDi |
| 2020-08-20 | 完善网络层部分（DHCP/NAT）； | JiangDi |
| 2020-08-23 | 完善UDP和IP分片； | JiangDi |
| 2020-09-01 | 完善TCP部分笔记； | JiangDi |

# 1. 摘要 

# 2. TCP/IP协议栈的实现
网络通信的目的是为了有效的传输数据到对端。为此，我们需要网络提供以下三点：
1. 网络连接如何建立；
2. 传输到对端地址；（端到端（目的地址与源地址））
3. 有效的传输；（数据包的 差错控制【数据传输的准确性】和 流量控制【数据的发送效率】）

![OSI5Layer](https://base.imgix.net/files/base/ebm/electronicdesign/image/2013/10/electronicdesign_com_sites_electronicdesign.com_files_uploads_2013_09_0913_WTD_osi_F2.png?auto=format&fit=max&w=1440)

TCP/IP 协议栈的设计和实现 主要是依据**分层**的思想。

通常可以分为五层模型和七层模型。这篇学习笔记我们主要以五层模型为主。其中对于我来说，最重要的是IP层和传输层。
- 物理层
- 数据链路层
- IP层
- 传输层
- 应用层

## 网络地址
在网络层中，终端之间的访问使用IP地址寻径。IPv4的地址通常由4字节（32位）组成。早期IP地址的划分采取**分类地址**的形式(即网络号+主机号)划分成五类地址，但是当因特网的规模变得越来越庞大时，分类地址会造成A、B类地址大量主机号被浪费，C类地址主机号不够用的情况出现。

因此，由诞生了基于分类寻址的**子网寻址**（网络号+子网号+主机号）以缓解网络地址分配的问题。子网掩码用于确认一台主机的网络号和子网号，一般只用于局域网内的地址分配。

子网掩码 是由一台主机或路由器使用的分配位，用来确定如何从一台主机对应IP地址中获得网络和子网信息。

在每个IPv4网络中，一个特殊地址被保留作为子网广播地址（主机位全部取1）。使用这种地址作为目的地的数据包，也被称作**定向广播**，这种数据包通过Internet路由转发到目的子网的所有主机。

一些特殊用途的地址：

组播地址（D类地址 224.0.0.0 - 239.255.255.255）

任播地址




参考文档：
- [详解：IP的分类、寻址规则及子网掩码zz](https://wdicc.com/about-ip/)
- [TCP/IP协议(1): IP 地址和寻址方式](https://juejin.im/entry/5b7378616fb9a0098c1fe700)
- [IP 寻址和子网划分](https://www.cisco.com/c/zh_cn/support/docs/ip/routing-information-protocol-rip/13788-3.html)
- [一文看懂IP地址：含义、分类、子网划分、查与改、路由器与IP地址](https://network.51cto.com/art/201911/605681.htm#topx)


# 3. 二层 （链路层）

## 以太网（IEEE802.3）

由于以太网是一个异步的局域网，所以以太网帧开头是七个字节的前导符（0xAA），用于确定一个帧的到达时间，第八个字节为SFD（0xAB）分割符。

一个普通的以太网帧基本格式：

目的MAC(48位 6字节) + 源MAC（48位 6字节）+ 长度/类型（2字节） + 数据字段（46 ~ 1500） + 帧校验序列（4字节） = 64~1518字节

其中包含 14字节 头部信息，4字节 CRC，则数据字节可以为 46~1500字节。

以太网帧存在最小帧（64字节） 要求数据区最小为46字节（书上此处位48字节）。若有效载荷较小，填充字节（0）被添加到有效载荷尾部，以达到最小长度。

## VLAN

为了应对二层广播到每台主机的大量网络流量，或者出于安全因素，可能要在不同终端（组）之间禁止通信。于是便提出了虚拟局域网（VLAN）的功能。

## 链路聚合

为了在不增加成本（高速网络接口），提高网络接口的性能和传输带宽，链路聚合提供了一种简单的解决方案。

链路聚合，也叫端口聚合。通过链路聚合，两个或更多接口被视为1个，通过冗余或将数据分割到多个接口，提高性能并获得更好的可靠性。

## 链路层流量控制

一些交换机通过交换机和网卡之间发送特殊信号帧（PAUSE帧）来实现流量控制。

## Wi-Fi（IEEE 802.11）

等待更新

## 点到点协议

PPP 一种支持建立链路的基本方法。

## 地址解析协议（ARP）

终端在IPv4网络中，当要发送一个数据到另外一台终端时，除了需要知道目的主机在网络中IP地址，还需要知道目的主机的硬件地址，以便直接向它发送数据。

因此，**ARP协议**被用来提供IP地址与MAC地址（硬件地址）之间的映射。

ARP正常工作时，`发送终端`通常向当前链路层网络中的`所有终端`发送一个“请求（广播）报文”，询问`目的终端（IP）`对应的ARP地址是什么？若链路层网络中存在`目的终端（IP）`，则`目的终端`会回复一条“应答报文”给`发送终端`，应答报文包含了IP地址与MAC的映射关系。当`发送终端`接受到“应答报文”后，便完成一次ARP协议流程。

### ARP 缓存

ARP高效运行的关键在于每个主机/路由器上的ARP缓存表。该缓存存储了每个接口IP地址到硬件MAC地址的最新映射。同时每对映射关系存在定时器对过期或是不完整的ARP缓存进行清理。

### ARP 报文格式

在以太网帧中，对应的ARP（请求/应答）类型字段为0x0806。

WireShark报文展示:

### 代理ARP

代理ARP通常是设置一台主机进行ARP应答，它使得ARP请求主机认为做出响应的系统就是目的主机，但实际上目的主机可能在其他地方甚至不存在。

Linux支持一种**自动代理**的功能，这样做允许自动代理一个地址范围，而不是单个地址。

### 免费ARP（上行ARP）

通常发生在终端刚刚接入网络时：

1. 更新ARP缓存表
2. 检测链路层网络中是否存在相同ip地址的设备。

# 4. 三层 （网络层）

## IP协议
IP协议具有**尽最大努力交付**和**无连接**的特点。
- **尽最大努力交付**的意思是不保证IP数据报能成功到达目的地。
- **无连接**的意思是每个数据包均是独立的。也就意味着IP数据包不按顺序交付。

### IP 数据包
ipv4数据包头部如下图所示：

![ipv4](https://notes.shichao.io/tcpv1/figure_5-1.png)

通常，一个ipv4头部包含20个字节。

IP数据包版本号 0.5字节（4 Bit）+ 头部长度 0.5字节（4 Bit） 【代表N个32位字长】 + 服务字段{ 区分服务字段 DS (6 Bit) + 显示拥塞通知 ECN（2 Bit） }   + IP数据包总长度 2字节（16 Bit） 【代表N个字节 最大位65535】
> 通过头部长度和IP数据包总长度，可以识别一个数据包的数据部分从何处开始。

标识 2字节（16 Bit） + 标志（ 3 Bit） + 分片偏移（13 Bit）
>标识：在IP分片中，用来标识分片。

生存期TTL 1字节（8 Bit） + 协议（8 Bit） + 头部校验和 2字节（16 Bit）
>生存期TTL 表示一个IP包可以经过的路由器数量的上限，若该值变为0，则该数据包被丢弃，并发送一个ICMP消息通知发送方。
>协议：用来表示IP包数据部分的数据类型。常用的是17（UDP）和6(TCP)。
>头部校验和：仅计算IPv4头部校验(TTL经过路由器减1也意味着校验和值也会改变)。这意味着IP协议不检查IP包的数据部分的正确性。

源IP地址 4字节（32 Bit） + 目的IP地址 4字节（32 Bit）

网络字节序：
>在网络传输中，1个32位值按以下顺序传输，首先是0~7位，8~15位，16~23位，最后是24~21位，即高位优先字节序。这也被称作网络字节序（大端）
。计算机的CPU通常采用小端字节序，在传输时则需要将头部值转换为网络字节序，并在接受时转换回来。

### IP转发
IP转发的意思是，当发送终端发送1个IP数据包，若目的主机与发送主机相连，则可以直接发送IP数据包到目的地主机。反之，则将IP数据报发送给路由器，由路由器负责转发到目的主机。

* 转发表
  * 目的地址
  * 掩码
  * 下一跳
  * 接口

- 直接交付：直接交付不需要路由器，IP数据报封装在一个链路层帧中，它可以直接转发到目的主机；
- 间接转发：间接交付需要路由器，并把数据报中目的主机的链路层MAC地址换为路由器的MAC地址；


## DHCP协议

> 通常，一台能够上网的终端，需要有IP地址和子网掩码，DNS服务器地址和路由器（网关）的地址这四个信息。为一台终端配置这些信息，可以通过手工配置或者自动获取的方式（DHCP协议）

DHCP协议的几个重要概念：
- 地址池：待分配的一段连续的IP地址范围；
- 租用：分配出去的IP地址只在一段时间有效；


### DHCP 消息格式

![DHCP消息格式](https://img2018.cnblogs.com/blog/702891/201901/702891-20190113165246254-1385949051.png)

部分字段解释：
- OP字段：请求（1） 应答（2）
- HW类型：通常为1（以太网）
- HW长度：通常为6
- 跳步数：消息的中继次数
- 事务ID：随机数
- 客户机IP地址：请求终端的IP地址
- 你的IP地址：待分配的IP地址
- 服务器IP地址：DHCP服务器地址
- 网关地址：

### DHCP协议操作
在一次DHCP交换中，包含如下四个步骤：

Discover - Offer - Request - Ack

终端初次接入网络后，首先广播一个Discover报文，接收到Discover报文请求的DHCP服务器，会响应一个Offer报文给请求终端，Offer报文包含“你的IP地址”中的待分配IP地址，以及租期。

终端收到Offer报文后，发送一个Request的报文请求分配，收到服务器的ACK报文则完成一次DHCP地址申请。

注：收到ACK后终端会发送一次ARP请求，检测ACD，若该地址已被使用，终端则不使用该地址，并向DHCP服务器发送一条HDCPDECLNE报文。

由于终端会记住之前网络的地址，所以有时候抓包会看见先发一个Request的报文 - Noack

Request - Noack - Discover - Offer - Request - Ack


```
#windows 
ipconfig /release
ipconfig /renew
ipconfig /all

#Linux
dhclient -r
dhclient
```

## 防火墙 与 NAT

防火墙的出现主要是为了避免**网络攻击**和**TCP/IP协议栈中未定义的协议操作造成的网络问题**。
- 包过滤防火墙：丢弃特定条件的数据包
- 代理防火墙：多宿主的服务器主机，通常不会再IP协议层中路由。

NAT的提出是为了解决日益枯竭的IPv4地址资源和全局路由可拓展的问题。NAT的工作原理是重写往一个方向传输的数据包源IP地址(SNAT)，或者重写往另一个方向的数据包目的地址(DNAT)。此外，NAT还具备防火墙的功能。因为默认情况下，外网的主机是无法直接访问私网的终端系统，也无法得知内网终端的部署范围和数量等。

简述一次正常NAT的建立：
在TCP第一次握手发出后，第一次握手的数据包源IP地址+端口号会被NAT服务修改为出口IP地址+端口号。同时，NAT创建一条记录处理这个新的连接（设定计时器 Connection timer）。
当第二次握手数据包到来时，NAT服务依据记录修改目的IP地址+端口号。如此，便能完成正常的NAT建立。

端口映射：从外放访问内网主机（服务）；


## ICMP协议
待更新
## 组播协议
待更新

# 5. 四层 （传输层）

## UDP

UDP数据包

![UDP 数据包](https://notes.shichao.io/tcpv1/figure_10-1.png)

在IPv4协议中，17用来表示udp。

UDP首部

![UDP 首部](https://images2015.cnblogs.com/blog/816350/201510/816350-20151024000156833-1505604161.png)

- UDP的长度字段是UDP头部+UDP数据的总和；
- UDP校验和覆盖UDP头部，UDP数据和一个伪头部；

注：由于在IP头部中已经决定了传输协议（UDP 17），所以TCP端口号只能被TCP使用，UDP端口号只能被UDP使用。

UDP伪头部

![UDP 伪头部](https://img2018.cnblogs.com/blog/1300168/202002/1300168-20200229124211763-819525016.png)

## IP分片

由于链路层MTU通常为1500，IP层接收的数据往往都大于这个值，为了保证发送成功，于是引入了IP分片。当IP数据包被分片了，只有到了接收端才会重组。**当一个数据包被分片后，每个IPv4头部中的总长度字段要被修改成该分片的总长度。** 标识字段的值被复制到每个分片，同时当分片到达目的地时利用它来组成分组。

分片的具体报文呈现可以发现，有更大偏移量的分片比第一个分片优先传递；因为最后一个分片传递过去，接收主机就可以确定所需的缓存空间的最大值，以重组整个数据包。

**UDP传输 IP分片示例**

1500字节的MTU，除去20字节的IP头部，8字节的UDP头部，若数据大于1472，则需要IP分片。


## TCP协议
TCP虽然和UDP一样均使用网络层，但是TCP提供了一种与UDP截然不同的服务。

TCP在IP包的封装如下所示：

![ippacket](https://user-images.githubusercontent.com/7577575/78691226-f8eb9f00-792a-11ea-86ad-e6953b52e05f.png)

TCP头部如下所示：

![TCPHeader](https://i.loli.net/2019/06/21/5d0c331c5378f22175.jpg)

默认不带选项的TCP头部通常是20字节，

### TCP 连接建立

# 6. 网络协议

## DNS协议

## 6.1 Net-SNMP协议

# 7. WLAN

## 7.1 WLAN技术简述

## 7.2 WLAN组网理论

## 7.3 WLAN组网实验

# 8. 开源网络协议栈

## 8.1 FD.IO VPP

# 9. 网络相关面试题

# 10. 网络实战

## 10.1 VLAN
>A virtual LAN (VLAN) is any broadcast domain that is partitioned and isolated in a computer network at the data link layer (OSI layer 2).--[WikiPedia](https://en.wikipedia.org/wiki/Virtual_LAN)

### 10.1.1 VLAN接口类型
VLAN根据接口的不同，会给数据帧进行不同的操作。
- Access  
  ![access](https://forum.huawei.com/enterprise/zh/data/attachment/forum/dm/ecommunity/uploads/2014/0623/18/53a80752c886a.png)
- Trunk  
  ![Trunk](https://forum.huawei.com/enterprise/zh/data/attachment/forum/dm/ecommunity/uploads/2014/0623/18/53a807dd56be6.png)
- Hybird  
  ![Hybird](https://forum.huawei.com/enterprise/zh/data/attachment/forum/dm/ecommunity/uploads/2014/0623/19/53a80929ce751.png)

### 10.1.2 VLAN内的通信
### 10.1.3 不同VLAN间的通信
1. VLANIF
2. SUPER-VLAN(VLAN聚合)

### 10.1.x 相关资料出处
1. [VLAN-Wikipedia](https://en.wikipedia.org/wiki/Virtual_LAN)
2. [VLAN简介](https://support.huawei.com/enterprise/zh/doc/EDOC1000141407/89d257c5)
3. [SuperVlan-VLAN聚合](https://support.huawei.com/enterprise/zh/doc/EDOC1000141407/e4ce1bc4)
4. [VLAN技术连载——VLAN基础](https://forum.huawei.com/enterprise/zh/thread-246713.html)
5. [VLAN技术连载——VLAN划分](https://forum.huawei.com/enterprise/zh/thread-278705.html)
6. [VLAN技术连载——VLAN通信](https://forum.huawei.com/enterprise/zh/thread-282003.html)
7. [VLAN技术连载——VLAN隔离](https://forum.huawei.com/enterprise/zh/thread-290173.html)
   
## 10.2 端口聚合
>In computer networking, the term link aggregation applies to various methods of combining (aggregating) multiple network connections in parallel in order to increase throughput beyond what a single connection could sustain, and to provide redundancy in case one of the links should fail. A link aggregation group (LAG) combines a number of physical ports together to make a single high-bandwidth data path, so as to implement the traffic load sharing among the member ports in the group and to enhance the connection reliability. ---[Wikipedia](https://en.wikipedia.org/wiki/Link_aggregation)

端口聚合有如下三个优势：
- 增加带宽  
  链路聚合接口的最大带宽为各子链路带宽之和
- 提高可靠性  
  如果某条链路存在故障，流量可以切换到其他可用的链路上去，从而提高端口聚合的可靠性
- 负载分担  
  在一个端口聚合组里，可以依据协议将流量分担到不同的链路

>定义：以太网链路聚合简称**链路聚合**，它通过将多条以太网物理链路捆绑在一起成为一条逻辑链路，从而实现增加链路带宽的目的。同时，这些捆绑在一起的链路通过相互间的动态备份，可以有效地提高链路的可靠性。-[以太网链路聚合](https://support.huawei.com/enterprise/zh/doc/EDOC1000141407/c1b2412c)


### 10.2.1 端口聚合模式
根据是否启用链路聚合控制协议LACP，链路聚合分为手工模式和LACP模式。
1. 静态端口聚合 （手工模式）  
   手工模式可以实现增加带宽、提高可靠性和负载分担的目的。
2. 动态端口聚合（LACP模式）  
   在手工模式的基础上。动态端口聚合提高了端口聚合的容错性，支持端口备份功能。

### 10.2.2 华为S5700 配置端口聚合命令
简述组网拓扑（有空画个图）：

网关侧将eth0 eth1 eth2 聚合成一个bond。交换机侧根据网关测设定的聚合模式，也需要配置成相对的聚合模式。配置好后，网关和网关下接能正常上网。
1. 静态聚合  
   ```
   sys
   undo info-center enable  #关闭配置变更提示信息 
   interface Eth-Trunk 1 #创建端口聚合口
   trunkport GigabitEthernet 0/0/1 to 0/0/3 # 将0-3口加入创建好的Eth-Trunk
   display interface Eth-Trunk  # 显示聚合口的信息
   ```
2. 动态聚合  
   ```
   sys
   interface Eth-Trunk 1 
   bpdu enable
   mode lacp-static
   trunkport GigabitEthernet 0/0/1 to 0/0/3
   ```


### 10.2.3 端口聚合 Q&A
- BPDU 是什么 ？  
  是一种生成树协议问候数据包，用来在网络的网桥间进行信息交换。
  >[BPDU保护](https://support.huawei.com/enterprise/zh/doc/EDOC1100090440)


## 10.3 静态域名
1. `/etc/resolv.conf`解释
   specifies the nameservers for resolver lookups, where it will actual use the DNS protocol for resolving the hostnames.
2. `/etc/hosts`解释
   overrides all nameservers by mapping url's shortnames to IPs.


## 10.4 端口映射
Linux 端口映射使用iptables规则实现：
```s
# 访问本机80端口被映射到本机777端口
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 777
# 访问本机80端口被映射到172.16.1.3的777端口
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.16.1.3:777
```
注意：在使用iptables规则前，确认linux支持IP转发。
```
sysctl -w net.ipv4.ip_forward=1
```

## 10.5 NAT
>Network address translation (NAT) is a method of remapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device.---[Wikepedia](https://en.wikipedia.org/wiki/Network_address_translation)

### 10.5.1 NAT in Linux
> In Linux, Network Address Translation generally involves "re-writing the source and/or destination addresses of IP packets as they pass through a router or firewall"---[Wikepedia](https://en.wikipedia.org/wiki/Network_address_translation)

Here goes some examples.
```s
#DNAT
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.31.0.23:80
#SNAT
iptables -t nat -A PREROUTING -s 192.168.0.0/16 -i eth0 -j SNAT --to-source 123.123.123.123
#MASQUERADE 
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 
```

### 10.5.X 引用
1. [FORWARD AND NAT RULES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/4/html/security_guide/s1-firewall-ipt-fwd)
2. [Nat Tutorial](https://www.karlrupp.net/en/computer/nat_tutorial)

## 10.6 VXLAN 

### 10.6.X 引用
1. [S5720, S6720 V200R011C10 配置指南-VXLAN](https://support.huawei.com/enterprise/zh/doc/EDOC1000178185/b6215386)
2. [服务器虚拟化](https://support.huawei.com/enterprise/zh/doc/EDOC1000178185?section=j00w)
3. [华为悦读汇 技术发烧友：认识VXLAN](https://forum.huawei.com/enterprise/zh/thread-334207.html)


# 11 相关课程
## 2019年秋网络程序设计
[学堂在线课程视频](https://next.xuetangx.com/course/USTC08091001656/1510805?fromArray=learn_title)
### 互联网概述
课程资料：[PPT](https://github.com/mengning/net/raw/master/lab1/%E4%BA%92%E8%81%94%E7%BD%91%E6%A6%82%E8%BF%B0.pptx)

由两台电脑的互通简述TCP/IP模型，说明为了电脑之间的互通，每一层的作用。
如果是多台电脑通信，那么又需要如何实现？ 交换机的工作原理？
### Socket网络编程
#### 编译、构建与调试
##### gcc
```shell
# 编译程序
gcc hello.c
#预处理 使用-E参数
gcc -E -o hello.cpp hello.c
# 汇编
gcc -x cpp-output -S -o hello.s hello.cpp
gcc -S -o hello.s1 hello.c
# 翻译机器代码
gcc -x assembler -c hello.s -o hello.o
gcc -c hello.c -o hello.o
as -o hello.o hello.s
# 链接成可执行文件
gcc -o hello hello.c
gcc -o hello hello.o
```
##### makefile 
[Makefile Fast快速笔记](https://jiangdiii.github.io/2020/04/14/MakefileNote.html)
##### GDB
[GDB Fast快速笔记](https://jiangdiii.github.io/2020/06/12/gdbNote.html)
#### socket编程
PF_INET 与AF_INET 的区别
1. TCP编程示例
2. UDP编程示例

### TCP协议与Linux内核
#### TCP协议概述
1. 三次握手
2. 四次握手

[Linux内核源码下载](https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.0.1.tar.gz)
```s
xz -d linux-5.0.1.tar.xz
tar -xvf linux-5.0.1.tar
```
### IP协议及路由表
#### IP协议基础
#### 路由表
默认网关
Next-hop method 方式
#### 路由协议简介

### ARP协议及ARP缓存
#### ARP协议简介
#### ARP解析过程
### 二层交换网络及转发过滤数据库
1. 交换机
2. 集线器

### 参考资料
- [2019年秋网络程序设计](https://github.com/mengning/net/blob/master/np2019.md)


# 0. 引用
1. [Computer Networking. A Top-Down Approach. Seventh Edition. James F. Kurose.](https://leonawang.com/books/Computer%20Networking%20A%20Top-Down%20Approach%207th%20edition.pdf)
2. [The Fundamentals of Networking  IBM](https://www.ibm.com/cloud/learn/networking-a-complete-guide)
