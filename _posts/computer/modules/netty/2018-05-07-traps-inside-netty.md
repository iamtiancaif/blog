---
layout: post
title:  "traps inside netty"
categories: computer
tags:  computer copy java netty concurrency
author: "舒润"
source: "http://www.cnblogs.com/rainy-shurun/p/5213086.html"
---

* content
{:toc}


Netty中的那些坑（上篇）
==============

最近开发了一个纯异步的redis客户端，算是比较深入的使用了一把netty。在使用过程中一边优化，一边解决各种坑。儿这些坑大部分基本上是Netty4对Netty3的改进部分引起的。

注：这里说的坑不是说netty不好，只是如果这些地方不注意，或者不去看netty的代码，就有可能掉进去了。

Netty 4的线程模型转变
--------------

在Netty 3的时候，upstream是在IO线程里执行的，而downstream是在业务线程里执行的。比如netty从网络读取一个包传递给你的handler的时候，你的handler部分的代码是执行在IO线程里，而你的业务线程调用write向网络写出一些东西的时候，你的handler是执行在业务线程里。而Netty 4修改了这一模型。在Netty 4里inbound(upstream)和outbound(downstream)都是执行在EventLoop(IO线程)里。也就是你如果在业务线程里通过channel.write向网络写出一些东西的时候，在某一点，netty 4会往这个channel的EventLoop里提交一个写出的任务。那也就是业务线程和IO线程是异步执行的。

这有什么问题呢？一般我们在网络通信里，业务层写出的都是对象。然后经过序列化等手段转换成字节流到网络，而Netty给我们提供了很好的编码解码的模型，一般我们也会将序列化和反序列化放到一个handler里处理，而在Netty 4里这些handler都是在EventLoop里执行，那么就意味着在Netty 4里下面的代码可能会导致一些微妙的结果：

	User user = new User();
	user.setName("admin");
	channel.write(user);
	user.setName("guest");

因为序列化和业务线程异步执行，那么在write执行后并不表示user对象已经序列化了，如果这个时候修改了user对象那么传递到peer的对象可能就不再是你期望的那个user了。所以在Netty 4里如果还是使用handler实现序列化就一定要小心了。你要么在调用channel.write写出之前将对象进行深度拷贝，要么就不在handler里进行序列化了，直接将序列化好的东西传递给channel。

<!--more-->

在不同的线程里使用PooledByteBufAllocator分配和回收
------------------------------------

这个问题其实是上面一个问题的续集。在碰到之前一个问题后，我们就决定不再在handler里做序列化了，而是直接在业务线程里做。但是为了减少内存的拷贝，我们就期望在序列化的时候直接将字节流序列化到DirectByteBuf里，这样通过socket写出的时候就不进行拷贝了。而DirectByteBuf的分配成本比HeapByteBuf的成本要高，为此Netty 4借鉴jemalloc的思路实现了一个PooledByteBufAllocator。顾名思义，就是将DirectByteBuf池化起来，回收的时候不真正回收，分配的时候从池里取一个空闲的。这对于大多数应用来说优化效果还是很明显的，比如在一些RPC场景中，我们所传递的对象的大小往往是差不多的，这可以充分利用池化的效果。

但是我们在使用类似下面的伪代码的时候内存占用不断飙高，然后疯狂Full GC，并且有的时候还会出现OOM。这好像是内存泄漏的迹象：

	//业务线程
	PooledByteBufAllocator allocator = PooledByteBufAllocator.DEFAULT;
	ByteBuf buffer = allocator.buffer();
	User user = new User();
	//将对象直接序列化到ByteBuf
	serialization.serialize(buffer, user);
	//进入EventLoop
	channel.writeAndFlush(buffer);

上面的代码表面看没什么问题。但实际上，PooledByteBufAllocator为了减少锁竞争，池是通过thread local来实现的。也就是分配的时候会从本线程(这里就是业务线程)的thread local里取。而channel.writeAndFlush调用后，在将buffer写到socket后，这个buffer将被回收到池里。回收的时候也是通过thread local找到对应的池，回收掉。这样就有一个问题，分配的时候是在业务线程，也就是说从业务线程的thread local对应的池里分配的，而回收的时候是在IO线程。这两个是不同的线程。池的作用完全丧失了，一个线程不断地去分配，不断地转移到另外一个池。

