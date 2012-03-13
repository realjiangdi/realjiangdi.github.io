---
layout: post
title: Linux 常用命令总结
tags: 效率
---
Some userful Linux commands here!

<!--more-->

# 目的 
Linux 常用命令快速查询
# Network
## 网口配置/查看 ifconfig
```shell
ifconfig -a 
ifconfig eth0

ifconfig eth0 up
ifconfig etho down

ifconfig eth0 69.77.77.77
ifconfig eth0 netmask 255.255.255.0
ifconfig eth0 broadcast 172.16.25.95
ifconfig eth0 69.77.77.77 netmask 255.255.255.0 broadcast 192.168.2.4
```
some other useful examples

## 命令 ping
```shell
# 指定ping包次数
ping -c 10 192.168.1.1
# 指定ping包次数与时间间隔
ping -c 10 -i 0.5 192.168.1.1
# 指定网卡接口
ping -I eth0 192.168.1.1
```
## 命令 ip 
```shell
# 显示网络设别运行状态
ip link list
# 显示更加详细的网络设备运行状态
ip -s link list
# 一些类似于ifconfig的实例运用
ip link set eth0 up
ip link set eth0 down
ip link set eth0 mtu 1500
ip addr add 192.168.0.1/24 dev eth0
ip addr del 192.168.0.1/24 dev eth0
ip addr show eth0
```
## 网络侦测 mtr
```shell
mtr -r 180.101.49.12
Start: Fri Jun 12 11:26:25 2020
HOST: jiangdi                     Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- gateway                    0.0%    10    1.3   1.4   1.1   1.7   0.0
  2.|-- 180.168.191.193            0.0%    10    3.2   2.4   1.7   3.5   0.5
  3.|-- 124.74.110.245             0.0%    10   13.5  12.1   3.8  31.3   8.6
  4.|-- 61.152.0.149              10.0%    10   16.1  11.5   4.4  25.7   7.4
  5.|-- 101.95.89.46              70.0%    10   42.8  20.5   5.6  42.8  19.6
  6.|-- 101.95.88.205             10.0%    10   16.1  19.4   5.9  69.0  19.7
  7.|-- 59.43.80.98                0.0%    10    7.8  15.8   6.9  66.3  18.3
  8.|-- 202.97.92.2               10.0%    10   15.9  21.3  11.2  60.0  16.2
  9.|-- 58.213.94.78              20.0%    10   15.6  15.4   9.6  42.2  11.0
 10.|-- 58.213.94.134             80.0%    10   11.7  12.3  11.7  13.0   0.0
 11.|-- 58.213.96.114              0.0%    10   15.6  16.0  10.4  37.7   8.2
 12.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 13.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 14.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 15.|-- ???                       100.0    10    0.0   0.0   0.0   0.0   0.0
 16.|-- 180.101.49.12              0.0%    10   20.3  13.3   8.0  21.2   4.4
```
参数解释
- Loss : 丢包率
- Snt : 每秒发包数量
- Last : 最后一个包的延时
- Avg : 平均延时
- Best : 最低时延
- Wrst : 最差时延
- StDev : 稳定性
## 网络路径 traceroute
```
traceroute xiaomi.com
 1  gateway (10.60.1.254)  1.257 ms  0.876 ms  0.637 ms
 2  180.168.191.193 (180.168.191.193)  1.933 ms  2.493 ms  2.292 ms
 3  124.74.110.245 (124.74.110.245)  16.428 ms  16.205 ms  16.015 ms
 4  124.74.215.21 (124.74.215.21)  15.757 ms 61.152.0.149 (61.152.0.149)  15.510 ms 61.152.49.181 (61.152.49.181)  15.171 ms
 5  101.95.89.62 (101.95.89.62)  14.956 ms 101.95.89.42 (101.95.89.42)  14.703 ms 101.95.89.38 (101.95.89.38)  14.368 ms
 6  101.95.88.33 (101.95.88.33)  10.612 ms 101.95.88.45 (101.95.88.45)  14.280 ms 61.152.24.197 (61.152.24.197)  13.667 ms
 7  59.43.80.102 (59.43.80.102)  13.504 ms 59.43.80.86 (59.43.80.86)  13.267 ms 59.43.80.90 (59.43.80.90)  13.020 ms
 8  202.97.56.165 (202.97.56.165)  29.668 ms 202.97.56.145 (202.97.56.145)  28.579 ms 202.97.56.33 (202.97.56.33)  34.163 ms
 9  218.30.25.38 (218.30.25.38)  33.329 ms 180.149.159.14 (180.149.159.14)  32.390 ms 180.149.159.6 (180.149.159.6)  32.742 ms
10  36.110.246.201 (36.110.246.201)  33.088 ms 36.110.246.197 (36.110.246.197)  32.886 ms *
11  36.110.244.106 (36.110.244.106)  38.478 ms * *
12  * 106.120.186.62 (106.120.186.62)  29.529 ms *
13  220.181.10.205 (220.181.10.205)  29.912 ms 220.181.10.214 (220.181.10.214)  29.770 ms  29.639 ms
14  220.181.10.166 (220.181.10.166)  31.327 ms 220.181.10.150 (220.181.10.150)  30.367 ms 220.181.10.166 (220.181.10.166)  31.191 ms
15  * * *
16  124.251.99.210 (124.251.99.210)  40.881 ms 124.251.99.218 (124.251.99.218)  43.154 ms 124.251.99.210 (124.251.99.210)  39.529 ms
17  124.251.99.198 (124.251.99.198)  39.760 ms 124.251.63.213 (124.251.63.213)  39.514 ms 124.251.99.198 (124.251.99.198)  40.127 ms
18  124.251.99.198 (124.251.99.198)  39.857 ms *  41.026 ms
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  * * *
26  * * *
27  * * *
28  * * *
29  * * *
30  * * *
```

