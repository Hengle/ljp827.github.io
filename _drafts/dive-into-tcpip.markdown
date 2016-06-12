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
运输层在网络层的基础之上，使用[IP协议](#2.1)，实现了两个常用的协议:[TCP协议](#2.4)和[UDP协议](#2.5)，这两个协议为两台主机上的应用程序提供了端对端的通信。
4. 应用层  
应用层拥有更多我们使用的协议，这些协议大都建立在[TCP协议](#2.4)和[UDP协议](#2.5)基础之上，如Telnet，FTP等协议。

## <span id="1.2"> 1.2 __封装__ </span> [目录](#top)  
　　当应用程序使用TCP传送数据时，该数据流(比特流)经过每一层时，每一层都会对收到的数据增加一些首部信息(有的还要增加尾部信息)。如下图所示，TCP传给IP的数据单元乘坐TCP报文段(TCP segment), IP传给网络接口的数据单元称作IP数据包(IP datagram), 通过以太网传输的比特流成为帧(Frame)。

![tcp ip package][tcp_ip_package.image]  

　　除了[TCP协议](#2.4)之外，还有[UDP协议](#2.5)，[ICMP协议](#2.2)，以及[IGMP协议](#2.3)甚至应用层的用户直接使用[IP协议](#2.1)，所以运输层协议在生成报文首部时要存入一个长度为8bit的数值，称作**协议域**。同样，一台主机上的多个程序可以同时使用[TCP协议](#2.4)和[UDP协议](#2.5)，所以在[TCP协议](#2.4)和[UDP协议](#2.5)的首部数据中，有一个16bit的[__端口号__](#1.4)用于区分不同的应用程序。TCP和UDP协议把源端口号和目的端口号分别存入报文首部中。  

## <span id="1.3"> 1.3 __分用__ </span> [目录](#top)  

　　分用是封装的逆过程，当目的主机收到一个以太网数据帧时，数据就开始从协议栈中由底向上升，同时去掉每一层协议加上的报文首部。每层协议都要检查报文首部的协议标识，以确定接收数据的上层协议。这个过程称为**分用**(Demultiplexing).  

## <span id="1.4"> 1.4 __端口号__ </span> [目录](#top)  

* TCP和UDP采用16bit的端口号来识别应用程序。
* 服务器一般使用知名端口号(1-255)提供服务，如SSH使用22端口。
* 客户端不关心自己使用的端口号，一般使用的是临时端口号(1024-5000)。
* 可以查看系统的`/etc/services`来查看常见的端口号。
  
# <span id="2"> 2 __协议__ </span> [目录](#top)

## <span id="2.1"> 2.1 __IP协议__ </span> [目录](#top)

　　本节主要解释IP协议首部数据结构。  
　　IP协议是TCP/IP协议族中最核心的协议，它提供__不可靠__，__无连接__的数据包传送服务。__不可靠__是它不能保证IP数据包能成功到达目的地。__无连接__是指IP并不维护关于后续数据包的状态信息。  
　　IP协议的首部结构如下图所示：  
![ip header][ip_header.image]

## <span id="2.2"> 2.2 __IMCP协议__ </span> [目录](#top)  

## <span id="2.3"> 2.3 __IGMP协议__ </span> [目录](#top)  


[tcp_ip_layer.image]: https://raw.githubusercontent.com/ljp827/ljp827.github.io/master/mePic/tcpip/tcp_ip%20layer.jpg "TCP/IP 分层"
[tcp_ip_package.image]: https://raw.githubusercontent.com/ljp827/ljp827.github.io/master/mePic/tcpip/tcp_ip%20package.jpg "TCP/IP 封装"
[ip_header.image]: https://raw.githubusercontent.com/ljp827/ljp827.github.io/master/mePic/tcpip/ip%20header.jpg "IP 协议首部结构"
