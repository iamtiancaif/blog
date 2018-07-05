The Rox Java NIO Tutorial

---
layout: post
title: "The Rox Java NIO Tutorial"
categories: computer
tags:  computer copy java concurrent nio
author: "web"
source: "http://rox-xmlrpc.sourceforge.net/niotut/"
---

* content
{:toc}

Introduction
------------

This tutorial is intended to collect together my own experiences using the Java NIO libraries and the dozens of hints, tips, suggestions and caveats that litter the Internet. When I wrote [Rox](http://rox-xmlrpc.sourceforge.net/) all of the useful information existed as just that: hints, tips, suggestions and caveats on a handful of forums. This tutorial actually only covers using NIO for asynchronous networking (non-blocking sockets), and not the NIO libraries in all their glory. When I use the term NIO in this tutorial I'm taking liberties and only talking about the non-blocking IO part of the API.

If you've spent any time looking at the JavaDoc documentation for the NIO libraries then you know they're not the most transparent docs floating around. And if you've spent any time trying to write code based on the NIO libraries and you use more than one platform you've probably run into something that works on one platform but locks up on another. This is particularly true if you started on Windows and moved over to Linux (using Sun's implementation).

This tutorial takes you from nothing to a working client-server implementation, and will hopefully help you avoid all of the pitfalls waiting to trap the unwary. I've turned it around and start with a server implementation, since that's the most common use-case for the NIO libraries.

Comments, criticisms, suggestions and, most important, corrections are welcome. The examples here have been developed and tested on Sun's NIO implementation using version 1.4.2 and 1.5.0 of Sun's Hotspot JVM on Windows and Linux. Your mileage may vary but if you do run into funnies please let me know and I'll incorporate them into this page.

Drop me a line at [nio@flat502.com](mailto:nio@flat502.com).

Credits
-------

Credit where credit is due. This tutorial pulls together a lot of ideas, none of which I can claim original ownership. Sources vary widely and I haven't managed to keep track of all of them, but an incomplete list of some of the larger contributors would include:

*   [The Java Developers Almanac](http://javaalmanac.com/egs/java.nio/pkg.html)
*   [Rob Grzywinski's thoroughly entertaining rant about getting SSL and the NIO libraries to play together on Java 1.4](http://www.realityinteractive.com/rgrzywinski/archives/000067.html)
*   [This post in the Java Forums regarding SSL and NIO on 1.4](http://forum.java.sun.com/thread.jspa?threadID=592674&tstart=0)
*   [The "Taming the NIO Circus" thread in the Java Forums](http://forums.java.sun.com/thread.jspa?threadID=459338&messageID=3327526)
*   [A discussion in the Java Forums on multithreaded access to NIO](http://forum.java.sun.com/thread.jspa?threadID=353135&messageID=1468837)
*   Half a dozen other posts in various forums and on various blogs

Source source code in this tutorial was generated from Java source using [Java2Html](http://www.java2html.de/).

General principles
------------------

A few general principles inform the approach I've taken. There may be other approaches (I've certainly seen a few other suggestions) but I know this one works, and I know it performs. Sometimes it's worth spending time finding the best possible approach. Sometimes it's enough to find an approach that works.

These general ideas apply on both the client and server side. In fact, given that the only difference between the client and the server is whether or not a connection is initiated or accepted, the bulk of the logic for using NIO for a comms implementation can be factored out and shared.

So, without further ado:

### Use a single selecting thread

Although NIO selectors are threadsafe their key sets are not. The upshot of this is that if you try to build a solution that depends on multiple threads accessing your selector you very quickly end up in one of two situations:

1.  Plagued by deadlocks and race conditions as you build up an increasingly fragile house of cards (or rather house of locks) to avoid stepping on your own toes while accessing the selector and its key sets.
2.  Effectively single-threading access to the selector and its key sets in an attempt to avoid, well, stepping on your own toes.

The upshot of this is that if you want to build a solution based on NIO then trust me and stick to a single selecting thread. Offload events as you see fit but stick to one thread on the selector.

I tend to handle all I/O within the selecting thread too. This means queuing writes and having the selecting thread perform the actual I/O, and having the selecting thread read off ready channels and offload the read data to a worker thread. In general I've found this scales well enough so I've not yet had a need to have other threads perform the I/O on the side.

### Modify the selector from the selecting thread only

If you look closely at the NIO documentation you'll come across the occasional mention of naive implementations blocking where more efficient implementations might not, usually in the context of altering the state of the selector from another thread. If you plan on writing code against the NIO libraries that must run on multiple platforms you have to assume the worst. This isn't just hypothetical either, a little experimentation should be enough to convince you that Sun's Linux implementation is "naive". If you plan on targeting one platform only feel free to ignore this advice but I'd recommend against it. The thing about code is that it oftens ends up in the oddest of places.

As a result, if you plan to hang onto your sanity don't modify the selector from any thread other than the selecting thread. This includes modifying the interest ops set for a selection key, registering new channels with the selector, and cancelling existing channels.

A number of these changes are naturally initiated by threads that aren't the selecting thread. Think about sending data. This pretty much always has to be initiated by a calling thread that isn't the selecting thread. Don't try to modify the selector (or a selection key) from the calling thread. Queue the operation where the selecting thread can get to it and call `Selector.wakeup`. This will wake the selecting thread up. All it needs to do is check if there are any pending changes, and if there are apply them and go back to selecting on the selector. There are variations on this but that's the general idea.

### Set OP_WRITE only when you have data ready

A common mistake is to enable OP\_WRITE on a selection key and leave it set. This results in the selecting thread spinning because 99% of the time a socket channel is ready for writing. In fact the only times it's not going to be ready for writing is during connection establishment or if the local OS socket buffer is full. The correct way to do this is to enable OP\_WRITE only when you have data ready to be written on that socket channel. And don't forget to do it from within the selecting thread.

### Alternate between OP\_READ and OP\_WRITE

If you try to mix OP\_READ and OP\_WRITE you'll quickly get yourself into trouble. The Sun Windows implementation has been seen to deadlock if you do this. Other implementations may fare better, but I prefer to play it safe and alternate between OP\_READ and OP\_WRITE, rather than trying to use them together.

With those out of the way, let's take a look at some actual code. The code presented here could do with a little cleaning up. I've simplified a few things in an attempt to stick to the core issue, using the NIO libraries, without getting bogged down in too many abstractions or design discussions.

The server
----------

### A starting point

We need a little infrastructure before we can start building a NIO server. First, we'll need a thread. This will be our selecting thread and most of our logic will live in the class that is home to it. And I'm going to have the selecting thread perform reads itself, so we'll need a ByteBuffer to read into. I'm going to use a non-direct buffer rather than a direct buffer. The performance difference is minor in this case and the code is slightly clearer. We're also going to need a selector and a server socket channel on which to accept connections. Throwing in a few other minor odds and ends we end up with something like this.

**public class **NioServer **implements **Runnable {  
// The host:port combination to listen on  
**private **InetAddress hostAddress;  
**private ****int **port;  
  
// The channel on which we'll accept connections  
**private **ServerSocketChannel serverChannel;  
  
// The selector we'll be monitoring  
**private **Selector selector;  
  
// The buffer into which we'll read data when it's available  
**private **ByteBuffer readBuffer = ByteBuffer.allocate(8192);  
  
**public **NioServer(InetAddress hostAddress, **int **port) **throws **IOException {  
this.hostAddress = hostAddress;  
this.port = port;  
this.selector = this.initSelector();  
}  
  
**public static ****void **main(String\[\] args) {  
**try **{  
**new **Thread(**new **NioServer(null, 9090)).start();  
} **catch **(IOException e) {  
e.printStackTrace();  
}  
}  
}

### Creating the selector and server channel

The astute will have noticed the call to `initSelector()` in the constructor. Needless to say this method doesn't exist yet. So let's write it. It's job is to create and initialize a non-blocking server channel and a selector. It must and then register the server channel with that selector.

	private Selector initSelector() throws IOException {  
		// Create a new selector  
		Selector socketSelector = SelectorProvider.provider().openSelector();  
		  
		// Create a new non-blocking server socket channel  
		this.serverChannel = ServerSocketChannel.open();  
		serverChannel.configureBlocking(false);  
		  
		// Bind the server socket to the specified address and port  
		InetSocketAddress isa = new InetSocketAddress(this.hostAddress, this.port);  
		serverChannel.socket().bind(isa);  
		  
		// Register the server socket channel, indicating an interest in   
		// accepting new connections  
		serverChannel.register(socketSelector, SelectionKey.OP_ACCEPT);  
		  
		**return **socketSelector;  
	}

### Accepting connections

Right. At this point we have a server socket channel ready and waiting and we've indicated that we'd like to know when a new connection is available to be accepted. Now we need to actually accept it. Which brings us to our "select loop". This is where most of the action kicks off. In a nutshell our selecting thread sits in a loop waiting until one of the channels registered with the selector is in a state that matches the interest operations we've registered for it. In this case the operation we're waiting for on the server socket channel is an accept (indicated by OP_ACCEPT). So let's take a look at the first iteration (I couldn't resist) of our `run()` method.

	public void run() {  
		while (true) {  
			try {  
				// Wait for an event one of the registered channels  
				this.selector.select();  
				  
				// Iterate over the set of keys for which events are available  
				Iterator selectedKeys = this.selector.selectedKeys().iterator();  
				while (selectedKeys.hasNext()) {
					SelectionKey key = (SelectionKey) selectedKeys.next();  
					selectedKeys.remove();  
					  
					if (!key.isValid()) {  
						continue;  
					}  
					  
					// Check what event is available and deal with it  
					if (key.isAcceptable()) {  
						this.accept(key);  
					}  
				}  
			} catch (Exception e) {  
				e.printStackTrace();  
			}  
		}  
	}

This only takes us half way though. We still need to accept connections. Which means, you guessed it, an `accept()` method.

**private ****void **accept(SelectionKey key) **throws **IOException {  
// For an accept to be pending the channel must be a server socket channel.  
ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();  
  
// Accept the connection and make it non-blocking  
SocketChannel socketChannel = serverSocketChannel.accept();  
Socket socket = socketChannel.socket();  
socketChannel.configureBlocking(**false**);  
  
// Register the new SocketChannel with our Selector, indicating  
// we'd like to be notified when there's data waiting to be read  
socketChannel.register(this.selector, SelectionKey.OP_READ);  
}

### Reading data

Once we've accepted a connection it's only of any use if we can read (or write) data on it. If you look back at the `accept()` method you'll notice that the newly accepted socket channel was registered with our selector with OP_READ specified at the operation we're interested in. That means our selecting thread will be released from the call to `select()` when data becomes available on the socket channel. But what do we do with it? Well, the first thing we need is a small change to our `run()`method. We need to check if a socket channel is readable (and deal with it if it is).

// Check what event is available and deal with it  
**if **(key.isAcceptable()) {  
this.accept(key);  
} **else if **(key.isReadable()) {  
this.read(key);  
}

Which means we need a `read()` method...

**private ****void **read(SelectionKey key) **throws **IOException {  
SocketChannel socketChannel = (SocketChannel) key.channel();  
  
// Clear out our read buffer so it's ready for new data  
this.readBuffer.clear();  
  
// Attempt to read off the channel  
**int **numRead;  
**try **{  
numRead = socketChannel.read(this.readBuffer);  
} **catch **(IOException e) {  
// The remote forcibly closed the connection, cancel  
// the selection key and close the channel.  
key.cancel();  
socketChannel.close();  
**return**;  
}  
  
**if **(numRead == -1) {  
// Remote entity shut the socket down cleanly. Do the  
// same from our end and cancel the channel.  
key.channel().close();  
key.cancel();  
**return**;  
}  
  
// Hand the data off to our worker thread  
this.worker.processData(this, socketChannel, this.readBuffer.array(), numRead);   
}

Hang on, where did `worker` come from? This is the worker thread we hand data off to once we've read it. For our purposes all we'll do is echo that data back. We'll take a look at the code for the worker shortly. However, assuming the existence of an `EchoWorker` class, our infrastructure needs a little updating. We need an extra instance member, a change to our constructor and one or two extras in our `main()` method.

**private **EchoWorker worker;  
  
**public **NioServer(InetAddress hostAddress, **int **port, EchoWorker worker) **throws **IOException {  
this.hostAddress = hostAddress;  
this.port = port;  
this.selector = this.initSelector();  
this.worker = worker;  
}   
  
**public static ****void **main(String\[\] args) {  
**try **{  
EchoWorker worker = **new **EchoWorker();  
**new **Thread(worker).start();  
**new **Thread(**new **NioServer(null, 9090, worker)).start();  
} **catch **(IOException e) {  
e.printStackTrace();  
}  
}

Let's take a closer look at `EchoWorker`. Our echo worker implementation is implemented as a second thread. For the purposes of this example we're only going to have a single worker thread. This means we can hand events directly to it. We'll do this by calling a method on the instance. This method will stash the event in a local queue and notify the worker thread that there's work available. Needless to say, the worker thread itself will sit in a loop waiting for events to arrive and when they do it will process them. Events in this case are just data packets. All we're going to do is echo those packets back to the sender.

Normally, instead of calling a method on the worker directly, you'll want to use a form of blocking queue that you can share with as many workers as you like. Java 1.5's concurrency package introduces a `BlockingQueue` interface and some implementations, but it's not that hard to roll your own. The logic in the `EchoWorker` code below is a good start.

**public class **EchoWorker **implements **Runnable {  
**private **List queue = **new **LinkedList();  
  
**public ****void **processData(NioServer server, SocketChannel socket, **byte**\[\] data, **int **count) {  
**byte**\[\] dataCopy = **new ****byte**\[count\];  
System.arraycopy(data, 0, dataCopy, 0, count);  
**synchronized**(queue) {  
queue.add(**new **ServerDataEvent(server, socket, dataCopy));  
queue.notify();  
}  
}  
  
**public ****void **run() {  
ServerDataEvent dataEvent;  
  
**while**(**true**) {  
// Wait for data to become available  
**synchronized**(queue) {  
**while**(queue.isEmpty()) {  
**try **{  
queue.wait();  
} **catch **(InterruptedException e) {  
}  
}  
dataEvent = (ServerDataEvent) queue.remove(0);  
}  
  
// Return to sender  
dataEvent.server.send(dataEvent.socket, dataEvent.data);  
}  
}  
}

If you look at the last line of the `run()` method you'll see we're calling a new method on the server. Let's take a look at what we need to do to send data back to a client.

### Writing data

Writing data to a socket channel is pretty straightforward. However, before we can do that we need to know that the channel is ready for more data. Which means setting the OP_WRITE interest op flag on that channel's selection key. Since the thread that wants to do this is not the selecting thread we need to queue this request somewhere and wake the selecting thread up. Putting all of this together we need a `write()` method and a few additional members in our server class.

// A list of ChangeRequest instances  
**private **List changeRequests = **new **LinkedList();  
  
// Maps a SocketChannel to a list of ByteBuffer instances  
**private **Map pendingData = **new **HashMap();  
  
**public ****void **send(SocketChannel socket, **byte**\[\] data) {  
**synchronized **(this.changeRequests) {  
// Indicate we want the interest ops set changed  
this.changeRequests.add(**new **ChangeRequest(socket, ChangeRequest.CHANGEOPS, SelectionKey.OP_WRITE));  
  
// And queue the data we want written  
**synchronized **(this.pendingData) {  
List queue = (List) this.pendingData.get(socket);  
**if **(queue == **null**) {  
queue = **new **ArrayList();  
this.pendingData.put(socket, queue);  
}  
queue.add(ByteBuffer.wrap(data));  
}  
}  
  
// Finally, wake up our selecting thread so it can make the required changes  
this.selector.wakeup();  
}

We've introduced another class: `ChangeRequest`. There's no magic here, it just gives us an easy way to indicate a change that we want made on a particular socket channel. I've jumped ahead a little with this class, in anticipation of some of the other changes we'll ultimately want to queue.

**public class **ChangeRequest {  
**public static final ****int **REGISTER = 1;  
**public static final ****int **CHANGEOPS = 2;  
  
**public **SocketChannel socket;  
**public ****int **type;  
**public ****int **ops;  
  
**public **ChangeRequest(SocketChannel socket, **int **type, **int **ops) {  
this.socket = socket;  
this.type = type;  
this.ops = ops;  
}  
}

Waking the selecting thread up is of no use until we modify our selecting thread's logic to do what needs to be done. So let's update our `run()` method.

**public ****void **run() {  
**while **(**true**) {  
**try **{  
// Process any pending changes  
**synchronized**(this.changeRequests) {  
Iterator changes = this.changeRequests.iterator();  
**while **(changes.hasNext()) {  
ChangeRequest change = (ChangeRequest) changes.next();  
**switch**(change.type) {  
**case **ChangeRequest.CHANGEOPS:  
SelectionKey key = change.socket.keyFor(this.selector);  
key.interestOps(change.ops);  
}  
}  
this.changeRequests.clear();   
}  
  
// Wait for an event one of the registered channels  
this.selector.select();  
  
// Iterate over the set of keys for which events are available  
Iterator selectedKeys = this.selector.selectedKeys().iterator();  
**while **(selectedKeys.hasNext()) {  
SelectionKey key = (SelectionKey) selectedKeys.next();  
selectedKeys.remove();  
  
**if **(!key.isValid()) {  
**continue**;  
}  
  
// Check what event is available and deal with it  
**if **(key.isAcceptable()) {  
this.accept(key);  
} **else if **(key.isReadable()) {  
this.read(key);  
}  
}  
} **catch **(Exception e) {  
e.printStackTrace();  
}  
}  
}

Waking up the selector will usually result in an empty selection key set. So our logic above will skip right over the key set loop and jump into processing pending changes. In truth the order of events here probably doesn't matter all that much. This is just how I prefer to do it. If you take a look at the logic we've added for handling those pending events there are a few things to point out. One is that it's pretty simple. All we do is update the interest ops for the selection key on a given socket channel. The other is that at the end of the loop we clear out the list of pending changes. This is easy to miss, in which case the selecting thread will continually reapply "old" changes.

We're almost done on the writing front, we just need one more piece, the actual write. This means another change to our `run()` method to check for a selection key in a writeable state.

// Check what event is available and deal with it  
**if **(key.isAcceptable()) {  
this.accept(key);  
} **else if **(key.isReadable()) {  
this.read(key);  
} **else if **(key.isWritable()) {  
this.write(key);  
}

And of course, we'll need a `write()` method. This method just has to pull data off the appropriate queue and keep writing it until it either runs out or can't write anymore. I add ByteBuffers to the per-socket internal queue because we'll need a ByteBuffer for the socket write anyway and ByteBuffer's conveniently track how much data remains in the buffer. This last bit is important because a given write operation may end up only writing a part of the buffer (at which point we can stop writing to that socket because further writes will achieve nothing: the socket's local buffer is full). Using a ByteBuffer saves us having to either resize byte arrays in the queue or track an index into the byte array at the head of the queue.

But I've digressed so let's take a look at `write()`.

**private ****void **write(SelectionKey key) **throws **IOException {  
SocketChannel socketChannel = (SocketChannel) key.channel();  
  
**synchronized **(this.pendingData) {  
List queue = (List) this.pendingData.get(socketChannel);  
  
// Write until there's not more data ...  
**while **(!queue.isEmpty()) {  
ByteBuffer buf = (ByteBuffer) queue.get(0);  
socketChannel.write(buf);  
**if **(buf.remaining() > 0) {  
// ... or the socket's buffer fills up  
**break**;  
}  
queue.remove(0);  
}  
  
**if **(queue.isEmpty()) {  
// We wrote away all data, so we're no longer interested  
// in writing on this socket. Switch back to waiting for  
// data.  
key.interestOps(SelectionKey.OP_READ);  
}  
}  
}

The only point I want to make about the above code is this. You'll notice that it writes as much data as it can, stopping only when there's no more to write or the socket can't take any more. This is one approach. Another is to write only the first piece of data queued and then continue. The first approach is likely to result in better per-client service while the second is probably more fair in the face of multiple active connections. I suggest you play around a bit before choosing an approach.

We now have the bulk of a working NIO server implementation. In fact, the server you've written so far (if you've been following along diligently) will accept connections, read data off those connections, and echo back the read data to the client that sent it. It will also handle clients disconnecting by deregistering the socket channel from the selector. There's not a whole lot more to a server.

The client
----------

Before getting started it's worth mentioning that the client implementation is going to end up looking a _lot_ like the server implementation we just finished writing. There's going to be a lot of common functionality that can be factored out and shared between the client and server implementation. For the most part I'm going to totally ignore that and write the client as a standalone piece of code to avoid clouding the picture with layer upon layer of abstraction. I'll leave that as an exercise to the reader. However, there are one or two little bits of code we've already written that will be shared between the implementations, because not doing so may reduce clarity. It's a judgement call on my part so if you disagree, forgive me and try not to let it bother you too much.

That said, let's get on with the client code. Once again, we're going to need some infrastructure. We can actually borrow a lot from the server code, since once a connection is accepted/established there's not a lot of difference between a client and server. A client still needs to read and write data, so at the very least we're going to need that logic. Given the similarities I'm going to build the client implementation by starting with the server implementation and tweaking it as needed.

First, let's remove some bits we don't need. The server socket channel can go, we won't be needing that. We can also remove the `accept()` method and the logic in the `run()` method that invokes it.

Next we can throw out most of the logic in the `initSelector()` method. In fact, we only need the first line, leaving us with a method that looks like this.

**private **Selector initSelector() **throws **IOException {  
// Create a new selector  
**return **SelectorProvider.provider().openSelector();  
}

We're still missing the most important piece of client-side logic: establishing a connection. This usually goes hand in hand with wanting to send some data. There are scenarios where that's not true: you might, for example, want to establish a bunch of connections to a remote server up front and then send all traffic over those connections. I'm not going to take that approach but it's not hard to get there from here. While we're on the topic of what I'm _not_ going to do, I'm also not going to implement connection pooling. I'm not going to implement a client that talks to a bunch of remote servers on different addresses; the client here will talk to one remote server (albeit using multiple simultaneous connections). What I _am_ going to implement is a client that establishes a new connection, sends a message, waits for a response and then disconnects (oh, and I'm not going to handle the case where a response isn't received). Anything beyond that adds very little beyond more code to confuse the issue.

Phew, that paragraph really got away from me. Back to the code! What we'll need is a `send()` method that requests a new connection and queues data to be sent on that connection when it comes up. We'll also need some way of notifying the caller when the response is received, but one step at a time.

But before we can put together the `send()` method we really need a method for initiating a new connection. The `send()` method will make more sense if we introduce `initiateConnection()` first.

**private **SocketChannel initiateConnection() **throws **IOException {  
// Create a non-blocking socket channel  
SocketChannel socketChannel = SocketChannel.open();  
socketChannel.configureBlocking(**false**);  
  
// Kick off connection establishment  
socketChannel.connect(**new **InetSocketAddress(this.hostAddress, this.port));  
  
// Queue a channel registration since the caller is not the   
// selecting thread. As part of the registration we'll register  
// an interest in connection events. These are raised when a channel  
// is ready to complete connection establishment.  
**synchronized**(this.pendingChanges) {  
this.pendingChanges.add(**new **ChangeRequest(socketChannel, ChangeRequest.REGISTER, SelectionKey.OP_CONNECT));  
}  
  
**return **socketChannel;  
}

In a nutshell, initiating a connection means creating a new non-blocking socket channel, initiating a (non-blocking) connection and registering an interest in finishing the connection. The only wrinkle is that, since the calling thread is not the selecting thread, the last step must be deferred. We _don't_ wake the selecting thread up because the method that calls this will want to queue some data to be written when the connection is completed. Waking the selecting thread up here opens us up to a race condition where the connection is completed by the selecting thread before the calling thread queues the data and OP_WRITE is never set on the channel's selection key.

Given the requirements we discussed earlier and the `initiateConnection()` above, our `send()` method needs to look something like this (with an additional instance member which will be clarified later).

  
// Maps a SocketChannel to a RspHandler  
**private **Map rspHandlers = Collections.synchronizedMap(**new **HashMap());  
  
  
**public ****void **send(**byte**\[\] data, RspHandler handler) **throws **IOException {  
// Start a new connection  
SocketChannel socket = this.initiateConnection();  
  
// Register the response handler  
this.rspHandlers.put(socket, handler);  
  
// And queue the data we want written  
**synchronized **(this.pendingData) {  
List queue = (List) this.pendingData.get(socket);  
**if **(queue == **null**) {  
queue = **new **ArrayList();  
this.pendingData.put(socket, queue);  
}  
queue.add(ByteBuffer.wrap(data));  
}  
  
// Finally, wake up our selecting thread so it can make the required changes  
this.selector.wakeup();  
}

It's important to note that nowhere in the two methods just introduced do we request that the OP\_CONNECT flag be set on the socket channel's selection key. If we did that we'd overwrite the OP\_CONNECT flag and never complete the connection. And if we combined them then we'd run the risk of trying to write on an unconnected channel (or at least having to deal with that case). Instead what we'll do is set the OP_WRITE flag when the connection is established (we could do this based on whether or not data was queued but for our scenario it's acceptable to do it this way).

Which brings us to the second change to our `run()` method. Remember that we started out with the server implementation, so we still have the logic for handling a pending change to the interest operation set for a selection key. To that we need to handle a pending channel registration. Voila...

**switch **(change.type) {  
**case **ChangeRequest.CHANGEOPS:  
SelectionKey key = change.socket.keyFor(this.selector);  
key.interestOps(change.ops);  
**break**;  
**case **ChangeRequest.REGISTER:  
change.socket.register(this.selector, change.ops);  
**break**;  
}

And, of course, we'll need to handle the connectable event...

// Check what event is available and deal with it  
**if **(key.isConnectable()) {  
this.finishConnection(key);  
} **else if **(key.isReadable()) {  
this.read(key);  
} **else if **(key.isWritable()) {  
this.write(key);  
}  

And of course we need an implementation for `finishConnection()`.

**private ****void **finishConnection(SelectionKey key) **throws **IOException {  
SocketChannel socketChannel = (SocketChannel) key.channel();  
  
// Finish the connection. If the connection operation failed  
// this will raise an IOException.  
**try **{  
socketChannel.finishConnection();  
} **catch **(IOException e) {  
// Cancel the channel's registration with our selector  
key.cancel();  
**return**;  
}  
  
// Register an interest in writing on this channel  
key.interestOps(SelectionKey.OP_WRITE);  
}

As I mentioned before, once the connection is complete we immediately register an interest in writing on the channel. Data has already been queued (or we wouldn't be establishing a connection in the first place). Since the current thread in this case is the selecting thread we're free to modify the selection key directly.

One last change before we're ready to put it all together. Our server implementation's `read()` handed the data off to a worker thread (that just echoed it back). For the client side we really want to hand the data (the response) back to whoever initiated the original send (the request). Scroll back up the page until you find mention of a `RspHandler` (it was passed as a parameter to `send()`). That's our conduit back to the original caller and it replaces our call to the `EchoServer` in the `read()` method. So the last line looks like this:

// Handle the response  
this.handleResponse(socketChannel, this.readBuffer.array(), numRead);

Which begs an implementation of `handleResponse`. All this method has to do is look up the handler we stashed for this channel and pass it the data we've read. We let the handler indicate whether or not it's seen enough. If it has we'll close the connection.

**private ****void **handleResponse(SocketChannel socketChannel, **byte**\[\] data, **int **numRead) **throws **IOException {  
// Make a correctly sized copy of the data before handing it  
// to the client  
**byte**\[\] rspData = **new ****byte**\[numRead\];  
System.arraycopy(data, 0, rspData, 0, numRead);  
  
// Look up the handler for this channel  
RspHandler handler = (RspHandler) this.rspHandlers.get(socketChannel);  
  
// And pass the response to it  
**if **(handler.handleResponse(rspData)) {  
// The handler has seen enough, close the connection  
socketChannel.close();  
socketChannel.keyFor(this.selector).cancel();  
}  
}

The response handler itself is pretty simple but for it to make complete sense we need to see how all of this hangs together in the form of our client's `main()` method.

**public static ****void **main(String\[\] args) {  
**try **{  
NioClient client = **new **NioClient(InetAddress.getByName("localhost"), 9090);  
Thread t = **new **Thread(client);  
t.setDaemon(**true**);  
t.start();  
RspHandler handler = **new **RspHandler();  
client.send("Hello World".getBytes(), handler);  
handler.waitForResponse();  
} **catch **(Exception e) {  
e.printStackTrace();  
}  
}

The response handler implementation is shown below. I waited until last so that the `waitForResponse()` would make sense, along with some of the synchronization logic. It's entirely possible to pull this logic into the `send()` method, effectively making it a synchronous call. But this tutorial is illustrating _asynchronous_ I/O so that seems kind of silly. It's not as silly as it sounds, mind you, since you might want to handle a large number of active connections even if the client is forced to _use_ them one at a time. But this illustrates how to perform the send asynchronously. Everything else is merely details.

NIO and SSL on 1.4
------------------

It's possible with a little craftiness to use SSL over NIO under Java 1.4 with no external libraries. Java 1.5 introduced the SSLEngine which I have yet to spend any "quality time" with. But from the outset my intention was to get Rox working on Java 1.4.

Before looking at some code it's worth outlining the basic approach. Under Java 1.4 the SSL libraries are written around blocking I/O. So the general idea is to use NIO to figure out efficiently when a socket has data available to be read, or is ready for data to be written. Then we cancel it's registration with our selector, configure it for blocking I/O, and read or write using the socket's blocking input or output streams. When we're done we configure the socket for non-blocking I/O and reregister it with our selector. To avoid blocking indefinitely when we've read all the data available on the socket we set the socket timeout to a very small value (1ms) and rely on a timeout when we've exhausted the socket's data. The actual SSL connection is established by wrapping the Socket in an SSLSocket instance when the connection is first established.

So the first thing we need is a very small change to our server's `accept()` method ...

**private ****void **accept(SelectionKey key) **throws **IOException {  
// For an accept to be pending the channel must be a server socket channel.  
ServerSocketChannel serverSocketChannel = (ServerSocketChannel) key.channel();  
  
// Accept the connection and make it non-blocking  
SocketChannel socketChannel = serverSocketChannel.accept();  
Socket socket = socketChannel.socket();  
socketChannel.configureBlocking(**false**);  
  
// Register the new socket. This will promote it to an SSLSocket  
this.registerSocket(socket, this.host, this.port, **false**);  
  
// Register the new SocketChannel with our Selector, indicating  
// we'd like to be notified when there's data waiting to be read  
socketChannel.register(this.selector, SelectionKey.OP_READ);  
}

... and our client's `finishConnection()` method.

**private ****void **finishConnection(SelectionKey key) **throws **IOException {  
SocketChannel socketChannel = (SocketChannel) key.channel();  
  
// Finish the connection. If the connection operation failed  
// this will raise an IOException.  
**try **{  
socketChannel.finishConnection();  
} **catch **(IOException e) {  
// Cancel the channel's registration with our selector  
key.cancel();  
**return**;  
}  
  
// Register the new socket. This will promote it to an SSLSocket  
this.registerSocket(socket, this.host, this.port, **false**);  
  
// Register an interest in writing on this channel  
key.interestOps(SelectionKey.OP_WRITE);  
}

The `registerSocket()` method is responsible for wrapping the plain socket connection in an SSL socket, and tracking it using a `Map` member.

**protected ****void **registerSocket(Socket socket, String host, **int **port, **boolean **client) **throws **IOException {  
// "Upgrade" the new socket to an SSLSocket  
SSLSocketFactory factory = (SSLSocketFactory) SSLSocketFactory.getDefault();  
SSLSocket sslSocket = (SSLSocket) factory.createSocket(socket, host, port, **true**);  
  
// Ensure that when we start reading on this socket it  
// acts the part of the server during the SSL handshake.  
sslSocket.setUseClientMode(client);  
  
// Configure acceptable cipher suites here  
  
this.sslSocketMap.put(socket, sslSocket);  
}

And we'll need a `deregisterSocket()` method so we can clean up when we handle errors and disconnections. The `sslSessionMap` member is another `Map` we'll see shortly. Simply put, we use it to track established SSL sessions so we know whether we're still handshaking.

**protected ****void **deregisterSocket(Socket socket) {  
this.sslSocketMap.remove(socket);  
this.sslSessionMap.remove(socket);  
}

Our `read()` method looks quite different when we're using SSL. The only thing to point out here that's not explicit is the use of a `blockingReadBuf` member. This is just a plain old byte array.

**private ****void **read(SelectionKey key) **throws **IOException {  
SocketChannel socketChannel = (SocketChannel) key.channel();  
Socket socket = socketChannel.socket();  
  
SSLSocket sslSocket = (SSLSocket) this.sslSocketMap.get(socket);  
key.cancel();  
key.channel().configureBlocking(**true**);  
  
this.configureSSLSocket(socket, sslSocket);  
  
InputStream is = sslSocket.getInputStream();  
**int **numRead;  
**try **{  
numRead = is.read(blockingReadBuf, 0, blockingReadBuf.length);  
} **catch **(SocketTimeoutException e) {  
// The read timed out so we're done.  
numRead = 0;  
} **catch **(IOException e) {  
this.deregisterSocket(socket);  
// The remote entity probably forcibly closed the connection.  
// Nothing to see here. Move on.  
// No need to cancel, already done  
**return**;  
}  
  
**if **(numRead == -1) {  
// Don't queue a cancellation since we have alread cancelled the   
// channel's registration. Just close the socket.  
this.deregisterSocket(socket);  
sslSocket.close();  
// The caller needs to be notifed. Although this is  
// a "clean" close from the caller's perspective this  
// is unexpected. So we manufacture an exception.  
**throw** **new **RemoteSocketClosedException("Remote entity closed connection");  
}  
  
**try **{  
**if **(numRead > 0) {  
// Hand the data off to our worker thread  
this.worker.processData(this, socketChannel, this.blockingReadBuf(), numRead);  
}  
} **finally **{  
key.channel().configureBlocking(**false**);  
// Queue a channel reregistration  
**synchronized**(this.pendingChanges) {  
this.pendingChanges.add(**new **ChangeRequest(socketChannel, ChangeRequest.REGISTER, SelectionKey.OP_CONNECT));  
}  
}  
}

This introduces one additional method, shown below. The `configureSSLSocket` method forces the SSL handshake, validates the SSL handshake completed successfully, and handles the weird case where the client is using SSL and the server is not. The way the SSL protocol is laid out means an SSL client connecting to a non-SSL server will tend to sit waiting patiently doing nothing. This is because the SSL protocol starts with the server sending a message to newly connecting clients. But a non-SSL HTTP server waits for a request from the client. To combat this we time the session out after 30 seconds. It's not pretty, but it works.

**private ****void **configureSSLSocket(Socket socket, SSLSocket sslSocket) **throws **IOException {  
**if **(!this.sslSessionMap.containsKey(socket)) {  
// We need a timeout here for the case where the server is not using SSL,  
// in which case both sides simply sit waiting for something from the  
// other side.  
sslSocket.setSoTimeout(this.sslHandshakeTimeout);  
  
// Get the SSL session. This forces a handshake and is used  
// to indicate that we've performed the handshake for this  
// SSLSocket.  
SSLSession session = sslSocket.getSession();  
this.sslSessionMap.put(socket, session);  
  
// Since we don't have isValid() in 1.4 this is the closest we can get to  
// a validity check.  
**if **(session.getId() != **null **&& session.getId().length != 0) {  
**if **(log.logTrace()) {  
log.trace("SSL session details: " + session);  
}  
} **else **{  
**if **(sslSocket.getUseClientMode()) {  
**throw new **SSLException("SSL session handshake failed (is the server SSL enabled?)");  
}  
// For server's we'll leave it to JSSE to raise an exception  
// when the client first tries to communicate with us.  
}  
}  
  
// Set the read timeout to the smallest legal value so  
// when data is available we can read it in blocking mode.  
// This must be done only after the handshake has completed.  
sslSocket.setSoTimeout(1);  
}

That about covers reading. Writing is quite similar and uses some of the same building blocks.

**private ****void **write(SelectionKey key) **throws **IOException {  
SocketChannel socketChannel = (SocketChannel) key.channel();  
Socket socket = socketChannel.socket();  
  
**try **{  
SSLSocket sslSocket = (SSLSocket) this.sslSocketMap.get(socket);  
key.cancel();  
key.channel().configureBlocking(**true**);  
  
this.configureSSLSocket(socket, sslSocket);  
  
OutputStream os = sslSocket.getOutputStream();  
ByteBuffer buf = this.getWriteBuffer(socket);  
os.write(buf.array());  
  
// We've written the data, reregister the channel  
key.channel().configureBlocking(**false**);  
this.queueRegistration(socketChannel);  
} **catch**(SSLException e) {  
// Don't suppress this, since we depend on it to detect non-SSL remote entities  
**throw **e;  
} **catch**(IOException e) {  
// An error occurred, close the channel to prevent spinning in a loop here  
this.deregisterSocket(socketChannel.socket());  
key.cancel();  
socketChannel.close();  
}  
}

That's a pretty quick run through of how I mixed SSL and NIO under Java 1.4. I rushed this section a little so if there are errors or omissions forgive me (and let me know so I can fix them).

The code
--------

The code samples here are not entirely complete. I've left out a few minor details for clarity. Things like import statements. Complete working source code is available here:

*   [NioServer.java](http://rox-xmlrpc.sourceforge.net/niotut/src/NioServer.java)
*   [EchoWorker.java](http://rox-xmlrpc.sourceforge.net/niotut/src/EchoWorker.java)
*   [ServerDataEvent.java](http://rox-xmlrpc.sourceforge.net/niotut/src/ServerDataEvent.java)
*   [ChangeRequest.java](http://rox-xmlrpc.sourceforge.net/niotut/src/ChangeRequest.java)
*   [NioClient.java](http://rox-xmlrpc.sourceforge.net/niotut/src/NioClient.java)
*   [RspHandler.java](http://rox-xmlrpc.sourceforge.net/niotut/src/RspHandler.java)

About the author
----------------

My name is James Greenfield. I'm currently employed by [Amazon](http://za.amazon.com/) in Cape Town, South Africa.

I enjoy writing code that other people actually use. Most of all though, I enjoy writing code for other programmer's. When the mood takes me I also enjoy writing documentation for other programmer's. I'm happiest when I can take something complicated, or unpleasant, or opaque, or just plain hard, and make it a non-issue.

I can be reached at [nio@flat502.com](mailto:nio@flat502.com) (I sense a lame "For a good time..." joke lurking in there).





