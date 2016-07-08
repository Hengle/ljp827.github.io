---
layout: post
title: "接发数据包(Sending And Receiving Packets)"
date: 2016-07-07 22:31 +0800
categories: tech translate game-networking
---


# <span id="top"> __目录__ </span> 
* [简介](#introduction)  
* [BSD sockets](#BSDSockets)
* [Platform specifics](#PlatformSpecifics)
* [Initializing the socket layer](#InitializingTheSocketLayer)
* [Creating a socket](#CreatingASocket)
* [Setting the socket as non-blocking](#SettingTheSocketAsNonBlocking)
* [Sending packets](#SendingPackets)
* [Receiving packets](#ReceivingPackets)
* [Destroying a socket](#DestroyingASocket)
* [Socket class](#SocketClass)
* [Conclusion](#Conclusion)
  
# <span id="introduction"> __简介__ </span> [目录](#top)  

嗨，我是GlennFiedler，欢迎来到我的《[游戏程序员的网络须知](http://gafferongames.com/networking-for-game-programmers/)》系列的第二篇文章。

在[上一篇文章](http://ljp827.github.io/tech/translate/game-networking/2016/06/18/udp-vs-tcp.html)中，我们讨论了电脑间发送数据的可选方法，并且决定使用UDP代替TCP。我们选择UDP是为了我们的数据可以及时的发送，而不必因为等待数据包重发而导致数据堆积。

现在，我来演示如何使用UDP来发送和接收数据包。


# <span id="BSDSockets"> __BSD sockets__ </span> [目录](#top)  

在大多数现代的平台上，你都有某种基于BSD套接字(socket)提供的套接字层。

BSD套接字通过使用类似"socket"、"bind"、"sendto"以及"recvfrom"的简单函数来控制。如果你愿意的话，你完全可以使用这些函数来工作，但是这样代码很难做到平台无关(platform independent)，因为每个平台的套接字有些细微的差别。

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

# <span id="InitializingTheSocketLayer"> __Initializing the socket layer__ </span> [目录](#top)  

大部分类Unix系统(包括macosx)并不需要特殊的步骤来初始化sockets层，然而Windows需要一些特殊的步骤来让套接字代码工作，你必须在调用socket函数之前调用`WSAStartup`来初始化套接字层，在用完之后(通信完成)调用`WSACleanup`来关闭。

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

下一步，我们绑定这个UDP套接字到一个端口号(比如30000)上。每个套接字必须绑定到一个唯一的端口上，因为当一个数据包到来时，(数据包里的)端口号用来决定这个数据包交付给哪个套接字。不要使用低于1024的端口号，这些端口号是为系统所保留。

特例：如果你不关心你的套接字绑定端口，你可以传递`0`作为端口号，系统会选择一个空闲的端口号给你。

	sockaddr_in address;
	address.sin_family = AF_INET;
	address.sin_addr.s_addr = INADDR_ANY;
	address.sin_port = htons((unsigned short)port);
	
	if(bind(handle, (const sockaddr*)&address, sizeof(sockaddr_in)) < 0)
	{
		printf("failed to bind socket\n");
		return false
	}

万事俱备，可以接发数据包了。

但是，上边代码里神秘的`htons`调用是什么鬼？这只是一个把16位整数从本机字节序(小端或者大端)转换成网络字节序(大端字节序)。当你要把一个整形数设置到socket结构中时，转换是必须的。

这篇文章中，你将会多次看到`htons`(short的主机序转网络序)以及它的32位版本`htonl`(long型的主机序转网络序)的使用，所以注意一下，你知道其中到底发生了什么。

# <span id="SettingTheSocketAsNonBlocking"> __Setting the socket as non-blocking__ </span> [目录](#top)  

缺省情况下，套接字是被设置为“阻塞模式”的。这意味着，如果你试图调用`recvfrom`读取一个数据包时，函数会一直阻塞，直到有一个可用的包可被读取为止。这与我们的目的背道而驰。视频游戏是实时程序，每一秒中都会有30-60帧的模拟，它们不能因为一个数据包没到而等待。

解决方法是：创建你的socket之后，把他设置为“非阻塞模式(non-blocking mode)”。这样，在没有可读数据包的情况下，`recvfrom`会立马返回，返回值表示你待会儿需要重新读取数据包。

下边是把一个socket设置为非阻塞模式：

	#if PLATFORM == PLATFORM_MAC || PLATFORM == PLATFORM_UNIX
	
	int nonBlocking = 1;
	if(fcntl(handle, F_SETFL, O_NONBLOCK, nonBlocking) == -1)
	{
		printf("failed to set non-blocking\n");
		return false;
	}
	
	#elif PLATFORM == PLATFORM_WINDOWS
	
	DWORD nonBlocking = 1;
	if(ioctlsocket(handle, FIONBIO, &nonBlocking) != 0)
	{
		printf("failed to set non-blocking\n");
		return false;
	}

	#endif

如你所见，Windows环境不提供`fcntl`函数，所以我们使用`ioctlsocket`函数代替。

# <span id="SendingPackets"> __Sending packets__ </span> [目录](#top)  

UDP是无连接协议，所以每次发送数据包，你都需要指定目标地址。你可以使用UDP给任意不同IP地址发送数据包，而在另一端并没有一个UDP套接字与你相连。

下边是发送一个数据包到指定地址：

	int send_bytes = sendto(handle, (const char*)packet_data, packet_size, 0, (sockaddr*)&address, sizeof(sockaddr_in));
	if(send_bytes != packet_size)
	{
		printf("failed to send packet\n");
		return false;
	}

务必注意：`sendto`函数的返回值仅仅表示数据包是否从本机成功发送，它**不能**告诉你数据包是否成功被目标机器接收到。UDP没有办法确定数据包是否到达目标机器。

在上边的代码里，我们传递`sockaddr_in`结构作为目标地址。那如何设置这个结构呢？

比如说我们想发送到207.45.186.98:30000

一开始我们的地址格式如下：

	unsigned int a = 207;
	unsigned int b = 45;
	unsigned int c = 186;
	unsigned int d = 98;
	unsigned short port = 30000;

我们需要做一些工作让它变成`sendto`需要的格式：

	unsigned int address = (a << 24) | (b << 16) | (c << 8) | d;
	sockaddr_in addr;
	addr.sin_family = AF_INET;
	addr.sin_addr.s_addr = htonl(address);
	addr.sin_port = htons(port);

如你所看到的，我们一开始把处于`[0,255]`范围内的数字a,b,c,d组合到一个整型数据中，后者的每一个字节分别对应不同的输入值。然后我们使用整型地址以及端口号来初始化`sockaddr_in`结构。我们确保使用`htons`和`htonl`来把我们的整型地址和端口号从主机序转换成网络序。

特例：如果你想给自己发送一个数据包，并不需要查询本机的IP地址，只需要传递环回地址`127.0.0.1`(给`sendto`函数)，数据包就会发送到本机。


# <span id="ReceivingPackets"> __Receiving packets__ </span> [目录](#top)  

一旦你把一个UDP套接字绑定到一个端口，任何发送到本机该端口的UDP数据包都会放置到一个队列中。通过循环和调用`recvfrom`就可以接受数据包，知道返回值表示队列中没有更多的数据包。

因为UDP是无连接的，所以数据包可能从许多不同的电脑到达。每次你通过`recvfrom`获取一个数据包，同时也可以获得发送者的IP地址和端口号，所以你知道这个数据包从哪里来的。

下边是如何循环以及接收所有到来的数据包：

	while(true)
	{
		unsigned char packet_data[256];
	
		unsigned int max_packet_size = sizeof(packet_data);
	
	#if PLATFORM == PLATFORM_WINDOWS
		typedef int socklen_t;
	#endif
	
		sockaddr_in from;
		socklen_t fromLength = sizeof(from);
	
		int bytes = recvfrom(socket, (char*)packet_data, max_packet_size, 0, (sockaddr*)&from, &fromLength);
		if(bytes <= 0)
			break;
	
		unsigned int from_address = ntohl(from.sin_addr.s_addr);
		unsigned int from_port = ntohs(from.sin_port);
	
		// process received packet
	}

队列中那些比你的接收缓存(receive buffer)大的数据包会被悄悄的丢掉。所以如果你有一个像如上代码一样的一个256字节的缓存，而别人发送给你一个300字节的数据包，那么这个数据包就会被丢到。你不会仅仅收到这个数据包的前256个字节。

考虑到你在写自己的游戏网络协议，只要确保你的接收缓存足够大去接收你的代码可能发送的最大数据包，这在实战中没有一点问题。

# <span id="DestroyingASocket"> __Destroying a socket__ </span> [目录](#top)  

大多数类Unix平台中，sockets就是文件描述符(file handles)，所以你用完之后只需要调用标准文件函数`close`来清理sockets。然而，Windows有一点不同，你需要`closesocket`来代替。

	#if PLATFORM == PLATFORM_MAC || PLATFORM == PLATFORM_UNIX
	close(socket);
	#elif PLATFORM == PLATFORM_WINDOWS
	closesocket(socket);
	#endif

去你的Windows!

# <span id="SocketClass"> __Socket class__ </span> [目录](#top)  

至此，我们已经介绍了所有基本的操作：创建套接字，绑定到一个端口，设置非阻塞模式，发送和接收数据包，销毁套接字。

但是你意识到了这些操作(函数)是平台相关的，每次进行套接字操作的时候，你需要记住使用`#ifdef`来区分平台，这挺烦的。

我们打算通过包装套接字的功能到一个`Socket`类中，来解决这一问题。同时，也会添加一个`Address`类来让指定网络地址变得更加容易。这就避免了每次接发数据时需要手动编码或者解码`sockaddr_in`结构。

如下是我们的socket类长相：

	class Socket
	{
	public:
		Socket();
		~Socket();
	
		bool Open(unsigned short port);
		void Close();
		bool IsOpen() const;
		bool Send(const Address& destination, const void * data, int size);
		int Receive(Address & sender, void * data, int size);
	private:
		int handle;
	};

如下是我们的Address类长相：

	class Address
	{
	public:
		Address();
		Address(unsigned char a, unsigned char b, unsigned char c, unsigned char d, unsigned short port);
		Address(unsigned int address, unsigned short port);
	
		unsigned int GetAddress() const;
		unsigned char GetA() const;
		unsigned char GetB() const;
		unsigned char GetC() const;
		unsigned char GetD() const;
		unsigned short GetPort() const;
	
	private:
		unsigned int address;
		unsigned short port;
	};

下边是你如何使用这些类来发送和接收数据包：

	// create socket
	const int port = 30000;
	Socket socket;
	if (!socket.Open(port))
	{
		printf("failed to create socket\n");
		return false;
	}
	
	// send a packet
	
	const char data[] = "hello world!";
	socket.Send(Address(127.0.0.1, port), data, sizeof(data));
	
	// receive packets
	
	while(true)
	{
		Address sender;
		unsigned char buffer[256];
		int bytes_read = socket.Receive(sender, buffer, sizeof(buffer));
	
		if (!bytes_read)
			break;
	
		// process packet
	}

这样比直接使用BSD套接字简单多了。额外好处是：这个代码在所有平台上是一样的，因为所有平台相关的处理已经放到`Socket`和`Address`类里了。

# <span id="Conclusion"> __Conclusion__ </span> [目录](#top)  

现在我们有了平台独立的方式来发送和接收UDP数据包了

UDP是无连接的，我想创建一个例子代码来彻底阐明观点(hammer point home)。我设置了一个[例子程序](http://netgame.googlecode.com/files/SendingAndReceivingPackets.zip)，它从一个文本文件中读取IP地址，并且每秒为这些IP地址发送一个数据包。每当这个程序收到一个数据包，它会打印出来收到数据包的来源地址以及数据包的大小。

在你的本机上设置大量的节点，让其互相发送数据包，这对你来说应该十分简单：只需要多开一些这个程序的例程，并传递不同的端口号即可，具体如下：

	> Node 30000
	> Node 30001
	> Node 30002
	> Node 30003
	etc...

然后每个节点会尝试互相发送数据包，就像一个迷你的P2P程序。

我是在MacOS上开发了这个程序，但是你应该能在任何类UNIX系统或者Windows系统上编译它，所以如果你在不同的机器上有一些兼容性补丁，**请让我知道**。
