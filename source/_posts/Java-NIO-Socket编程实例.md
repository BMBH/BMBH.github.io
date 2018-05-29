---
title: Java NIO Socket编程实例
date: 2018-05-29 17:42:22
tags: Java
categories: Java
---


##### 各I/O模型优缺点

- BIO通信模型

	BIO主要的问题在于每当有一个新的客户端请求接入时，服务端必须创建一个新的线程处理新接入的客户端链路，一个线程只能处理一个客户端连接

- 线程池I/O编程

	假如所有可用线程都被阻塞，后续I/O都将在队列中排队
	线程池采用阻塞队列实现，队列积满之后，后续入队列操作将被阻塞，新的客户端请求被拒绝，发生大量连接超时

- NIO编程
	- 缓冲区Buffer 

		每一种Java基本类型都有对一种缓冲区
		大多数标准I/O使用ByteBuffer

	- 通道Channel

		Channel分为两大类：用于网络读写的SelectableChannel和用于文件操作的FileChannel

	- 多路复用器Selector

		多路复用器提供选择已经就绪的任务的能力

- NIO2.0 AIO

	异步套接字通道不需要通过多路复用器(Selector)对注册的通道进行轮询操作即可实现异步读写

##### NIO实例分析

- NIO服务端序列
	- 步骤一：打开ServerSocketChannel，用于监听客户端的连接
	- 步骤二：绑定监听端口，设置连接为非阻塞模式
	- 步骤三：创建Reactor线程，创建多路复用器并启动线程
	- 步骤四：将ServerSocketChannel注册到Reactor线程的多路复用器Selector上，监听ACCEPT事件
	- 步骤五：多路复用器在线程run方法的无限循环体内轮休准备就绪的Key
	- 步骤六：多路复用器监听到有新的客户端接入，处理新的计入请求，完成TCP三次握手，建立物理链路
	- 步骤七：设置客户端链路为非阻塞模式
	- 步骤八：将新接入的客户端连接注册到Reactor线程的多路复用器上，监听读操作
	- 步骤九：异步读取客户端请求消息到缓冲区
	- 步骤十：对ByteBuffer进行编解码，如果有半包消息指针reset，继续读取后续的报文，将解码成功的消息封装成Task，投递到业务线程池中
	- 步骤十一：将POJO对象encode成ByteBuffer，调用SocketChannel的异步write接口，将消息异步发送给客户端

- NIO客户端序列
	- 步骤一：打开SocketChannel，绑定客户端本机地址
	- 步骤二：设置SocketChannel为非阻塞模式，设置客户端连接的TCP参数
	- 步骤三：异步连接服务器
	- 步骤四：判断是否连接成功，如果连接成功，则直接注册读状态位到多路复用器中，如果当前没有连接成功
	- 步骤五：向Reactor线程的多路复用器注册OP_CONNECT状态位，监听服务端的TCP ACK应答
	- 步骤六：创建Reactor线程，创建多路复用器并启动线程
	- 步骤七：多路复用器在线程run方法的无限循环体内轮询准备就绪的Key
	- 步骤八：接收connect事件进行处理
	- 步骤九：判断连接结果，如果连接成功，注册读事件到多路复用器
	- 步骤十：注册读事件到多路复用器
	- 步骤十一：异步读客户端请求消息到缓冲区
	- 步骤十二：对ByteBuffer进行编解码，如果有半包消息接收缓冲区Reset，继续读取后续的报文，将解码成功的消息封装成Task，投递到业务线程池中，进行业务逻辑编排。
	- 步骤十三：将POJO对象encode成ByteBuffer，调用SocketChannel的异步write接口，将消息异步发送给客户端

##### NIO实例代码

- 服务端

