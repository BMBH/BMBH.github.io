---
title: 'C# Socket模拟发送接收'
date: 2018-05-11 10:34:29
tags: Socket
categories: .NET
---

#### Socket简介


	通过TCP/IP与仪器或设备通讯，在C#语言中，我们通常采用Socket。本项目是一个简单的Socket建立服务监听与Socket作为客户端请求的一个示例。

#### 项目结构

- 客户端项目 SocketClient

	主要负责作为Socket客户端发起连接请求，并发送数据

- 服务端项目 SocketDemo

	主要负责作为Socket服务端，监听端口并接收连接请求，并返回应答数据

#### 项目演示

- 先运行SocketDemo进行服务监听

```
            Console.WriteLine("Starting:Creating Socket object");
            Socket listener = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
            listener.Bind(new IPEndPoint(IPAddress.Any, 2112));
            listener.Listen(10);

            while (true)
            {
                Console.WriteLine("Waiting for connection on port 2112");
                Socket socket = listener.Accept();
                string receivedValue = string.Empty;

                while (true)
                {
                    byte[] receivedBytes = new byte[1024];
                    int numBytes = socket.Receive(receivedBytes);
                    Console.WriteLine("Receiving.");
                    receivedValue += Encoding.ASCII.GetString(receivedBytes, 0, numBytes);
                    if (receivedValue.IndexOf("[FINAL]") > -1)
                    {
                        break;
                    }
                }

                Console.WriteLine("Received value:{0}", receivedValue);
                string replyValue = "Message successfully received.";
                byte[] replyMessage = Encoding.ASCII.GetBytes(replyValue);
                socket.Send(replyMessage);
                socket.Shutdown(SocketShutdown.Both);
                socket.Close();
            }
```

- 运行SocketClient进行模拟连接，并发送接收数据。

```
            byte[] receivedBytes = new byte[1024];
            IPHostEntry ipHost = Dns.Resolve("127.0.0.1");
            IPAddress ipAddress = ipHost.AddressList[0];
            IPEndPoint ipEndPoint = new IPEndPoint(ipAddress, 2112);
            Console.WriteLine("Starting:Creating Socket object");

            Socket sender = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);

            sender.Connect(ipEndPoint);
            Console.WriteLine("Successfully connected to {0}", sender.RemoteEndPoint);

            string sendingMessage = "Hello World Socket Test";
            Console.WriteLine("Creating Message;Hello World Socket Test");

            byte[] forwardMessage = Encoding.ASCII.GetBytes(sendingMessage + "[FINAL]");


            sender.Send(forwardMessage);

            int totalBytesReceived = sender.Receive(receivedBytes);
            Console.WriteLine("Message provided from server: {0}", Encoding.ASCII.GetString(receivedBytes, 0, totalBytesReceived));

            sender.Shutdown(SocketShutdown.Both);
            sender.Close();
            Thread.Sleep(1000);

            Console.ReadLine();
```

#### 后记

	H5支持WebSocket，预计将来在通讯领域应用会更加广泛，示例程序见博客：[C# WebSocket模拟发送接收](http://www.cnblogs.com/bmbh/p/5174884.html)

#### GitHub

[BMBH/.NET-Demo](https://github.com/BMBH/.NET-Demo/tree/master/SocketDemo)