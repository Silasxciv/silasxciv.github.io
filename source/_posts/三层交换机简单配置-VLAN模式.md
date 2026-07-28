---
layout: post
title: 三层交换机简单配置-VLAN模式
comments: true
tags:
  - 交换机
categories: 
  - 教程
date: 2021-11-28 21:06:15
---
三层交换机的配置除了网关外，还有一种VLAN的方法；
VLAN可以隔绝广播，起到预防广播风暴的作用。
<!--more-->
### 1. 三层交换机配置-网关模式
依然以Cisco的[模拟软件](https://www.netacad.com/zh-hans/courses/packet-tracer)为例，建立一台名为L2-Switch1的交换机，连接两台IP地址分别为192.168.1.1和192.168.2.1的计算机，建立一台名为L2-Switch2的交换机，连接两台IP地址为192.168.1.2和192.168.2.2的计算机，将L2-Switch1交换机连接到L3-Switch交换机的1口，将L2-Switch2交换机连接到L3-Switch交换机的2口。
![](/assets/images/211128_1.jpg)
此时，相同网段的计算机可以相互ping通，不同网段的计算机不能ping通。

进入L2-Switch1交换机的配置界面，键入命令：
```
en  
conf t  
vlan 10
vlan 20
exit
int f0/1
switchport access vlan 10
exit
int f0/2
switchport access vlan 20
exit
int fa0/24
switchport mode trunk
switchport trunk allowed vlan all
```

进入L2-Switch2交换机的配置界面，键入命令：
```
en  
conf t  
vlan 10
vlan 20
exit
int f0/1
switchport access vlan 10
exit
int f0/2
switchport access vlan 20
exit
int fa0/24
switchport mode trunk
switchport trunk allowed vlan all
```

进入L3-Switch交换机的配置界面，键入命令：
```
en  
conf t 
ip touting  
vlan 10
vlan 20
exit
int vlan 10
ip address 192.168.1.253 255.255.255.0
exit
int vlan 20
ip address 192.168.2.253 255.255.255.0
exit
```

在192.168.1.x的两台电脑内设置上述192.168.1.253的默认网关，在192.168.2.x的电脑内设置192.168.2.253的默认网关，就能够实现跨网络的计算机互相访问。
![](/assets/images/211128_2.jpg)

因为对电口设置了VLAN，如果计算机接入交换机未设置的电口，则需要设置VLAN，并在三层交换机和计算机中添加网关，才能进行跨域访问。