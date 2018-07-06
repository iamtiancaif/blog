---
layout: post
title:  "Netty 4 multiple client"
categories: computer
tags:  computer copy stackoverflow java concurrent netty
author: "web"
source: "https://stackoverflow.com/questions/12989078/netty-4-multiple-client"
---

* content
{:toc}


Question
========

I need to make the client is able to make many connections. I use Netty 4.0. Unfortunately all existing examples do not show how to create a lot of connections.

    public class TelnetClient {
        private Bootstrap b;
        public TelnetClient() {
            b = new Bootstrap();
        }
        public void connect(String host, int port) throws Exception {
            try {
                b.group(new NioEventLoopGroup()).channel(NioSocketChannel.class).remoteAddress(host, port).handler(new TelnetClientInitializer());
                Channel ch = b.connect().sync().channel();
                ChannelFuture lastWriteFuture = null;
                BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
                for (;;) {
                    String line = in.readLine();
                    if (line == null) break;
                    lastWriteFuture = ch.write(line + "\r\n");
                    if (line.toLowerCase().equals("bye")) {
                        ch.closeFuture().sync();
                        break;
                    }
                }
                if (lastWriteFuture != null) lastWriteFuture.sync();
            } finally {
                b.shutdown();
            }
        }
        public static void main(String[] args) throws Exception {
            TelnetClient tc = new TelnetClient();
            tc.connect("127.0.0.1", 1048);
            tc.connect("192.168.1.123", 1050);
        //...
        }
    }

Is this the correct decision? or could it be better?

Answer
======

Yes its almost correct.. The only thing you MUST change is the creation of NioEventLoopGroup on every connect.  

NioEventLoopGroup instances are expensive so they should be shared. Create one instance and share it, by pass the same instance to the Bootstrap.group(...) everytime..  


reference
=========

[example code](/2018-04-13-a-c10m-iot-platform#multiplexclient)   