```
    /**
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException{
        int port = 8080;
        if(args != null &&args.length >0){
            try{
                port = Integer.valueOf(args[0]);
            }catch (NumberFormatException ex){
                //采用默认值
            }
        }
        MultiplexerTimeServer timeServer = new MultiplexerTimeServer(port);
        new Thread(timeServer,"NIO-MultiplexerTimeServer-001").start();
    }
```
```
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class MultiplexerTimeServer implements Runnable {

    private Selector selector;
    private ServerSocketChannel serverChannel;
    private volatile  boolean stop;

    /**
     * 初始化多路复用器，绑定监听端口
     * @param port
     */
    public MultiplexerTimeServer(int port){
        try{
            selector = Selector.open();//创建多路复用器
            serverChannel = ServerSocketChannel.open();
            serverChannel.configureBlocking(false);//设置为异步非阻塞模式
            serverChannel.socket().bind(new InetSocketAddress(port),1024);//绑定端口
            serverChannel.register(selector,SelectionKey.OP_ACCEPT);//注册到Selector
            System.out.println("The time server is start in port:" + port);
        }catch (IOException e){
            e.printStackTrace();
            System.exit(1);
        }
    }

    public void stop(){
        this.stop = true;
    }

    public void run(){
        while(!stop){
            try{
               selector.select(1000);//selector每隔1s都被唤醒一次
               Set<SelectionKey> selectedKeys = selector.selectedKeys();
               Iterator<SelectionKey> it = selectedKeys.iterator();
               SelectionKey key = null;
               while(it.hasNext()){
                   key = it.next();
                   it.remove();
                   try{
                       handleInput(key);
                   }catch (Exception e){
                       if(key !=null){
                           key.cancel();
                           if(key.channel() !=null)
                               key.channel().close();
                       }
                   }
               }
            }catch (Throwable t){
                t.printStackTrace();
            }
        }
        //多路复用器关闭后，所有注册在上面的Channel和Pipe等资源都会被自动去注册并关闭，所以不需要重复释放资源
        if(selector != null){
            try{
                selector.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }

    private void handleInput(SelectionKey key) throws IOException{
        if(key.isValid()){
            //处理新接入的请求消息
            if(key.isAcceptable()){
                //Accept the new connection
                ServerSocketChannel ssc = (ServerSocketChannel)key.channel();
                SocketChannel sc = ssc.accept();//接收客户端的连接请求，完成TCP三次握手
                sc.configureBlocking(false);//设置为异步非阻塞
                //Add the new connection to the selector
                sc.register(selector,SelectionKey.OP_READ);
            }
            if(key.isReadable()){
                //Read the data
                SocketChannel sc = (SocketChannel)key.channel();
                ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                int readBytes = sc.read(readBuffer);
                if(readBytes > 0 ){
                    readBuffer.flip();//将缓冲区当前的limit设置为position,position设置为0
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes);
                    String body = new String(bytes,"UTF-8");
                    System.out.println("The time server receive order :" + body);
                    String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body)?new java.util.Date(System.currentTimeMillis()).toString():"BAD ORDER";
                    doWrite(sc,currentTime);
                }else if(readBytes <0){
                    //对端链路关闭
                    key.cancel();
                    sc.close();
                }else{
                    //读到0字节，忽略
                }
            }
        }
    }

    /**
     * 将应答消息异步发送给客户端
     * @param channel
     * @param response
     * @throws IOException
     */
    private void doWrite(SocketChannel channel,String response) throws IOException{
        if(response !=null && response.trim().length() >0){
            byte[] bytes = response.getBytes();
            ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
            writeBuffer.put(bytes);
            writeBuffer.flip();
            channel.write(writeBuffer);
        }
    }
}
```

- 客户端