ByteBuf扩展引起的问题
--------------

其实这个问题和上面一个问题是一样的。但是比之前的问题更加隐晦，就在你弹冠相庆的时候给你致命一击。在碰到上面一个问题后我们就在想，既然分配和回收都得在同一个线程里执行，那我们是不是可以启动一个专门的线程来负责分配和回收呢？于是就有了下面的代码：

	import io.netty.buffer.ByteBuf;
	import io.netty.buffer.ByteBufAllocator;
	import io.netty.buffer.PooledByteBufAllocator;
	import io.netty.util.ReferenceCountUtil;
	import qunar.tc.qclient.redis.exception.RedisRuntimeException;
	import java.util.ArrayList;
	import java.util.List;
	import java.util.concurrent.ArrayBlockingQueue;
	import java.util.concurrent.BlockingQueue;
	import java.util.concurrent.LinkedBlockingQueue;
	public class Allocator {
		public static final ByteBufAllocator allocator = PooledByteBufAllocator.DEFAULT;

		private static final BlockingQueue<ByteBuf> bufferQueue = new ArrayBlockingQueue<ByteBuf>(100);

		private static final BlockingQueue<ByteBuf> toCleanQueue = new LinkedBlockingQueue<ByteBuf>();

		private static final int TO_CLEAN_SIZE =50;

		private static final long CLEAN_PERIOD = 100;

		private static class AllocThread implements Runnable {

			@Override

			public void run() {

				long lastCleanTime = System.currentTimeMillis();

				while (!Thread.currentThread().isInterrupted()) {

					try {

						ByteBuf buffer = allocator.buffer();

						//确保是本线程释放

						buffer.retain();

						bufferQueue.put(buffer);

					} catch (InterruptedException e) {

						Thread.currentThread().interrupt();

					}

					if (toCleanQueue.size() > TO\_CLEAN\
					_SIZE || System.currentTimeMillis() - lastCleanTime > CLEAN_PERIOD){

						final List<ByteBuf> toClean = new ArrayList<ByteBuf>(toCleanQueue.size());

						toCleanQueue.drainTo(toClean);

						for (ByteBuf buffer : toClean) {

							ReferenceCountUtil.release(buffer);

						}

						lastCleanTime = System.currentTimeMillis();

					}

				}

			}

		}

		static {

			Thread thread = new Thread(new AllocThread(), "qclient-redis-allocator");

			thread.setDaemon(true);

			thread.start();

		}

		public static ByteBuf alloc() {

			try {

				return bufferQueue.take();

			} catch (InterruptedException e) {

				Thread.currentThread().interrupt();

				throw new RedisRuntimeException("alloc interrupt");

			}

		}

		public static void release(ByteBuf buf) {

			toCleanQueue.add(buf);

		}

	}

在业务线程里调用alloc，从queue里拿到专用的线程分配好的buffer。在将buffer写出到socket之后再调用release回收：

	//业务线程	
	ByteBuf buffer=Allocator.alloc();
	//序列化

	//	........
	
	//写出

	ChannelPromise promise=channel.newPromise();
	promise.addListener(new GenericFutureListener<Future<Void>>(){
		@Override   	
		public void operationComplete(Future<Void> future)throws Exception{
			//buffer已经输出，可以回收，交给专用线程回收                        	
			Allocator.release(buffer);
		}
	});

	//进入EventLoop
	channel.write(buffer,promise);




好像问题解决了。而且我们通过压测发现性能果然有提升，内存占用也很正常，通过写出各种不同大小的buffer进行了几番测试结果都很OK。

不过你如果再提高每次写出包的大小的时候，问题就出现了。在我这个版本的netty里，ByteBufAllocator.buffer()分配的buffer默认大小是256个字节，当你将对象往这个buffer里序列化的时候，如果超过了256个字节ByteBuf就会自动扩展，而对于PooledByteBuf来说，自动扩展是会去池里取一个，然后将旧的回收掉。而这一切都是在业务线程里进行的。意味着你使用专用的线程来做分配和回收功亏一篑。

上面三个问题就好像冥冥之中，有一双看不见的手将你一步一步带入深渊，最后让你绝望。一个问题引出一个必然的解决方案，而这个解决方案看起来将问题解决了，但却是将问题隐藏地更深。

