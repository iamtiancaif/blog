---
layout: post
title:  "a c10m iot platform"
categories: computer
tags:  computer linux java
author: JiXianzhao
---

* content
{:toc}

A rough estimation about memory overhead
========================================

conclusion first
----------------

28000 sockets consume 70M memory of Old Generation.

2661.800285714 Byte per socket

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
