---
layout: post
title:  "TCP/IP 协议详解!"
date:   2016-05-29 00:31:30 +0800
categories: jekyll techs
---

# <span id="top"> 目录 </span>
* [1 概述](#1)
	* [1.1 分层](#1.1)
	* [1.2 封装](#1.2)
	* [1.3 分用](#1.3)
	* [1.4 端口号](#1.4)
* [2 协议](#2)
	* [2.1 IP协议](#2.1)
	* [2.2 IMCP协议](#2.2)
	* [2.3 IGMP协议](#2.3)
	* [2.4 TCP协议](#2.4)
	* [2.5 UDP协议](#2.5)

# <span id="1"> 1 __概述__ </span> [目录](#top)  

## <span id="1.1"> 1.1 __分层__ </span> [目录](#top)  
网络协议通常会分成不同的层级，分别进行开发，每一层分别负责不同的通信功能。TCP/IP协议族通常分四层，如下图:  

![tcp ip layer][tcp_ip_layer.image]  

从下到上分别负责不同的功能：

1. 链路层
链路层，也叫数据链路层或者网络接口层。通常包括操作系统中的设备驱动程序和计算机中对应的网络接口卡。
2. 网络层
网络层，处理分组在网络中的活动。TCP/IP协议族中，网络层包括[IP协议](#2.1)，[ICMP协议](#2.2)，以及[IGMP协议](#2.3)。
3. 运输层
运输层在网络层的基础之上，使用[IP协议](#2.1)，实现了两个常用的协议:[TCP]
4. 应用层

## <span id="1.2"> 1.2 __封装__ </span> [目录](#top)  

## <span id="1.3"> 1.3 __分用__ </span> [目录](#top)  

## <span id="1.4"> 1.4 __端口号__ </span> [目录](#top)  
  
# <span id="2"> 2 __协议__ </span> [目录](#top)

## <span id="2.1"> 2.1 __IP协议__ </span> [目录](#top)

## <span id="2.2"> 2.2 __IMCP协议__ </span> [目录](#top)  

## <span id="2.3"> 2.3 __IGMP协议__ </span> [目录](#top)  


[tcp_ip_layer.image]: https://raw.githubusercontent.com/ljp827/ljp827.github.io/master/mePic/tcpip/tcp_ip%20layer.jpg "TCP/IP 分层"
