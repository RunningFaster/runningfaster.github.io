---
layout: post
title: Wireshark手册
subtitle: 
cover-img: 
thumbnail-img: /assets/img/Wireshark手册/Snipaste_2024-04-23_18-31-22.png
share-img: 
tags: [install]
author: Jamie
---

# Wireshark工具操作

## 基础知识

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
- tcpdump -F a.bpf 指定一个负载的过滤器

