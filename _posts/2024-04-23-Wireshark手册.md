---
layout: post
title: Wireshark手册
subtitle: 
cover-img: 
thumbnail-img: /assets/img/Wireshark手册/Snipaste_2024-04-23_18-31-22.png
share-img: 
tags: [Manual]
comments: true
author: Jamie
---

# Wireshark工具操作

## 基础知识

{: .box-success}
tcpdump -i any tcp and port 8080 -s0 -c 1000 -w 33333.cap -vvv

### 英语

- hierarchy 分层
- protocol 协议
- gratuitous 无偿的
- identification 标识
- neighbor solicitation 邻居请求
- Classless Inter Domain Router (CIDR) 无类型域间选路

### 网络层协议

#### IP

- IPv4 IPv6
- 使得网络间可以互相沟通
- ipv4根据子网掩码决定 网络地址 和 网络主机

#### ARP-IPv4

- ARP (Address Resolution Protocol) 地址解析协议
- 获取IP和MAC的对应关系
- 包含请求和应答类型

#### ICMPv6

- 单播，任播，多播
  - 单播：在单一网络中的一个设备和另一个设备的通信
- ipv6的地址接口标识符部分是通过MAC地址演变得到
- 邻居请求

### 互联网控制消息协议

- ICMP（Internet Control Message Protocol）

#### TCP

- Transmission Control Protocol
- 三次握手 发起连接
- 四次挥手 断开连接

#### UDP

- User Datagram Protocol

### 应用层协议

- DHCP（Dynamic Host Configuration Protocol）
- DNS
- HTTP
- SMTP

## 捕获

- Capture -> Input 条件过滤
  - src host 192.168.201.97
  - host testserver2
  - ether host 00-1a-a0-52-e2-a0
  - src host 192.168.0.10 && port 80
  - dst host 172.16.16.149
  - port 8080
  - !port 8080
  - dst port 80
  - icmp
  - !ip6

## 过滤

- ctrl + F 按照不同的要求进行搜索
- ctrl + M 标记包
- ctrl + shift + N 下一个标记包
- ctrl + shift + B 上一个标记包
- ctrl + T 重新标记开始时间
- ctrl + shift + T 手动偏移时间
  
## Tcpdump

- tcpdump -h 检查命令
- tcpdump -i eth0 –w packets.pcap 从指定网卡获取流量
- tcpdump –r packets.pcap 读取文件
- tcpdump –r packets.pcap –c10 获取前10个包
- tcpdump -vvv 格式化，v设置打印的层数
- tcpdump -nn 禁用ip地址别名解析
- tcpdump -nn 禁用ip地址和端口的别名解析
- tcpdump 'tcp dst port 80' 设置过滤
- tcpdump 'port 80 and host www.baidu.com' 设置过滤
- tcpdump -F a.bpf 指定一个负载的过滤器

#### 说明

1. 类型的关键字

```
host：指明一台主机。如：host 10.1.110.110
net：指明一个网络地址，如：net 10.1.0.0
port：指明端口号：如：port 8090
```
 

2. 确定方向的关键字

```
src：ip包的源地址，如：src 10.1.110.110
dst：ip包的目标地址。如：dst 10.1.110.110
```
 

3. 协议的关键字（缺省是所有协议的信息包）

```
fddi、ip、arp、rarp、tcp、udp
```
 

4. 其它关键字

```
gateway、broadcast、less、greater
```
 

5. 常用表达式

```
! or not
&& or and
|| or or
```
 

6. 参数详解

```
A：以ascii编码打印每个报文（不包括链路的头）。
a：将网络地址和广播地址转变成名字。
c：抓取指定数目的包。
C：用于判断用 -w 选项将报文写入的文件的大小是否超过这个值，如果超过了就新建文件（文件名后缀是1、2、3依次增加）；
d：将匹配信息包的代码以人们能够理解的汇编格式给出；
dd：将匹配信息包的代码以c语言程序段的格式给出；
ddd：将匹配信息包的代码以十进制的形式给出；
D：列出当前主机的所有网卡编号和名称，可以用于选项 -i；
e：在输出行打印出数据链路层的头部信息；
f：将外部的Internet地址以数字的形式打印出来；
F<表达文件>：从指定的文件中读取表达式,忽略其它的表达式；
i<网络界面>：监听主机的该网卡上的数据流，如果没有指定，就会使用最小网卡编号的网卡（在选项-D可知道，但是不包括环路接口），linux 2.2 内核及之后的版本支持 any 网卡，用于指代任意网卡；
l：如果没有使用 -w 选项，就可以将报文打印到 标准输出终端（此时这是默认）；
n：显示ip，而不是主机名；
nn：显示port，而不是服务名；
N：不列出域名；
O：不将数据包编码最佳化；
p：不让网络界面进入混杂模式；
q：快速输出，仅列出少数的传输协议信息；
r<数据包文件>：从指定的文件中读取包(这些包一般通过-w选项产生)；
s<数据包大小>：指定抓包显示一行的宽度，-s0表示可按包长显示完整的包，经常和-A一起用，默认截取长度为60个字节，但一般ethernet MTU都是1500字节。所以，要抓取大于68字节的包时，使用默认参数就会导致包数据丢失；
S：用绝对而非相对数值列出TCP关联数；
t：在输出的每一行不打印时间戳；
tt：在输出的每一行显示未经格式化的时间戳记；
T<数据包类型>：将监听到的包直接解释为指定的类型的报文，常见的类型有rpc （远程过程调用）和snmp（简单网络管理协议）；
v：输出一个稍微详细的信息，例如在ip包中可以包括ttl和服务类型的信息；
vv：输出详细的报文信息；
x/-xx/-X/-XX：以十六进制显示包内容，几个选项只有细微的差别，详见man手册；
w<数据包文件>：直接将包写入文件中，并不分析和打印出来；
expression：用于筛选的逻辑表达式；
```