如果说前面三个问题是因为你不熟悉Netty的新机制造成的，那么下面这个问题我觉得就是Netty本身的API设计不合理导致使用的人出现这个问题了。

连接超时
----

在网络应用中，超时往往是最后一道防线，或是最后一根稻草。我们不怕干脆利索的宕机，怕就怕要死不活。当碰到要死不活的应用的时候往往就是依靠超时了。

在使用Netty编写客户端的时候，我们一般会有类似这样的代码：

	bootstrap.connect(address).await(1000, TimeUnit.MILLISECONDS)

向对端发起一个连接，超时等待1秒钟。如果1秒钟没有连接上则重连或者做其他处理。而其实在bootstrap的选项里，还有这样的一项：

	bootstrap.option(ChannelOption.CONNECT\_TIMEOUT\_MILLIS, 3000);

如果这两个值设置的不一致，在await的时候较短，而option里设置的较长就出问题了。这个时候你会发现connect里已经超时了，你以为连接失败了，但实际上await超时Netty并不会帮你取消正在连接的链接。这个时候如果第2秒的时候连上了对端服务器，那么你刚才的判断就失误了。如果你根据connect(address).await(1000, TimeUnit.MILLISECONDS)来决定是否重连，很有可能你就建立了两个连接，而且很有可能你的handler就在这两个channel里共享起来了，这就有可能让你产生：哎呀，Netty的handler不是在单线程里执行的这样的假象。所以我的建议是，不要在await上设置超时，而总是使用option上的选项来设置。这个更准确些，超时了就是真的表示没有连上。

异步处理，流控先行
---------

这个坑其实也不算坑，只是因为懒，该做的事情没做。一般来讲我们的业务如果比较小的时候我们用同步处理，等业务到一定规模的时候，一个优化手段就是异步化。异步化是提高吞吐量的一个很好的手段。但是，与异步相比，同步有天然的负反馈机制，也就是如果后端慢了，前面也会跟着慢起来，可以自动的调节。但是异步就不同了，异步就像决堤的大坝一样，洪水是畅通无阻。如果这个时候没有进行有效的限流措施就很容易把后端冲垮。如果一下子把后端冲垮倒也不是最坏的情况，就怕把后端冲的要死不活。这个时候，后端就会变得特别缓慢，如果这个时候前面的应用使用了一些无界的资源等，就有可能把自己弄死。那么现在要介绍的这个坑就是关于Netty里的ChannelOutboundBuffer这个东西的。这个buffer是用在netty向channel write数据的时候，有个buffer缓冲，这样可以提高网络的吞吐量(每个channel有一个这样的buffer)。初始大小是32(32个元素，不是指字节)，但是如果超过32就会翻倍，一直增长。大部分时候是没有什么问题的，但是在碰到对端非常慢(对端慢指的是对端处理TCP包的速度变慢，比如对端负载特别高的时候就有可能是这个情况)的时候就有问题了，这个时候如果还是不断地写数据，这个buffer就会不断地增长，最后就有可能出问题了(我们的情况是开始吃swap，最后进程被linux killer干掉了)。

为什么说这个地方是坑呢，因为大部分时候我们往一个channel写数据会判断channel是否active，但是往往忽略了这种慢的情况。

那这个问题怎么解决呢？其实ChannelOutboundBuffer虽然无界，但是可以给它配置一个高水位线和低水位线，当buffer的大小超过高水位线的时候对应channel的isWritable就会变成false，当buffer的大小低于低水位线的时候，isWritable就会变成true。所以应用应该判断isWritable，如果是false就不要再写数据了。高水位线和低水位线是字节数，默认高水位是64K，低水位是32K，我们可以根据我们的应用需要支持多少连接数和系统资源进行合理规划。

	.option(ChannelOption.WRITE\_BUFFER\_HIGH\_WATER\_MARK, 64 * 1024)  
	.option(ChannelOption.WRITE\_BUFFER\_LOW\_WATER\_MARK, 32 * 1024)

在使用一些开源的框架上还真是要熟悉人家的实现机制，然后才可以大胆的使用啊，不然被坑死都觉得自己很冤枉

Netty中的坑(下篇)
============

其实这篇应该叫Netty实践，但是为了与前一篇名字保持一致，所以还是用一下坑这个名字吧。