```
    /**
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException{
        int port = 8080;
        if(args != null &&args.length >0){
            try{
                port = Integer.valueOf(args[0]);
            }catch (NumberFormatException ex){
                //采用默认值
            }
        }
        new Thread(new TimeClientHandle("127.0.0.1",port),"TimeClient-001").start();
    }
```
```
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class TimeClientHandle implements Runnable{
    private String host;
    private int port;
    private Selector selector;
    private SocketChannel socketChannel;
    private volatile boolean stop;

    public TimeClientHandle(String host,int port){
        this.host = host == null?"127.0.0.1":host;
        this.port = port;
        try{
            selector = Selector.open();
            socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);
        }catch (IOException e){
            e.printStackTrace();
            System.exit(1);
        }
    }

    public void run(){
        try{
            doConnect();
        }catch (IOException e){
            e.printStackTrace();
            System.exit(1);
        }
        while(!stop){
            try{
                selector.select(1000);
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                Iterator<SelectionKey> it = selectionKeys.iterator();
                SelectionKey key = null;
                while(it.hasNext()){
                    key = it.next();
                    it.remove();
                    try{
                        handleInput(key);
                    }catch (Exception e){
                        if(key != null){
                            key.cancel();
                            if(key.channel() !=null)
                                key.channel().close();
                        }
                    }
                }
            }catch (Exception e){
                e.printStackTrace();
                System.exit(1);
            }
        }
        if(selector !=null){
            try{
                selector.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }

    private void handleInput(SelectionKey key) throws IOException{
        if(key.isValid()){
            //判断是否连接成功
            SocketChannel sc = (SocketChannel)key.channel();
            if(key.isConnectable()){
                if(sc.finishConnect()){
                    sc.register(selector,SelectionKey.OP_READ);
                    doWrite(sc);
                }else{
                    System.exit(1);//连接失败，进程退出
                }
            }
            if(key.isReadable()) {
                ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                int readBytes = sc.read(readBuffer);
                if (readBytes > 0) {
                    readBuffer.flip();//将缓冲区当前的limit设置为position,position设置为0
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes);
                    String body = new String(bytes, "UTF-8");
                    System.out.println("The time server receive order :" + body);
                    this.stop = true;
                } else if (readBytes < 0) {
                    //对端链路关闭
                    key.cancel();
                    sc.close();
                } else {
                    //读到0字节，忽略
                }
            }
        }
    }

    private void doConnect() throws IOException{
        if(socketChannel.connect(new InetSocketAddress(host,port))){
            socketChannel.register(selector,SelectionKey.OP_READ);
            doWrite(socketChannel);
        }else{
            socketChannel.register(selector,SelectionKey.OP_CONNECT);
        }
    }

    private void doWrite(SocketChannel sc) throws IOException {
        byte[] bytes = "QUERY TIME ORDER".getBytes();
        ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
        writeBuffer.put(bytes);
        writeBuffer.flip();
        sc.write(writeBuffer);
        if (!writeBuffer.hasRemaining())
            System.out.println("Send order 2 server succeed.");
    }
}

```

先启动服务端，再启动客户端运行实例。

##### NIO2.0 AIO实例代码

- 服务端

```
    /**
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        int port = 8080;
        if(args != null &&args.length >0){
            try{
                port = Integer.valueOf(args[0]);
            }catch (NumberFormatException ex){
                //采用默认值
            }
        }

        AsyncTimeServerHandler timeServer = new AsyncTimeServerHandler(port);
        new Thread(timeServer,"AIO-AsyncTimeServerHandler-001").start();
    }
```
```
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.util.concurrent.CountDownLatch;

public class AsyncTimeServerHandler implements Runnable {
    private int port;
    CountDownLatch latch;
    AsynchronousServerSocketChannel asynchronousServerSocketChannel;

    public AsyncTimeServerHandler(int port){
        this.port = port;
        try{
            //创建一个异步的服务端通道
            asynchronousServerSocketChannel = AsynchronousServerSocketChannel.open();
            //绑定端口
            asynchronousServerSocketChannel.bind(new InetSocketAddress(port));
            System.out.println("The time server is start in port:"+ port);
        }catch (IOException e){
            e.printStackTrace();
        }
    }

    public void run(){
        latch = new CountDownLatch(1);
        doAccept();
        try{
            latch.await();//允许当前线程阻塞，防止服务端执行完退出
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    public void doAccept(){
        //传递一个CompletionHandler实例来接收通知
        asynchronousServerSocketChannel.accept(this,new AcceptCompletionHandler());
    }
}

```
```
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;

public class AcceptCompletionHandler implements CompletionHandler<AsynchronousSocketChannel, AsyncTimeServerHandler> {

    @Override
    public void completed(AsynchronousSocketChannel result, AsyncTimeServerHandler attachment) {
        //继续接收
        attachment.asynchronousServerSocketChannel.accept(attachment, this);
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        result.read(buffer, buffer, new ReadCompletionHandler(result));
    }

    @Override
    public void failed(Throwable exc, AsyncTimeServerHandler attachment) {
        exc.printStackTrace();
        attachment.latch.countDown();
    }
}
```