## http客户端 curl/wget
模拟一些get或者post的请求
```s
curl www.baidu.com
weget wwww.baidu.com
```

## 桥 Bridge
```s
#create a new bridge and change its state to up 
ip link add name $bridge_name type bridge
ip link set $bridge_name up
#add an interfcae into the bridge, its state must up
ip link set eth0 up
ip link set eth0 master $bridge_name
#show the existing bridges and associated interfaces
bridge link
#remove an interface from a bridge 
ip link set eth0 nomaster
#delete a bridge issue 
ip link delete $bridge_name type  bridge
#assign an ip address 
ip addr add dev bridge_name 172.16.17.1

#manage a network bridge using the legacy brctl tool.
#create a new bridge:
brctl addbr btidge_name
#dele a bridge
brctl delbr bridge_name
#add a device to a bridge fow exaple eth0
brctl addif bridge_name eth0
#show current bridges and what interfaces they are connected to:
brctl show

ip link set dev bridge_name up
```
## 路由 route
```shell
# show route
route -n 
# add a route
route add -net 192.168.2.0 netmask 255.255.255.0 dev eth0
# del a route
route del -net 192.168.2.0 netmask 255.255.255.0  dev eth0 
# add a default gateway
route add default gw 192.168.1.10 
# del a defaule gateway 
route del default gw 192.168.1.10
# block a host run
route add -host 192.168.1.19 reject
```
## 端口查看 netstat
```shell
# list all ports 
netstat -a | more
# list all tcp ports' connections
netstat -at
# list all udp ports' connections
netstat -au
# 列出TCP和UDP端口
netstat -at | -au
# 显示进程id和名称
netstat -p
# 显示路由信息
netstat -r
# 显示所有监听端口
netstat -tnl
# list all unix socket connections
netstat -ax
# list only listening ports 
netstat -l
# list only listening tcp ports
netstat -lt
# list only listening udp ports
netstat -lu
# show service name with pid
netstat -tp
# show port number used by pid
netstat -anlp | grep $PID
# show which process using a particular port
netstat -anly | grep $portnumber
# show kernel routing information
netstat -r
# show the list of networks interfaces
netstat -i
#example
netstat -atep | grep ssh
netstat -atnep | grep 443
```
## 防火墙 iptables
```shell
# show iptables rules
iptables -L
iptables -L INPUT
iptables -t filter -L
iptables -t nat -L
# add rules
iptables -t filter -A INPUT -s 192.168.146 -j DROP//递增
iptables -t filter -I INPUT -s 192.168.146 -j DROP//插入
iptables -t 表 -A 链名 匹配条件 -j 动作
iptables -t 表 -I 链名 匹配条件 -j 动作
# dele rules
iptables -t filter -D INPUT -s 192.168.146 -j DROP//删除命令
iptables -t filter -D INPUT 3//删除指定行命令
# clear rules
iptables -F
# example
#block SSH connections from any IP address
iptables -A INPUT -p tcp --dport ssh -j DROP
#block SSH connections from 10.10.10.10.
iptables -A INPUT -p tcp --dport ssh -s 10.10.10.10 -j DROP
#block SSH connections from 59.45.175.0
iptables -A INPUT -p tcp -m tcp --dport 22 -s 59.45.175.0/24 -j DROP
#屏蔽单个IP的命令
iptables -I INPUT -s 123.45.6.7 -j DROP
#重定向80端口打上0x55标签的报文到10.1.0.1:2060
iptables -t nat -A PREROUTING -p tcp --dport 80 -m mark --mark 0x55 -j DNAT --to-destination 10.1.0.1:2060
```
## 抓包工具 tcpdump
```shell
# interface
tcpdump -i br0 -w br0_packet.cap
tcpdump -i br0 -c 5000 -w br0_packet.cap
# ip
tcpdump host 1.1.1.1
tcpdump src 1.1.1.1
tcpdump dst 1.1.1.1
# ip port
tcpdump port 22
tcpdump src port 22
# mac 针对mac抓包
tcpdump ether 6c:11:61:a1:11:41
tcpdump ether src 1c:11:18:1e:1b:54
tcpdump -i br0 -w test.cap ether src 6c:11:61:a1:11:41 or ether dst 6c:11:61:a1:11:41
# or and not
tcpdump src 192.168.1.100 or dst 192.168.1.50 && port 22
tcpdump port 443 or 80
# read tcpdump packets
tcpdump -r br0_packet.cap
# more detail about tcpdump
man tcpdump
```
# Tool
## 文件传输 tftp 
```shell
#获取Windows的文件
tftp -r $filename -g $DstIp
#Linux上传文s件
tftp -l $filename -g $DstIP
```
## 文件操作进程 lsof
```shell
# show a list of all open files and the processes opened them.
lsof | less
#list of open files
lsof [file_name]
#list all open file
lsof
#list files opened by specified processes
lsof -u nginx
#list files opened by specified to a process
lsof -p [PID]
lsof -p 1100
```
## 压缩解压 tar
压缩:
```shell
#将所有.jpg后缀的文件打包成all.tar的包
tar -cf all.tar *.jpg
#将所有.gif后缀的文件增加到all.tar包里去
tar -rf all.tar *.gif
#将压缩包里logo.gif的文件更新
tar -uf all.tar logo.gif
#将所有.jpg后缀的文件打包成.tar.gz
tar –czf jpg.tar.gz *.jpg 
```
解压：
```shell
#解压出all.tar包中的所有文件
tar -xf all.tar
#解压all.tar包中的所有文件，并打印输出解压结果
tar -xvf all.tar
#解压.tar.gz
tar -xzvf all.tar.gz
#夹压.tar.bz2
tar -xjvf all.tar.bz2
```
查看压缩包：
```shell
tar -tf aaa.tar.gz
```
[参考资料](https://www.cnblogs.com/jyaray/archive/2011/04/30/2033362.html)
# Stream
## 文本分析 awk
```shell
# 打印输出每行的第一和第四个字段
awk '{print $1 ,$4}' test.txt
# 指定分隔符为=
awk -F= 'print $1'
#实际应用
nvramcli show | grep ACL | awk -F= '{print $1}' | xargs  -t  -L 1 nvramcli unset
```
## 字符匹配 grep
```shell
# 精准匹配
grep -w "pattern"
# 匹配多个字符串
grep -E "pattern1|pattern2|pattern3"
# 打印输出上下10行信息
grep -A 10 file
```
## 字符计算 wc
```shell
# 统计行数
wc -l
```
## 输入转命令参数 xargs
```shell
echo "abc def ghi" | xargs echo 

echo "abc def ghi" | xargs  -t touch 
# -t 打印出最终执行命令， 然后直接执行
echo "abc def ghi" | xargs -p touch
# -p 打印出最终执行命令， 然后询问是否执行
echo "abc def ghi" | xargs -L 1 touch
# -L 指定执行一次命令输入1行参数
echo "abc def ghi" | xargs -n 1 touch
# -n 指定执行一次命令执行一个参数 
```
# Service
## 视频下载工具 youtube-dl
```shell
#List all format of this video
youtube-dl -F  "URL"
#Download playlist of Youtube
youtybe-dl --yes-playlist "URL"
#Specifed the best format of playlist to download
youtube-dl -f best --yes-playlist "URL"

```
## 文件共享 smb
- 创建用户；
- 开通SUDO权限；
- 创建SMB用户；
```
sudo useradd chiang
sudo passwd chiang
sudo visudo
sudo smbpasswd -a chiang
```
## HTTP服务器 nginx
```shell
nginx -s reload
nginx -s stop
# kill nginx process
ps -ef | grep nginx | cut -c 9-15 | sudo xargs kill -9
```
## 服务管理命令 service
```shell
service xxx start
service xxx status 
service xxx restart
service xxx stop
```
## 服务管理命令 systemctl

```shell
systemctl start nginx.service
systemctl stop nginx.service
systemctl restart nginx.service
systemctl status nginx.service
systemctl kill nginx.service
```
## CLI下载工具 aria2
```s
aria2c -s 5 -x 2 ''
```
# Disk and Memory

# System

# Safe
## 网页证书 openssl
CA 根证书生成
```shell
# Generate CA private key 
openssl genrsa -out ca.key 2048 
# Generate CSR 
openssl req -new -key ca.key -out ca.csr
# Generate Self Signed certificate（CA 根证书）
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt

mkdir demoCA
mkdir demoCA/newcerts
touch demoCA/index.txt
echo 01 > demoCA/serial
touch demoCA/index.txt.attr

# private key
openssl genrsa -des3 -out server.key 1024 
openssl genrsa  -out server.key 1024 
# generate csr
openssl req -new -key server.key -out server.csr
# generate certificate
openssl ca -in server.csr -out server.crt -cert ca.crt -keyfile ca.key -days 36500
```

# Other

# 他山之石-可以攻玉
1. [Linux bash总结(一) 基础部分(适合初学者学习和非初学者参考)](https://www.cnblogs.com/skywang12345/archive/2013/05/30/3106570.html)
2. [Linux bash总结(二) 高级部分(适合初学者学习和非初学者参考)](https://www.cnblogs.com/skywang12345/archive/2013/05/31/3107871.html)
3. [编辑器之神VIM 总结(一) 基础部分](https://www.cnblogs.com/skywang12345/p/3144118.html)
# 参考