Netty是高性能Java NIO网络框架，在很多开源系统里都有她的身影，而在绝大多数互联网公司所实施的服务化，以及最近流行的MicroService中，她都作为基础中的基础出现。

Netty的出现让我们可以简单容易地就可以使用NIO带来的高性能网络编程的潜力。她用一种统一的流水线方式组织我们的业务代码，将底层网络繁杂的细节隐藏起来，让我们只需要关注业务代码即可。并且用这种机制将不同的业务划分到不同的handler里，比如将编码，连接管理，业务逻辑处理进行分开。Netty也力所能及的屏蔽了一些NIO bug，比如著名的epoll cpu 100% bug。而且，还提供了很多优化支持，比如使用buffer来提高网络吞吐量。

但是，和所有的框架一样，框架为我们屏蔽了底层细节，让我们可以很快上手。但是，并不表示我们不需要对框架所屏蔽的那一层进行了解。本文所涉及的几个地方就是Netty与底层网络结合的几个地方，看看我们使用的时候应该怎么处理，以及为什么要这么处理。

autoread
--------

在Netty 4里我觉得一个很有用的功能是autoread。autoread是一个开关，如果打开的时候Netty就会帮我们注册读事件(这个需要对NIO有些基本的了解)。当注册了读事件后，如果网络可读，则Netty就会从channel读取数据，然后我们的pipeline就会开始流动起来。那如果autoread关掉后，则Netty会不注册读事件，这样即使是对端发送数据过来了也不会触发读时间，从而也不会从channel读取到数据。那么这样一个功能到底有什么作用呢？

它的作用就是更精确的速率控制。那么这句话是什么意思呢？比如我们现在在使用Netty开发一个应用，这个应用从网络上发送过来的数据量非常大，大到有时我们都有点处理不过来了。而我们使用Netty开发应用往往是这样的安排方式：Netty的Worker线程处理网络事件，比如读取和写入，然后将读取后的数据交给pipeline处理，比如经过反序列化等最后到业务层。到业务层的时候如果业务层有阻塞操作，比如数据库IO等，可能还要将收到的数据交给另外一个线程池处理。因为我们绝对不能阻塞Worker线程，一旦阻塞就会影响网络处理效率，因为这些Worker是所有网络处理共享的，如果这里阻塞了，可能影响很多channel的网络处理。

但是，如果把接到的数据交给另外一个线程池处理就又涉及另外一个问题：速率匹配。

比如现在网络实在太忙了，接收到很多数据交给线程池。然后就出现两种情况：

1\. 由于开发的时候没有考虑到，这个线程池使用了某些无界资源。比如很多人对ThreadPoolExecutor的几个参数不是特别熟悉，就有可能用错，最后导致资源无节制使用，整个系统crash掉。

	//比如开始的时候没有考虑到会有这么大量
	//这种方式线程数是无界的，那么有可能创建大量的线程对系统稳定性造成影响
	Executor executor = Executors.newCachedTheadPool(); 
	executor.execute(requestWorker);
	//或者使用这个
	//这种queue是无界的，有可能会消耗太多内存，对系统稳定性造成影响
	Executor executor = Executors.newFixedThreadPool(8); 
	executor.execute(requestWorker);

2\. 第二种情况就是限制了资源使用，所以只好把最老的或最新的数据丢弃。

	//线程池满后，将最老的数据丢弃
	Executor executor = new ThreadPoolExecutor(8, 8, 1, TimeUnit.MINUTES, 
				new ArrayBlockingQueue<Runnable>(1000), namedFactory, new ThreadPoolExecutor.DiscardOldestPolicy());

其实上面两种情况，不管哪一种都不是太合理。不过在Netty 4里我们就有了更好的解决办法了。如果我们的线程池暂时处理不过来，那么我们可以将autoread关闭，这样Netty就不再从channel上读取数据了。那么这样造成的影响是什么呢？这样socket在内核那一层的read buffer就会满了。因为TCP默认就是带flow control的，read buffer变小之后，向对端发送ACK的时候，就会降低窗口大小，直至变成0，这样对端就会自动的降低发送数据的速率了。等到我们又可以处理数据了，我们就可以将autoread又打开这样数据又源源不断的到来了。