```
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;

public class ReadCompletionHandler implements CompletionHandler<Integer, ByteBuffer> {

    private AsynchronousSocketChannel channel;

    public ReadCompletionHandler(AsynchronousSocketChannel channel){
        if(this.channel == null){
            this.channel = channel;
        }
    }

    @Override
    public void completed(Integer result,ByteBuffer attachment){
        attachment.flip();
        byte[] body = new byte[attachment.remaining()];
        attachment.get(body);
        try{
            String req = new String(body,"UTF-8");
            System.out.println("The time server receive order:"+req);
            String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(req)?
                    new java.util.Date(System.currentTimeMillis()).toString():"BAD ORDER";
            doWrite(currentTime);
        }catch (UnsupportedEncodingException e){
            e.printStackTrace();
        }
    }

    private void doWrite(String currentTime){
        if(currentTime !=null && currentTime.trim().length()>0){
            byte[] bytes = (currentTime).getBytes();
            final ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
            writeBuffer.put(bytes);
            writeBuffer.flip();
            channel.write(writeBuffer, writeBuffer, new CompletionHandler<Integer, ByteBuffer>() {
                @Override
                public void completed(Integer result, ByteBuffer buffer) {
                    //如果没有发送完成，继续发送
                    if(buffer.hasRemaining())
                        channel.write(buffer,buffer,this);
                }

                @Override
                public void failed(Throwable exc, ByteBuffer attachment) {
                    try{
                        channel.close();
                    }catch (IOException e){
                        //ingnore on close
                    }
                }
            });
        }
    }

    public void failed(Throwable exc,ByteBuffer attachment){
        try{
            this.channel.close();
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

- 客户端

```
    /**
     *
     * @param args
     * @throws IOException
     */
    public static void main(String[] args) throws IOException {
        int port = 8080;
        if(args != null &&args.length >0){
            try{
                port = Integer.valueOf(args[0]);
            }catch (NumberFormatException ex){
                //采用默认值
            }
        }
        new Thread(new AsyncTimeClientHandler("127.0.0.1",port),"AIO-AsyncTimeClientHandler-001").start();
    }
```
```
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.util.concurrent.CountDownLatch;

public class AsyncTimeClientHandler implements CompletionHandler<Void,AsyncTimeClientHandler>,Runnable {
    private AsynchronousSocketChannel client;
    private String host;
    private int port;
    private CountDownLatch latch;

    public AsyncTimeClientHandler(String host,int port){
        this.host = host;
        this.port = port;
        try{
            client = AsynchronousSocketChannel.open();
        }catch (IOException e){
            e.printStackTrace();
        }
    }

    @Override
    public void run(){
        latch = new CountDownLatch(1);
        client.connect(new InetSocketAddress(host,port),this,this);
        try{
            latch.await();
        }catch (InterruptedException el){
            el.printStackTrace();
        }
        try{
            client.close();
        }catch (IOException e){
            e.printStackTrace();
        }
    }

    @Override
    public void completed(Void result,AsyncTimeClientHandler attachment){
        byte[] req = "QUERY TIME ORDER".getBytes();
        ByteBuffer writeBuffer = ByteBuffer.allocate(req.length);
        writeBuffer.put(req);
        writeBuffer.flip();
        client.write(writeBuffer, writeBuffer,
                new CompletionHandler<Integer, ByteBuffer>() {
                    @Override
                    public void completed(Integer result, final ByteBuffer buffer) {
                        if(buffer.hasRemaining()){
                            client.write(buffer,buffer,this);
                        }else{
                            ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                            client.read(
                                    readBuffer,
                                    readBuffer,
                                    new CompletionHandler<Integer, ByteBuffer>() {
                                        @Override
                                        public void completed(Integer result, ByteBuffer attachment) {
                                            attachment.flip();
                                            byte[] bytes = new  byte[attachment.remaining()];
                                            attachment.get(bytes);
                                            String body;
                                            try{
                                                body = new String(bytes,"UTF-8");
                                                System.out.println("Now is:"+body);
                                                latch.countDown();
                                            }catch (UnsupportedEncodingException e){
                                                e.printStackTrace();
                                            }
                                        }

                                        @Override
                                        public void failed(Throwable exc, ByteBuffer attachment) {
                                            try{
                                                client.close();
                                                latch.countDown();
                                            }catch (IOException e){
                                                //ingnore on close
                                            }
                                        }
                                    }
                            );
                        }
                    }

                    @Override
                    public void failed(Throwable exc, ByteBuffer attachment) {
                        try{
                            client.close();
                            latch.countDown();
                        }catch (IOException e){
                            //ingnore on close
                        }
                    }
                });
    }

    @Override
    public void failed(Throwable exc,AsyncTimeClientHandler attachment){
        exc.printStackTrace();
        try{
            client.close();
            latch.countDown();
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

##### GitHub地址
[Java-DEMO/nettys/](https://github.com/BMBH/Java-DEMO/tree/master/nettys)