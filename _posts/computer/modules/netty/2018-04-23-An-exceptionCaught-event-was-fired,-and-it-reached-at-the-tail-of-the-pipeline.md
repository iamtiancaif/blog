---
layout: post
title:  "An exceptionCaught event was fired, and it reached at the tail of the pipeline"
categories: computer
tags:  computer netty java
author: web 
source: http://www.voidcn.com/article/p-dxqbroah-ee.html
ignore: true
---

* content
{:toc}




问题描述：

    警告: An exceptionCaught() event was fired, and it reached at the tail of the pipeline. It usually means the last handler in the pipeline did not handle the exception. java.io.IOException: 远程主机强迫关闭了一个现有的连接。
        at sun.nio.ch.SocketDispatcher.read0(Native Method)
        at sun.nio.ch.SocketDispatcher.read(Unknown Source)
        at sun.nio.ch.IOUtil.readIntoNativeBuffer(Unknown Source)
        at sun.nio.ch.IOUtil.read(Unknown Source)
        at sun.nio.ch.SocketChannelImpl.read(Unknown Source)
        at io.netty.buffer.UnpooledUnsafeDirectByteBuf.setBytes(UnpooledUnsafeDirectByteBuf.java:446)
        at io.netty.buffer.AbstractByteBuf.writeBytes(AbstractByteBuf.java:871)
        at io.netty.channel.socket.nio.NioSocketChannel.doReadBytes(NioSocketChannel.java:208)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:119)
        at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:485)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:452)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:346)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:794)
        at java.lang.Thread.run(Unknown Source)

> 我这里问题发生原因是服务端未给客户端回应消息前客户端提前关闭连接造成。

解决方案：

    @Override
    public  void  exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
        Channel channel = ctx.channel();
        //… …/
        if(channel.isActive())ctx.close();
    }
