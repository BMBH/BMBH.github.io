---
title: 'C# Socket模拟发送接收'
date: 2018-05-11 10:34:29
tags: Socket
categories: .NET
---

### C# Socket模拟发送接收

- Socket简介


	通过TCP/IP与仪器或设备通讯，在C#语言中，我们通常采用Socket。本项目是一个简单的Socket建立服务监听与Socket作为客户端请求的一个示例。

- 项目结构

	- 客户端项目 SocketClient

		主要负责作为Socket客户端发起连接请求，并发送数据

	- 服务端项目 SocketDemo

		主要负责作为Socket服务端，监听端口并接收连接请求，并返回应答数据

- 项目演示

	- 先运行SocketDemo进行服务监听
	- 运行SocketClient进行模拟连接，并发送接收数据。

- 后记

	H5支持WebSocket，预计将来在通讯领域应用会更加广泛，示例程序见博客：[C# WebSocket模拟发送接收](http://www.cnblogs.com/bmbh/p/5174884.html)

- GitHub

	[BMBH/.NET-Demo](https://github.com/BMBH/.NET-Demo/tree/master/SocketDemo)