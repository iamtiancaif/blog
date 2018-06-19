---
layout: post
title:  "a c10m iot platform"
categories: computer
tags:  computer linux java jixianzhao question
author: JiXianzhao
---

* content
{:toc}


Benchmark
=========

qps test 1
------

**environment**  
1. 7 test machines run on docker. 1 for server, 6 others for clients. Both server and clients were based on netty.  
2. Server run in full asynchronized mode.  
	It means using callback instead of method sync in everywhere possible.
3. clients run in half asynchronized mode.   
	It means using method sync when connect and send out data; operation of handling received data was based on netty eventloop, thread count was double of cpus cores.  
	After connecting phase was done, there was only one thread run for sending ping request and other data.  
	Here is [client source code](#multiplexclient).  

**test points**  
1. max devices connect at once.  
	2660 per second = (168000-8400)/60.  
2. max ping request at once.  
	40983.6 per second = 10000/(0.579−0.335).

**conclusion**  
I thing these score wasn't the upper limit of server. Because clients hadn't simulate a real concurrent environment.

A rough estimation about memory overhead
========================================

conclusion first
----------------

28000 sockets consume 70M memory of Old Generation.

2661 Byte per socket

**simple calculation**

161316040 Bytes, before connecting,  
235846448 Bytes, after connected and holding.  
28000	  connections.  

2661.800285714 = (235846448−161316040) / 28000

## train of thought
1. Large Old Generation only get increased and never reclaimed.
2. A server do nothing but only accpect and hold connections, based on netty.
3. A client do nothing but only request and hold connections, based on netty too.
4. Boot client to connect to server with thousands of sockets, hold on a minute, then stop client.  
5. Repeat step 4 several times and record the gc state each time. 


Cause each time the calculation as **simple calculation** resulted in the nearly same value, I just present the last time I did. The following:
<!--more-->

## gc state of server before connecting
	$ jmap -heap 5440
	Attaching to process ID 5440, please wait...
	Debugger attached successfully.
	Server compiler detected.
	JVM version is 25.121-b13
	
	using thread-local object allocation.
	Parallel GC with 2 thread(s)
	
	Heap Configuration:
	   MinHeapFreeRatio         = 0
	   MaxHeapFreeRatio         = 100
	   MaxHeapSize              = 2147483648 (2048.0MB)
	   NewSize                  = 67108864 (64.0MB)
	   MaxNewSize               = 67108864 (64.0MB)
	   OldSize                  = 2080374784 (1984.0MB)
	   NewRatio                 = 2
	   SurvivorRatio            = 8
	   MetaspaceSize            = 21807104 (20.796875MB)
	   CompressedClassSpaceSize = 1073741824 (1024.0MB)
	   MaxMetaspaceSize         = 17592186044415 MB
	   G1HeapRegionSize         = 0 (0.0MB)
	
	Heap Usage:
	PS Young Generation
	Eden Space:
	   capacity = 58195968 (55.5MB)
	   used     = 46308640 (44.163360595703125MB)
	   free     = 11887328 (11.336639404296875MB)
	   79.57362269496059% used
	From Space:
	   capacity = 4718592 (4.5MB)
	   used     = 180240 (0.1718902587890625MB)
	   free     = 4538352 (4.3281097412109375MB)
	   3.8197835286458335% used
	To Space:
	   capacity = 4194304 (4.0MB)
	   used     = 0 (0.0MB)
	   free     = 4194304 (4.0MB)
	   0.0% used
	PS Old Generation
	   capacity = 2080374784 (1984.0MB)
	   used     = 161316040 (153.84296417236328MB)
	   free     = 1919058744 (1830.1570358276367MB)
	   7.754181661913472% used
	
	14720 interned Strings occupying 1409304 bytes.


## gc state of server after connectted
	$ jmap -heap 5440
	Attaching to process ID 5440, please wait...
	Debugger attached successfully.
	Server compiler detected.
	JVM version is 25.121-b13
	
	using thread-local object allocation.
	Parallel GC with 2 thread(s)
	
	Heap Configuration:
	   MinHeapFreeRatio         = 0
	   MaxHeapFreeRatio         = 100
	   MaxHeapSize              = 2147483648 (2048.0MB)
	   NewSize                  = 67108864 (64.0MB)
	   MaxNewSize               = 67108864 (64.0MB)
	   OldSize                  = 2080374784 (1984.0MB)
	   NewRatio                 = 2
	   SurvivorRatio            = 8
	   MetaspaceSize            = 21807104 (20.796875MB)
	   CompressedClassSpaceSize = 1073741824 (1024.0MB)
	   MaxMetaspaceSize         = 17592186044415 MB
	   G1HeapRegionSize         = 0 (0.0MB)
	
	Heap Usage:
	PS Young Generation
	Eden Space:
	   capacity = 58720256 (56.0MB)
	   used     = 6935384 (6.614097595214844MB)
	   free     = 51784872 (49.385902404785156MB)
	   11.81088856288365% used
	From Space:
	   capacity = 4194304 (4.0MB)
	   used     = 2326528 (2.21875MB)
	   free     = 1867776 (1.78125MB)
	   55.46875% used
	To Space:
	   capacity = 4194304 (4.0MB)
	   used     = 0 (0.0MB)
	   free     = 4194304 (4.0MB)
	   0.0% used
	PS Old Generation
	   capacity = 2080374784 (1984.0MB)
	   used     = 235846448 (224.9207000732422MB)
	   free     = 1844528336 (1759.0792999267578MB)
	   11.336728834336803% used
	
	14722 interned Strings occupying 1409440 bytes.


Questions
---------
### 1. how to beyond 29000 connections on one client
Finally I google out the answer:  
[Can't open more than 28234 sockets?](https://stackoverflow.com/questions/6244532/cant-open-more-than-28234-sockets)

>The ephemeral port range is set in:
>
>/proc/sys/net/ipv4/ip_local_port_range
>Prevents you from creating more connections.
>

and  


>The usual workaround is to create additional IP addresses on the host, each IP will gain you an additional ephemeral port range as per dan_waterworth's answer as long as you bind the socket to the interface.
>
>Microsoft have a discussion on the topic here:
>
>[http://msdn.microsoft.com/en-us/library/cc150670(v=vs.85).aspx](http://msdn.microsoft.com/en-us/library/cc150670(v=vs.85).aspx) 
 
I backup this Microsoft article on my blog at SO_PORT_SCALABILITY 


#### the original answer
The max connections I can hit is 28231 per client.  
Exactly 28231 every time.  
Here is the linux system config of client:

	$ ulimit -a
	core file size          (blocks, -c) 0
	data seg size           (kbytes, -d) unlimited
	scheduling priority             (-e) 0
	file size               (blocks, -f) unlimited
	pending signals                 (-i) 99999999
	max locked memory       (kbytes, -l) 64
	max memory size         (kbytes, -m) unlimited
	open files                      (-n) 655350
	pipe size            (512 bytes, -p) 8
	POSIX message queues     (bytes, -q) 819200
	real-time priority              (-r) 0
	stack size              (kbytes, -s) 8192
	cpu time               (seconds, -t) unlimited
	max user processes              (-u) unlimited
	virtual memory          (kbytes, -v) unlimited
	file locks                      (-x) unlimited

	$ cat /proc/sys/vm/max_map_count
	600000

	# this one seems make nonsense
	$ cat /proc/sys/kernel/pid_max
	999999


#### client java code:
##### MultiplexClient

	package com.sciov.demo.multilex.client;

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStreamReader;
	import java.util.Queue;
	import java.util.Random;
	import java.util.UUID;
	import java.util.concurrent.LinkedBlockingQueue;
	import java.util.concurrent.atomic.AtomicInteger;
	import org.apache.commons.lang3.StringUtils;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import io.netty.bootstrap.Bootstrap;
	import io.netty.buffer.ByteBuf;
	import io.netty.buffer.ByteBufAllocator;
	import io.netty.buffer.Unpooled;
	import io.netty.buffer.UnpooledByteBufAllocator;
	import io.netty.channel.Channel;
	import io.netty.channel.ChannelInitializer;
	import io.netty.channel.ChannelPipeline;
	import io.netty.channel.nio.NioEventLoopGroup;
	import io.netty.channel.socket.SocketChannel;
	import io.netty.channel.socket.nio.NioSocketChannel;
	import io.netty.handler.codec.mqtt.MqttConnectMessage;
	import io.netty.handler.codec.mqtt.MqttConnectPayload;
	import io.netty.handler.codec.mqtt.MqttConnectVariableHeader;
	import io.netty.handler.codec.mqtt.MqttDecoder;
	import io.netty.handler.codec.mqtt.MqttEncoder;
	import io.netty.handler.codec.mqtt.MqttFixedHeader;
	import io.netty.handler.codec.mqtt.MqttMessageType;
	import io.netty.handler.codec.mqtt.MqttPublishMessage;
	import io.netty.handler.codec.mqtt.MqttPublishVariableHeader;
	import io.netty.handler.codec.mqtt.MqttQoS;
	import io.netty.handler.codec.mqtt.MqttVersion;

	public class MultiplexClient implements Runnable {
		private static final Logger logger = LoggerFactory.getLogger(MultiplexClient.class);

		private static final int MAX_PAYLOAD_SIZE = 64 * 1024 * 1024;

		private static final int groupSize = Runtime.getRuntime().availableProcessors() * 2;
		private static final NioEventLoopGroup group = new NioEventLoopGroup(groupSize);

		private static final Queue<Channel> channelQueue = new LinkedBlockingQueue<>();

		private final AtomicInteger messageId = new AtomicInteger();
		private static final ByteBufAllocator ALLOCATOR = new UnpooledByteBufAllocator(false);

		public static void main(String[] args) throws Exception {
			int connectionCount = 2;
			String ipAddr = "127.0.0.1";
			int port = 1883;
			for (String arg : args) {
				if (arg.startsWith("--")) {
					int indexOfEquator = arg.indexOf("=");
					String argName = arg.substring(2, indexOfEquator);
					String value = arg.substring(indexOfEquator+1);
					logger.info("arg name {}, value {}", argName, value);

					switch (argName) {
						case "connection.count":
							try {
								connectionCount = Integer.valueOf(value);
							} catch (Exception e) {
								logger.error("failed to parse integer with string {} for count.", value);
							}
							break;
						case "connection.server":
							ipAddr = value;
							break;
						case "connection.port":
							try {
								port = Integer.valueOf(value);
							} catch (Exception e) {
								logger.error("failed to parse integer with string {} for port.", value);
							}
							break;
					}
				}
			}
			MultiplexClient tc = new MultiplexClient();
			Thread thread = new Thread(tc);
			thread.start();
			for (int i = 0; i< connectionCount; i++) {
				tc.connect(ipAddr, port);
			}
	//		Random random = new Random();
	//		while (true) {
	//			tc.mqttPublish("hello " + random.nextInt(1000000));
	//		}
		}

		public void run() {
			BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
			for (;;) {
				String line = null;
				try {
					line = in.readLine();
				} catch (IOException e) {
					e.printStackTrace();
					System.exit(-1);
				}
				if (line == null) {
					break;
				}
				Channel ch = channelQueue.poll();
				try {
					mqttPublish(ch, line);
				} catch (Throwable t) {
					t.printStackTrace();
				}
				if (ch != null) {
					channelQueue.add(ch);
				}
			}
		}

		public void connect(String host, int port) throws Exception {
			try {
				Bootstrap b = new Bootstrap();
				b.group(group).channel(NioSocketChannel.class).remoteAddress(host, port).handler(
						new ChannelInitializer<SocketChannel>() {
							@Override
							protected void initChannel(SocketChannel ch) throws Exception {
								ChannelPipeline pipeline = ch.pipeline();
								pipeline.addLast("decoder", new MqttDecoder(MAX_PAYLOAD_SIZE));
								pipeline.addLast("encoder", MqttEncoder.INSTANCE);
								pipeline.addLast("mqttHandle", new MqttTransportHandler(UUID.randomUUID().toString()));
							}
						}
				);

				// shack
				Channel ch = b.connect().sync().channel();

				mqttConnect(ch);
				channelQueue.add(ch);
			} catch (Throwable e) {
				e.printStackTrace();
			}
		}

		protected void mqttConnect(Channel ch) {
			MqttFixedHeader mqttFixedHeader =
					new MqttFixedHeader(MqttMessageType.CONNECT, false, MqttQoS.AT_LEAST_ONCE, false, 0);
			MqttConnectVariableHeader variableHeader = new MqttConnectVariableHeader(
					MqttVersion.MQTT_3_1_1.protocolName(),
					MqttVersion.MQTT_3_1_1.protocolLevel(),
					true,
					false,
					false,
					1,
					false,
					true,
					10);

			MqttConnectPayload payload = new MqttConnectPayload(
					String.valueOf(new Random().nextInt(100000000)),
					null,
					null,
					"123456",
					""
			);

			MqttConnectMessage connectMessage = new MqttConnectMessage(mqttFixedHeader, variableHeader, payload);
			ch.writeAndFlush(connectMessage);
		}

		protected void mqttPublish(String message) {
			Channel ch = channelQueue.poll();
			try {
				mqttPublish(ch, message);
			} catch (Throwable t) {
				t.printStackTrace();
			}
			if (ch != null) {
				channelQueue.add(ch);
			}
		}

		protected void mqttPublish(Channel ch, String message) {
			if (StringUtils.isBlank(message)) {
				return;
			}
			
			byte[] bytes = message.getBytes();
			ByteBuf msg = Unpooled.buffer(bytes.length);
			msg.writeBytes(bytes);

			MqttFixedHeader mqttFixedHeader =
					new MqttFixedHeader(MqttMessageType.PUBLISH, false, MqttQoS.AT_LEAST_ONCE, false, 0);
			MqttPublishVariableHeader header = new MqttPublishVariableHeader("test", messageId.incrementAndGet());

			ByteBuf payload = ALLOCATOR.buffer();
			payload.writeBytes(msg.copy());
			MqttPublishMessage publishMessage = new MqttPublishMessage(mqttFixedHeader, header, payload);
			ch.writeAndFlush(publishMessage);
		}
	}