这样整个系统就通过TCP的这个负反馈机制，和谐的运行着。那么autoread涉及的网络知识就是，发送端会根据对端ACK时候所携带的advertises window来调整自己发送的数据量。而ACK里的这个window的大小又跟接收端的read buffer有关系。而不注册读事件后，read buffer里的数据没有被消费掉，就会达到控制发送端速度的目的。

不过设计关闭和打开autoread的策略也要注意，不要设计成我们不能处理任何数据了就立即关闭autoread，而我们开始能处理了就立即打开autoread。这个地方应该留一个缓冲地带。也就是如果现在排队的数据达到我们预设置的一个高水位线的时候我们关闭autoread，而低于一个低水位线的时候才打开autoread。不这么弄的话，有可能就会导致我们的autoread频繁打开和关闭。autoread的每次调整都会涉及系统调用，对性能是有影响的。类似下面这样一个代码，在将任务提交到线程池之前，判断一下现在的排队量(注：本文的所有数字纯为演示作用，所有线程池，队列等大小数据要根据实际业务场景仔细设计和考量)。

	int highReadWaterMarker = 900;
	int lowReadWaterMarker = 600;
	ThreadPoolExecutor executor = new ThreadPoolExecutor(8, 8, 1, TimeUnit.MINUTES, new ArrayBlockingQueue<Runnable>(1000),
					namedFactory, new ThreadPoolExecutor.DiscardOldestPolicy());
	int queued = executor.getQueue().size();
	if (queued > highReadWaterMarker) {
		channel.config().setAutoRead(false);
	}
	if (queued < lowReadWaterMarker) {
		channel.config().setAutoRead(true);
	}

但是使用autoread也要注意一件事情。autoread如果关闭后，对端发送FIN的时候，接收端应用层也是感知不到的。这样带来一个后果就是对端发送了FIN，然后内核将这个socket的状态变成CLOSE\_WAIT。但是因为应用层感知不到，所以应用层一直没有调用close。这样的socket就会长期处于CLOSE\_WAIT状态。特别是一些使用连接池的应用，如果将连接归还给连接池后，一定要记着autoread一定是打开的。不然就会有大量的连接处于CLOSE_WAIT状态。

其实所有异步的场合都存在速率匹配的问题，而同步往往不存在这样的问题，因为同步本身就是带负反馈的。

isWritable
----------

isWritable其实在上一篇文章已经介绍了一点，不过这里我想结合网络层再啰嗦一下。上面我们讲的autoread一般是接收端的事情，而发送端也有速率控制的问题。Netty为了提高网络的吞吐量，在业务层与socket之间又增加了一个ChannelOutboundBuffer。在我们调用channel.write的时候，所有写出的数据其实并没有写到socket，而是先写到ChannelOutboundBuffer。当调用channel.flush的时候才真正的向socket写出。因为这中间有一个buffer，就存在速率匹配了，而且这个buffer还是无界的。也就是你如果没有控制channel.write的速度，会有大量的数据在这个buffer里堆积，而且如果碰到socket又『写不出』数据的时候，很有可能的结果就是资源耗尽。而且这里让这个事情更严重的是ChannelOutboundBuffer很多时候我们放到里面的是DirectByteBuffer，什么意思呢，意思是这些内存是放在GC Heap之外。如果我们仅仅是监控GC的话还监控不出来这个隐患。

那么说到这里，socket什么时候会写不出数据呢？在上一节我们了解到接收端有一个read buffer，其实发送端也有一个send buffer。我们调用socket的write的时候其实是向这个send buffer写数据，如果写进去了就表示成功了(所以这里千万不能将socket.write调用成功理解成数据已经到达接收端了)，如果send buffer满了，对于同步socket来讲，write就会阻塞直到超时或者send buffer又有空间(这么一看，其实我们可以将同步的socket.write理解为半同步嘛)。对于异步来讲这里是立即返回的。

那么进入send buffer的数据什么时候会减少呢？是发送到网络的数据就会从send buffer里去掉么？也不是这个样子的。还记得TCP有重传机制么，如果发送到网络的数据都从send buffer删除了，那么这个数据没有得到确认TCP怎么重传呢？所以send buffer的数据是等到接收端回复ACK确认后才删除。那么，如果接收端非常慢，比如CPU占用已经到100%了，而load也非常高的时候，很有可能来不及处理网络事件，这个时候send buffer就有可能会堆满。这就导致socket写不出数据了。而发送端的应用层在发送数据的时候往往判断socket是不是有效的(是否已经断开)，而忽略了是否可写，这个时候有可能就还一个劲的写数据，最后导致ChannelOutboundBuffer膨胀，造成系统不稳定。

