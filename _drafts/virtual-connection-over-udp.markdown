---
layout: post
title: "基于UDP的虚拟连接"
date: 2016-07-11 21:41 +0800
categories: tech translate game-networking
---

# <span id="top"> __目录__ </span> 
* [简介](#introduction)  
  
# <span id="introduction"> __简介__ </span> [目录](#top)  

嗨，我是GlennFiedler，欢迎来到我的《[游戏程序员的网络须知](http://gafferongames.com/networking-for-game-programmers/)》系列的第三篇文章。

在[上一篇文章](http://www.gafferongames.com/networking-for-game-programmers/sending-and-receiving-packets)中，我演示了如何通过UDP来发送和接收数据包。

鉴于UDP是无连接的，一个UDP套接字可以用来与大量不同的电脑交换数据包。然而在多人游戏中，我们经常仅仅希望在一小部分连接的电脑中交换数据包。

作为探索连接系统的第一步，我们将从最简单的例开始：在两台电脑之间建立基于UDP的连接。

但是首先，我们需要深度挖掘互联网是如何工作的。

# <span id="TheInternetNotASeriesOfTubes"> __The internet not a series of tubes__ </span> [目录](#top)  

在2006年，参议员Ted Stevens用他关于互联网中立(netneutrality)法案的[著名演讲](http://en.wikipedia.org/wiki/Series_of_tubes)创造了互连网的历史:

> 互联网不是一个你把一些东西放进去的容器，它不是一个大卡车。他是一系列的管子(tubes)。

当我我第一次使用互联网时，和Ted一样。那是1995年，我坐在悉尼大学的电脑实验室中，使用网景浏览器(Netscape Navigator)在“网上冲浪”，而完全不知道其中发生着什么。

