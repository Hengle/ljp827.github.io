---
layout: post
title: "接发数据包(Sending And Receiving Packets)"
date: 2016-07-07 22:31 +0800
categories: tech translate game-networking
---


# <span id="top"> __目录__ </span> 
* [简介](#introduction)  
  
# <span id="introduction"> __简介__ </span> [目录](#top)  

嗨，我是GlennFiedler，欢迎来到我的《[游戏程序员的网络须知](http://gafferongames.com/networking-for-game-programmers/)》系列的第一篇文章。

在[上一篇文章](http://ljp827.github.io/tech/translate/game-networking/2016/06/18/udp-vs-tcp.html)中，我们讨论了电脑间发送数据的可选方法，并且决定使用UDP代替TCP。我们选择UDP是为了我们的数据可以及时的发送，而不必因为等待数据包重发而导致数据堆积。

现在，我来演示如何使用UDP来发送和接收数据包。


# <span id="BSDSockets"> __BSD sockets__ </span> [目录](#top)  

在大多数现代的平台上，你都有某种基于BSD套接字(socket)提供的套接字层。

BSD套接字通过使用类似"socket"、"bind"、"sendto"以及"recvfrom"的简单函数来操作。如果你愿意的话，你完全可以使用这些函数来工作，但是这样代码很难做到平台无关(platform independent)，因为每个平台的套接字有些细微的差别。

所以，尽管一开始我会演示BSD套接字代码例子来说明基本的socket用法，但是我们以后不会这么做。相反，一旦我们了解了所有的socket基本功能，我们会把它抽象成一些类，这样就更容易写跨平台代码。


# <span id="PlatformSpecifics"> __Platform specifics__ </span> [目录](#top)  

首先，定义一部分宏(define)来让我们检测我们的当前平台，这样我们可以处理不同平台之间的细微差别:

	// platform detection
	#define PLATFORM_WINDOWS	1
	#define PLATFORM_MAC		2
	#define PLATFORM_UNIX		3
	#if defined(_WIN32)
	#define PLATFORM PLATFORM_WINDOWS
	#elif defined(__APPLE__)
	#define PLATFORM PLATFORM_MAC
	#else
	#define PLATFORM PLATFORM_UNIX
	#endif

现在我们添加套接字所需头文件。因为不同平台头文件不一样，我们用`#define`来区分不同平台，并添加不同的头文件。

	#if PLATFORM == PLATFORM_WINDOWS
		#include <winsocks.h>
	#elif PLATFORM == PLATFORM_MAC || PLATFORM == PLATFORM_UNIX
		#include <sys/socket.h>
		#include <netinet/in.h>
		#include <fcntl.h>
	#endif

在类Unix平台里，套接字是内置的标准库，所以我们不需要额外链接其他东西。然而在Windows系统里，我们需要链接`winsock`库以保证正常工作。

下边是一个不需要你改变你的工程或者Makefile文件的窍门:

	#if PLATFORM == PLATFORM_WINDOWS
	#pragma comment(lib, "wsock32.lib")
	#endif

我喜欢这个诀窍，因为我超懒。当然，你也可以总是在你的工程或者makefile中指定链接这个库。

# <span id="InitializingTheSocketLayer"> 1 __Initializing the socket layer__ </span> [目录](#top)  

大部分类Unix系统(包括macosx)并不需要特殊的步骤来初始化sockets层，然后Windows需要一些特殊的步骤来让套接字代码工作，你必须在调用socket函数之前调用`WSAStartup`来初始化套接字层，在用完之后(通信完成)调用`WSACleanup`来关闭。

我们添加两个新函数：

	bool InitializeSockets()
	{
	#if PLATFORM == PLATFORM_WINDOWS
		WSAData WsaData;
		return WSAStartup(MAKEWORD(2,2), &WsaData) == NO_ERROR;
	#else
		return true;
	#endif
	}
	
	void ShutdownSockets()
	{
	#if PLATFORM == PLATFORM_WINDOWS
		WSACleanup();
	#endif
	}

现在我们有了跨平台方法来初始化套接字层。在不需要初始化的平台，这些函数不做任何事情。


# <span id="CreatingASocket"> __Creating a socket__ </span> [目录](#top)  

是时候创建一个UDP套接字了，如下：

	int handle = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
	if (handle <= 0)
	{
		printf("failed to create socket\n");
		return false;
	}

下一步，我们绑定这个UDP套接字到一个端口号(比如30000)上。每个套接字必须绑定到一个唯一的端口上，因为当一个数据包到来时，(数据包里的)端口号用来决定这个数据包交付给哪个套接字。不要使用低于1024的端口号，因为这些端口号是为系统所保留。

特例：如果你不关心你的套接字绑定端口，你可以传递`0`作为端口号，系统会选择一个空闲的端口号给你。