所以，Netty已经为我们考虑了这点。channel有一个isWritable属性，可以来控制ChannelOutboundBuffer，不让其无限制膨胀。至于isWritable的实现机制可以参考前一篇。

序列化
---

所有讲TCP的书都会有这么一个介绍：TCP provides a connection-oriented, reliable, byte stream service。前面两个这里就不关心了，那么这个byte stream到底是什么意思呢？我们在发送端发送数据的时候，对于应用层来说我们发送的是一个个对象，然后序列化成一个个字节数组，但无论怎样，我们发送的是一个个『包』。每个都是独立的。那么接收端是不是也像发送端一样，接收到一个个独立的『包』呢？很遗憾，不是的。这就是byte stream的意思。接收端没有『包』的概念了。

这对于应用层编码的人员来说可能有点困惑。比如我使用Netty开发，我的handler的channelRead这次明明传递给我的是一个ByteBuf啊，是一个『独立』的包啊，如果是byte stream的话难道不应该传递我一个Stream么。但是这个ByteBuf和发送端的ByteBuf一点关系都没有。比如：

	public class Decorder extends ChannelInboundHandlerAdapter {
		@Override
		public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
			//这里的msg和发送端channel.write(msg)时候的msg没有任何关系
	
		}
	
	}

这个ByteBuf可能包含发送端多个ByteBuf，也可能只包含发送端半个ByteBuf。但是别担心，TCP的可靠性会确保接收端的顺序和发送端的顺序是一致的。这样的byte stream协议对我们的反序列化工作就带来了一些挑战。在反序列化的时候我们要时刻记着这一点。对于半个ByteBuf我们按照设计的协议如果解不出一个完整对象，我们要留着，和下次收到的ByteBuf拼凑在一起再次解析，而收到的多个ByteBuf我们要根据协议解析出多个完整对象，而很有可能最后一个也是不完整的。不过幸运的是，我们有了Netty。Netty为我们已经提供了很多种协议解析的方式，并且对于这种半包粘包也已经有考虑，我们可以参考ByteToMessageDecoder以及它的一连串子类来实现自己的反序列化机制。而在反序列化的时候我们可能经常要取ByteBuf中的一个片段，这个时候建议使用ByteBuf的readSlice方法而不是使用copy。

另外，Netty还提供了两个ByteBuf的流封装：ByteBufInputStream, ByteBufOutputStream。比如我们在使用一些序列化工具，比如Hessian之类的时候，我们往往需要传递一个InputStream(反序列化)，OutputStream(序列化)到这些工具。而很多协议的实现都涉及大量的内存copy。比如对于反序列化，先将ByteBuf里的数据读取到byte\[\]，然后包装成ByteArrayInputStream，而序列化的时候是先将对象序列化成ByteArrayOutputStream再copy到ByteBuf。而使用ByteBufInputStream和ByteBufOutputStream就不再有这样的内存拷贝了，大大节约了内存开销。

另外，因为socket.write和socket.read都需要一个direct byte buffer(即使你传入的是一个heap byte buffer，socket内部也会将内容copy到direct byte buffer)。如果我们直接使用ByteBufInputStream和ByteBufOutputStream封装的direct byte buffer再加上Netty 4的内存池，那么内存将更有效的使用。这里提一个问题：为什么socket.read和socket.write都需要direct byte buffer呢？heap byte buffer不行么？

总结起来，对于序列化和反序列化来讲就是两条：  
1 减少内存拷贝  
2 处理好TCP的粘包和半包问题  

后记
--

作为一个应用层程序员，往往是幸福的。因为我们有丰富的框架和工具为我们屏蔽下层的细节，这样我们可以更容易的解决很多业务问题。但是目前程序设计并没有发展到不需要了解所有下层的知识就可以写出更有效率的程序，所以我们在使用一个框架的时候最好要对它所屏蔽和所依赖的知识进行一些了解，这样在碰到一些问题的时候我们可以根据这些理论知识去分析原因。这就是理论和实践的相结合。



















